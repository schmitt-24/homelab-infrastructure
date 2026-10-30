# Setup Main Server
## Install OMV
Download OpenMediaVault ISO from official Website.
<br />Flash ISO with Rufus on USB Stick.
<br />Install OpenMediaVault.

## Install OMV Plugins
OMV Extras Plugin:
In Terminal or SSH
`wget -O - https://github.com/OpenMediaVault-Plugin-Developers/packages/raw/master/install | bash`

### Docker (compose) Installation:
<br />In the web UI go to System -> Plugins -> search for omv-extras -> install, then install omv-compose plugin as well.
<br />After that Services -> Compose menu. Docker containers can be managed there.

### Portainer Installation:
Via Terminal or SSH
`docker volume create portainer_data`
`docker run -d -p 9000:9000 --name portainer --restart=always -v /var/run/docker.sock:/docker.sock -v portainer_data:/data portainer/portainer-ce`

Then go to http://<ip>:9000 and create an Portainer admin account.

### SearXNG Installation:
In the Portainer local Environment -> Add stack.
Using Docker compose in Web add this YAML:
```yaml
services:
  searxng:
    image: searxng/searxng:latest
    container_name: searxng
    ports:
      - "8080:8080"
    volumes:
      - ./searxng:/etc/searxng
    restart: unless-stopped
```
<br />Then deploy stack.
<br />SearXNG will be accessable at `http://<ip>:8080`

### Twingate Setup:
Create an Account on Twingate and create a Network. Deploy a Connector with Docker.
<br />Create Access and Refresh Tokens.
<br />Enable 'Make Connector available on local network' and 'Local network connection logs'.
<br />Via SSH paste the `docker run` command and run it.
<br />Install App whereever wanted and add Resources.

### Gitea Installation:
Via SSH create a Directory first.
`mkdir -p /opt/gitea`

Then in Portainer Add a new Stack again and paste this with Docker web:
```yaml
services:
  gitea:
    image: gitea/gitea:latest
    container_name: gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
    volumes:
      - /opt/gitea:/data
    ports:
      - "3000:3000"
      - "222:22"
    restart: unless-stopped
```
<br />Configure and Instal Gitea.

### Nextcloud Installation:
Create a Direcotry.
`mkdir -p /opt/nextcloud`

Then paste this as well in Portainer:
Change the `changeme` to a strong password
```yaml
services:
  nextcloud:
    image: nextcloud:latest
    container_name: nextcloud
    ports:
      - "8081:80"
    volumes:
      - /opt/nextcloud:/var/www/html
    environment:
      - NEXTCLOUD_ADMIN_USER=admin
      - NEXTCLOUD_ADMIN_PASSWORD=changeme
    restart: unless-stopped
```