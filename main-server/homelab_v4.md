# Setup

## Install Debian
In the Installation Process use this configuration for Debian.

```
Hostname: services
Domain: homelab
Partitioning: Guided
Software selection: uncheck everything except SSH server and standard system utilities
```

---

## Mount HDD
Find the HDD device name:
`lsblk`

Create mount point:
`sudo mkdir -p /mnt/hdd`

Get UUID of the partition:
`sudo blkid /dev/sdb1`

Copy UUID value and Add it to /etc/fstab for persistent mounting:
`sudo nano /etc/fstab`
Add this line at the bottom, replacing the UUID with yours:
UUID=your-uuid-here  /mnt/hdd  ext4  defaults  0  2

Mount without rebooting:
`sudo mount -a`

Verify:
`df -h | grep hdd`

---

## Setup Docker

Update and install sudo:
```bash
apt update && apt upgrade -y
apt install curl -y
apt install sudo curl -y
usermod -aG sudo username
```

Install Docker:
```bash
curl -fsSL https://get.docker.com | sh
```

Add user to docker group:
```bash
usermod -aG docker username
```

---

### Setup Compose Stack

Switch to newly created User and create directory + compose file:
```bash
sudo mkdir -p /opt/stacks
sudo nano /opt/stacks/services.yml
```

Create Prometheus config file before deploying:
```bash
sudo mkdir -p /opt/stacks/prometheus
sudo nano /opt/stacks/prometheus/prometheus.yml
```

Paste Prometheus config:
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

Create Database file for Filebrowser:
```bash
sudo mkdir -p /opt/filebrowser
sudo touch /opt/filebrowser/filebrowser.db
```

Add Write Permission to User:
```bash
sudo chmod 666 /opt/filebrowser/filebrowser.db
```

---

### Move Compose Stack

Paste the services.yml into `/opt/stacks/services.yml`:

---

## Deploy

```bash
sudo docker compose -f /opt/stacks/services.yml up -d
```

---

## Setup Domain

In Nginx Proxy Manager go to SSL Certificates -> Add (Lets Encrypt), enter the domain for example:

```bash
*.yourdomain.ltd
yourdomain.ltd
```

Enable DNS Challenge and paste the DNS Providers API Token.


After that add Pi-hole to the Compose Stack and change placeholder
```bash
services:
  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    restart: unless-stopped
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "8083:80/tcp"
    environment:
      TZ: "Europe/Berlin"
      FTLCONF_webserver_api_password: "CHANGE_ME"
      FTLCONF_dns_listeningMode: "all"
      FTLCONF_dns_upstreams: "194.242.2.2"
      FTLCONF_misc_dnsmasq_lines: "address=/yourdomain.ltd/your_ip_address"
    volumes:
      - ./etc-pihole:/etc/pihole
```

Login to Pi-hole with http://your_ip_address:8083/admin and Password, then test it from Host PC:
```bash
dig @your_ip_address git.yourdomain.tld
```

Create first Proxy in Nginx Proxy Manager with Gitea as Test
Insert Address in Details Tab and under SSL choose the created SSL Certificate *.yourdomain.ltd
Enable Force SSL, HTTP/2 Support and HSTS Enabled.
Edit in services.yml these variables:
```bash
    environment:
      - GITEA__server__DOMAIN=git.yourdomain.tld
      - GITEA__server__SSH_DOMAIN=git.yourdomain.tld
      - GITEA__server__SSH_PORT=222
      - GITEA__server__ROOT_URL=https://git.yourdomain.tld/
```

Enable these Rules so the Host PC knows DNS Name:

Windows (Admin Powershell):
```bash
Add-DnsClientNrptRule -Namespace ".yourdomain.tld" -NameServers "your_ip_address"
Clear-DnsClientCache
```

MacOS:
```bash
sudo mkdir -p /etc/resolver
echo "nameserver your_ip_address" | sudo tee /etc/resolver/yourdomain.tld
```

Fedora Linux:
```bash
sudo nano /etc/hosts
```

And add this at the end of the file
```bash
your_ip_address git.yourdomain.tld search.yourdomain.tld files.yourdomain.tld grafana.yourdomain.tld portainer.yourdomain.tld pihole.yourdomain.tld prometheus.yourdomain.tld auth.yourdomain.tld
```

Create the Rest of the Proxies in the same way and add these in services.yml under the correct service.
```bash
  searxng:
    environment:
      - SEARXNG_BASE_URL=https://search.yourdomain.tld/

  grafana:
    environment:
      - GF_SERVER_DOMAIN=grafana.yourdomain.tld
      - GF_SERVER_ROOT_URL=https://grafana.yourdomain.tld/
```

To Add authelia 2FA, in the Directory that contains services.yml run these commands:
```bash
mkdir -p authelia/secrets
for f in jwt session storage; do openssl rand -hex 32 > authelia/secrets/$f; done
chmod 600 authelia/secrets/*
docker run --rm -it authelia/authelia:4.39 authelia crypto hash generate argon2
```

Copy the Argon2 key for Later but first in the Authelia Directory create these files and change Filler:

authelia/configuration.yml
```bash
theme: auto
server:
  address: 'tcp://:9091'
log:
  level: info
authentication_backend:
  file:
    path: /config/users_database.yml
access_control:
  default_policy: deny
  rules:
    - domain: '*.yourdomain.ltd'
      policy: two_factor
session:
  cookies:
    - domain: 'yourdomain.ltd'
      authelia_url: 'https://auth.yourdomain.ltd'
storage:
  local:
    path: /config/db.sqlite3
notifier:
  filesystem:
    filename: /config/notification.txt
```

authelia/users_database.yml
```bash
users:
  FILLER:
    disabled: false
    displayname: 'FILLER'
    password: 'PASTE_ARGON2_DIGEST_HERE'
    email: 'you@example.com'
    groups:
      - admins
```

And add to services.yml
```bash
  authelia:
    image: authelia/authelia:4.39
    container_name: authelia
    restart: unless-stopped
    volumes:
      - ./authelia:/config
    environment:
      TZ: Europe/Berlin
      AUTHELIA_IDENTITY_VALIDATION_RESET_PASSWORD_JWT_SECRET_FILE: /config/secrets/jwt
      AUTHELIA_SESSION_SECRET_FILE: /config/secrets/session
      AUTHELIA_STORAGE_ENCRYPTION_KEY_FILE: /config/secrets/storage
```

Restart Container and Check Logs
```bash
docker compose -p stacks -f services.yml up -d
docker logs authelia --tail 20
```

In Nginx Proxy Manager create the Authelia Proxy the same way
Test 2FA with Prometheus and Paste in Nginx Proxy Manager under SSL Advanced this and change Filler:
```bash
set $upstream_authelia http://authelia:9091/api/authz/auth-request;

location /internal/authelia/authz {
    internal;
    auth_request off;
    proxy_pass $upstream_authelia;
    proxy_set_header X-Original-Method $request_method;
    proxy_set_header X-Original-URL $scheme://$http_host$request_uri;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_set_header Content-Length "";
    proxy_set_header Connection "";
    proxy_pass_request_body off;
    proxy_http_version 1.1;
}

auth_request /internal/authelia/authz;
error_page 401 =302 https://yourdomain.ltd/?rd=$scheme://$http_host$request_uri;
```

Open Prometheus, Login to Authelia with the first Code send to authelia/notification.txt on the Server.
After confirming that it works and setting up normal 2FA add the command above to all Proxies except Authelia itself.
