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

Create the mount point:
`sudo mkdir -p /mnt/hdd`

Get the UUID of the partition:
`sudo blkid /dev/sdb1`

Copy the UUID value and Add it to /etc/fstab for persistent mounting:
`sudo nano /etc/fstab`
Add this line at the bottom, replacing the UUID with yours:
UUID=your-uuid-here  /mnt/hdd  ext4  defaults  0  2

Mount it now without rebooting:
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

Switch to the newly created User and create directory + compose file:
```bash
sudo mkdir -p /opt/stacks
sudo nano /opt/stacks/services.yml
```

Create Prometheus config file before deploying:
```bash
sudo mkdir -p /opt/stacks/prometheus
sudo nano /opt/stacks/prometheus/prometheus.yml
```

Paste this minimal Prometheus config:
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

Add Write Permission to the User running the Container:
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

## Service Access

| Service | URL | Setup |
|---|---|---|
| Portainer | http://your_ip:9000 | Create admin account |
| Nginx Proxy Manager | http://your_ip:81 | Default login: Create admin account |
| Gitea | http://your_ip:3000 | Create admin account |
| SearXNG | http://your_ip:8080 | None |
| Filebrowser | http://your_ip:8081 | Default login: admin / admin |
| Vaultwarden | http://your_ip:8082 | Create admin account |
| Uptime Kuma | http://your_ip:3001 | Create admin account |
| Netdata | http://your_ip:19999 | None or Create admin account |
| Prometheus | http://your_ip:9090 | None |
| Grafana | http://your_ip:3002 | Default login: admin / admin |
