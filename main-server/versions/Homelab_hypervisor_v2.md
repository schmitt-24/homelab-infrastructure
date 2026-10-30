Homelab

# Hypervisor

## Setup Proxmox
Download the Proxmox VE ISO, and flash it with Rufus on an USB Stick.
Follow the installation steps in the Installer.
`https://www.proxmox.com/`
After installation Proxmox web Interface will be available at `https://<ip>:8006`

To create a Virtual Machine click the button `Create VM` in the top right and use this configuration in the dedicated sections
```bash
General:
Name: services
VM ID: 100

OS:
Select Debian ISO
Type: Linux, kernel 6.x

System:
Machine: q35
BIOS: SeaBIOS
Qemu Agent: check

Disks:
Storage: select SSD | HDD
Size: 60GB

CPU:
Cores: 1

Memory:
6144 MB

Network:
vmbr0
```

# Virtual Machines

## Machine 1 ( 100 Debian Services )
To Get the ISO simply go in the Proxmox Web Interface to local → ISO Images → Download from URL and paste with the correct version number:
`https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-X.X.X-amd64-netinst.iso`

In the Installation Process use this configuration.
<br />Hostname: services
<br />Domain: homelab
<br />Partitioning: Guided
<br />Software selection: uncheck everything except SSH server and standard system utilities

### Setup Docker
Update and install sudo
```bash
apt update && apt upgrade -y
apt install sudo curl -y
usermod -aG sudo username
```

Install docker
`curl -fsSL https://get.docker.com | sh`

Add User to docker group
`usermod -aG docker username`

#### Setup Compose Stack
Create a Directory and compose file.
```bash
sudo mkdir -p /opt/stacks
sudo nano /opt/stacks/services.yml
```

Insert this YAML into the file and change the `change me`.
```yaml
services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    ports:
      - "9000:9000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    restart: unless-stopped

  searxng:
    image: searxng/searxng:latest
    container_name: searxng
    ports:
      - "8080:8080"
    volumes:
      - searxng_data:/etc/searxng
    restart: unless-stopped

  gitea:
    image: gitea/gitea:latest
    container_name: gitea
    ports:
      - "3000:3000"
      - "222:22"
    volumes:
      - gitea_data:/data
    restart: unless-stopped

  nextcloud:
    image: nextcloud:latest
    container_name: nextcloud
    ports:
      - "8081:80"
    volumes:
      - nextcloud_data:/var/www/html
    environment:
      - NEXTCLOUD_ADMIN_USER=admin
      - NEXTCLOUD_ADMIN_PASSWORD=changeme
    restart: unless-stopped

  filebrowser:
    image: filebrowser/filebrowser:latest
    container_name: filebrowser
    ports:
      - "8082:80"
    volumes:
      - /mnt/hdd:/srv
      - filebrowser_data:/database
    restart: unless-stopped

volumes:
  portainer_data:
  searxng_data:
  gitea_data:
  nextcloud_data:
  filebrowser_data:
```

Deploy with this command
`sudo docker compose -f /opt/stacks/services.yml up -d`