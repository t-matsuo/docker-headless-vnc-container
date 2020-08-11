# Docker container images with "headless" VNC session and butterfly web console for Japanese

This repository contains a collection of Docker images with headless VNC environments and butterfly web console through reverse proxy(nginx(pam auth)).
This repository is based on [docker-headless-vnc-container](https://github.com/ConSol/docker-headless-vnc-container), but it supports CentOS 7 with xfce only.
Dockerfile "Dockerfile.centos.xfce.vnc" is only maintained.

Each Docker image is installed with the following components:

* Desktop environment [**Xfce4**](http://www.xfce.org)
* VNC-Server (default VNC port `5901`)
* [**noVNC**](https://github.com/novnc/noVNC) - HTML5 VNC client (port `6901`)
* [**butterfly**](https://github.com/paradoxxxzero/butterfly) - Terminal Emulator on browser (port `57575`)
* [**butterfly**](https://github.com/filebrowser/filebrowser) - File browser (port `57576`)

* Browsers:
  * Chromium
  * Firefox
  * Edge
  
![Docker VNC Desktop access via HTML page](.pics/screen-desktop.png)

![Docker Terminal access via HTML page](.pics/screen-term.png)

## Kubernetes

* [Kubernetes usage](./kubernetes/README.md)

## Usage

- Docker (ssl)

      docker run -d -p 8080:8080 -e PASSWORD=password --name centos-xfce-ja tmatsuo/centos-xfce-ja

- Docker (ssl with your key and csr)

      docker run -d -p 8080:8080 -e PASSWORD=password -v /path/to/server.key:/etc/pki/nginx/server.key -v /path/to/server.crt:/etc/pki/nginx/server.crt  --name centos-xfce-ja tmatsuo/centos-xfce-ja

- Docker (non ssl)

      docker run -d -p 8080:8080 -e PASSWORD=password -e NOSSL=true --name centos-xfce-ja tmatsuo/centos-xfce-ja

Access http(s)://your-host-name:8080/desktop/ to access xfce desktop.
Access http(s)://your-host-name:8080/term/ to access web console.
Access http(s)://your-host-name:8080/file/ to access file browser.

Reverse proxy(Nginx) requires basic auth. You enter "root" user to login.
You can change root password using passwd command after login.

### Override VNC environment variables

The following VNC environment variables can be overwritten at the `docker run` phase to customize your desktop environment inside the container:
* `VNC_COL_DEPTH`, default: `24`
* `VNC_RESOLUTION`, default: `1800x850`

### Override reverse proxy listen port

* `PORT`, default: `8080`

