# Configuring Nginx with Docker

## Overview

This guide explains step-by-step how to run an **Nginx web server** inside a **Docker container**. Nginx will serve a simple website, and we will access it through a web browser.

![Nginx-Docker](https://linuxiac.b-cdn.net/wp-content/uploads/2021/06/nginx-docker.png)

## Prerequisites

- Docker installed
- Basic understanding of Docker commands
- Static website files ready for deployment

## Step-by-Step Guide with Detailed Explanations

### 1. Prepare Your Website Files

Create your static website files:

```bash
vim index.html
```

Example content:
```html
<!DOCTYPE html>
<html>
<head>
    <title>My Static Website</title>
</head>
<body>
    <h1>Hello, World!</h1>
</body>
</html>
```

**Command Explanation:**
- `vim index.html`: Opens the Vim text editor to create a new HTML file
  - Vim is a powerful text editor in Unix-like systems
  - Used here to manually create the website's main page

### 2. Select a Base Image

Pull the Alpine Linux image:

```bash
docker pull alpine:3.12.7
```

**Command Explanation:**
- `docker pull`: Downloads a Docker image from Docker Hub
- `alpine:3.12.7`: Specifies the Alpine Linux distribution version
  - Alpine is a lightweight Linux distribution ideal for containers
  - Version 3.12.7 ensures consistency and compatibility

### 3. Create and Configure the Container

#### 3.1 Launch an Interactive Container

```bash
docker run -it --name web01 alpine:3.12.7
```

**Command Breakdown:**
- `docker run`: Creates and starts a new container
- `-it`: Combines two flags
  - `-i` (interactive): Keeps STDIN open
  - `-t` (tty): Allocates a pseudo-TTY (terminal)
- `--name web01`: Assigns a human-readable name to the container
- `alpine:3.12.7`: Specifies the image to use for the container

#### 3.2 Install Nginx

Within the container, install Nginx:

```bash
apk add nginx
```

**Command Explanation:**
- `apk`: Alpine Linux package manager
- `add nginx`: Installs the Nginx web server
  - Uses Alpine's package management system
  - Installs the latest available Nginx version compatible with the Alpine version

#### 3.3 Configure Nginx

Create or modify the default Nginx configuration:

```bash
vim /etc/nginx/conf.d/default.conf
```

Recommended configuration:
```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

**Configuration Explanation:**
- `listen 80 default_server`: 
  - Listens on port 80 (standard HTTP port)
  - `default_server` means it handles requests not matched by other server blocks
- `listen [::]:80 default_server`: 
  - Enables IPv6 listening on port 80
- `root /var/www/;`: 
  - Sets the root directory for serving files
- `index index.html;`: 
  - Specifies the default index file
- `location / { ... }`:
  - Handles all incoming requests
  - `try_files $uri $uri/ =404;`: 
    - Tries to find the exact file requested
    - Falls back to directory index
    - Returns 404 if no match is found

### 4. Deploy Website Files

Copy your website files into the container:

```bash
docker cp index.html web01:/var/www/index.html
```

**Command Explanation:**
- `docker cp`: Copies files between a container and the local filesystem
- `index.html web01:/var/www/index.html`:
  - Sources from local `index.html`
  - Destinations to `/var/www/index.html` inside the `web01` container

### 5. Start Nginx

Start Nginx in the foreground to prevent container exit:

```bash
docker exec -dt web01 nginx -g 'pid /tmp/nginx.pid; daemon off;'
```

**Command Breakdown:**
- `docker exec`: Runs a command in a running container
- `-dt`: 
  - `-d` runs in detached mode (background)
  - `-t` allocates a pseudo-TTY
- `nginx`: The Nginx command
- `-g 'pid /tmp/nginx.pid; daemon off;'`:
  - `-g` sets global directives
  - `pid /tmp/nginx.pid`: Sets the PID file location
  - `daemon off;`: Prevents Nginx from daemonizing, keeping the container running

### 6. Create a Persistent Image

Commit your configured container as a new image:

```bash
docker commit web01 web-base
```

**Command Explanation:**
- `docker commit`: Creates a new image from a container's changes
- `web01`: Source container
- `web-base`: Name of the new image
  - Captures all modifications made to the original container
  - Creates a reproducible image for future deployments

### 7. Deploy with Port Mapping

Run the new image, mapping host port 80 to container port 80:

```bash
docker run -p 80:80 -dt --name web02 web-base
```

**Command Breakdown:**
- `-p 80:80`: Maps host port 80 to container port 80
  - First `80`: Host machine port
  - Second `80`: Container's internal port
- `-dt`: Detached mode with pseudo-TTY
- `--name web02`: Names the new container
- `web-base`: Uses the image created in the previous step

Restart Nginx in the new container:

```bash
docker exec -dt web02 nginx -g 'pid /tmp/nginx.pid; daemon off;'
```
Check the published website:

- Get the `IPAddress` of the running container:
```bash
docker inspect web02 | grep IPAddress
```
- Check the website:
```bash
curl <IPAddress>
```
- You can also check it with:
```bash
curl localhost
```

- Now copy the public IP address of the virtual machine to see the hosted website in the browser.

---