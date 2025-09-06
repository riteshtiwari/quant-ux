[![Docker Image Build and Push to Dockerhub - CI/CD](https://github.com/KlausSchaefers/quant-ux/actions/workflows/docker.yml/badge.svg)](https://github.com/KlausSchaefers/quant-ux/actions/workflows/docker.yml)

# Quant-UX - Prototype, Test and Learn

Quant UX is a research, usability and prototyping tool to quickly test your designs and get data driven insights. 
This repo contains the front end. You can find a working demo at https://quant-ux.com/#/

![Alt text](docs/release.png?raw=true "Quant-UX preview")

## Develpment setup
### Prerequisite
```
npm install
```


### Running Locally on the Host Machine

#### Compiles and hot-reloads for development
```
npm run serve
```

#### Compiles and minifies for production
```
npm run build
```

#### Run your unit tests
```
npm run test:unit
```

#### Lints and fixes files
```
npm run lint
```

### Developing inside a Docker Container
If you wish to develop by running the service exclusively through Docker, you can build a development image using:
```bash
make build-dev
```
This will create a Docker Image tagged under `quant-ux`. You can then replace the `klausenschaefersinho/quant-ux` inside your docker-compose file with the newly build `quant-ux` image. Don't forget to mount the source code after replacing the image.

If you're using the provided `docker/docker-compose.yml`, you can simply add the following volume mount to the qux-fe service:
```yml
    volumes:
      - ../src:/home/node/src
```

You can then make use of the following Makefile rules for quick docker environment setup and teardown:
```bash
# docker compose up - targets docker/docker-compose.yml
make up

# docker compose down - targets docker/docker-compose.yml
make down
```

# Installation

The easiest way to get your own installation up and running is using the prebuild Docker images by [Brian McGonagill](https://github.com/bmcgonag).  You can find the repo and instructions at https://github.com/bmcgonag/quant-ux-docker/


## Manual Installation

Quant-UX has two components. A front-end (this package) and a backend (qux-java). The front-end needs Node.js (> 12) installed. The backend needs a Mongo DB, a Mail Server (SMTP) and Java (> 1.8). The front-end comes with it's own mini web server, which also include a proxy that redirects all request to the correct backend.

## Docker

The easiest way to get your own Quant-UX installation running is using the Docker images. 

1) Create a docker compose file (`docker-compose.yaml`) and set the environment variables.

```yaml
version: '3'

services:
  mongo:
    restart: always
    container_name: quant-ux-mongo
    image: mongo
    volumes:
      - ./data:/data/db        # pth for the data to be stored and kept on your host machine is on the left side of the ":"
  qux-fe:
    restart: always
    container_name: quant-ux-frontend
    image: klausenschaefersinho/quant-ux
    environment:
      - QUX_PROXY_URL=http://quant-ux-backend:8080        # this is the path the front end uses to talk tot he backend
      - QUX_AUTH=qux
      - QUX_KEYCLOAK_REALM=
      - QUX_KEYCLOAK_CLIENT=
      - QUX_KEYCLOAK_URL=
      - QUX_WS_URL=ws://127.0.0.1:8086        # change to where the websocket server is deployed for external access
    links:
      - mongo
      - qux-be
    ports:
      - 8082:8082        # change the left side port if your host machine already has 8082 in use
    depends_on:
      - qux-be
  qux-be:
    restart: always
    container_name: quant-ux-backend
    image: klausenschaefersinho/quant-ux-backend
    volumes:
      - ./quant-ux-data:/app-data
    environment:
      - QUX_HTTP_HOST=http://quant-ux-frontend:8082   # this is the URL included in the mails, e.g. password resets
      - QUX_HTTP_PORT=8080  # This is the port the backend will use
      - QUX_MONGO_DB_NAME=quantux  # the database / collection name in mongodb
      - QUX_MONGO_TABLE_PREFIX=quantux  # table / document prefix in mongodb
      - QUX_MONGO_CONNECTION_STRING=mongodb://quant-ux-mongo:27017 # this assumes your mongodb container will be called "quant-ux-mongo" in the docker-compose file
      - QUX_MAIL_USER=mail_admin@example.com        # this should be your smtp email user
      - QUX_MAIL_PASSWORD=sTr0ngPa55w0Rd        # this should be your smtp email password
      - QUX_MAIL_HOST=mail.example.com        # this should be your smtp host address
      - QUX_JWT_PASSWORD=some-long-string-of-mix-case-chars-and-nums        # you should change this to a real JWT secret
      - QUX_IMAGE_FOLDER_USER=/app-data/qux-images        # this folder should mapped in the volume
      - QUX_IMAGE_FOLDER_APPS=/app-data/qux-image-apps        # this folder should mapped in the volume
      - TZ=America/Chicago        # change to your timezone
      - QUX_AUTH_SERVICE=qux
      - QUX_KEYCLOAK_SERVER= # just the keycloak host & port
      - QUX_KEYCLOAK_REALM=
      - QUX_USER_ALLOW_SIGNUP=true # set the false to not allow users to signup
      - QUX_USER_ALLOWED_DOMAINS=* # comma separated list of domains, e.g. 'my-server.com' or '*' for all
    depends_on:
      - mongo
  qux-ws:
    restart: always
    container_name: quant-ux-websocket-server
    image: klausenschaefersinho/quant-ux-websocket
    environment:
      - QUX_SERVER=http://quant-ux-backend:8080/
      - QUX_SERVER_PORT=8086
    ports:
      - 8086:8086
    links:
      - qux-be
    depends_on:
      - qux-be

```

Make sure to update `QUX_JWT_PASSWORD` the ENV variable to make sure your installation is secure.
Update `QUX_HTTP_HOST`, `QUX_MAIL_USER`, `QUX_MAIL_PASSWORD` and `QUX_MAIL_HOST` to sure correct mail handling


2) Start the containers with the following command

```bash
docker compose up
```

## One-Click deployment

### Elestio
You can deploy an instance of Quant UX with few clicks and minimal configuration on cloud service provider of your choice.
 
[![Deploy on Elestio](https://elest.io/images/logos/deploy-to-elestio-btn.png)](https://elest.io/open-source/quant-ux)

### RepoCloud.io
You can deploy an instance of Quant UX with one click on RepoCloud.
 
[![Deploy](https://d16t0pc4846x52.cloudfront.net/deploylobe.svg)](https://repocloud.io/details/?app_id=302)

## Kubernets

You can find a kubernets configuration here https://github.com/engmsilva/quant-ux-k8s/tree/master/k8s

### Backend

- Install Mongo DB (> 4.4)

- Install Java (1.8)

- Checkout the backend

```
git clone https://github.com/KlausSchaefers/qux-java.git
```

- This contains already a compiled version of the backend in the release folder

- Edit the matc.conf file to setup the correct mongo and mails server details. More details can be found here: https://github.com/KlausSchaefers/qux-java

- Start the server, or install as a service in Linux. 

```
java -jar release/matc.jar -Xmx2g -conf matc.conf -instances 1
```


### Front-end

- Install Node.js (> 12)

- Clone repo

```
git clone https://github.com/KlausSchaefers/quant-ux.git
```

- Install all dependecies:

```
npm install
```

- Build 
```
npm run build
```

### Config front-end
- Set the proxy server url as an ENV variable

```
export QUX_PROXY_URL=https://your.quant-ux.server.com // backend host

export QUX_WS_URL= wss.quant-ux.server.com // web socket server

```

- Start
```
node server/start.js
```

### Reverse Proxy

Now you should have a running system. It is not secure yet. The best is to put both behind a NGINX reverse proxy, which handles SSL.

- https://www.scaleway.com/en/docs/tutorials/nginx-reverse-proxy/

You can use https://letsencrypt.org/ to create SSL certificates

### With Cloudflare

# Quant-UX — Local + Share (Cloudflare Tunnels) Quick Start

Spin up the full Quant-UX stack (frontend + Java backend + WebSocket + MongoDB) on your machine with Docker, then share it publicly using **Cloudflare Quick Tunnels** — no cloud account, no DNS.

> **You’ll get**
> - Local app at `http://localhost:8082`
> - Two public URLs (one for the app, one for the WebSocket)
> - Containers only; easy to start/stop

---

## 0) Prerequisites

**Docker + Compose**
- **macOS**: Install Docker Desktop.
- **Ubuntu/Linux**:
  ```bash
  sudo apt-get update
  sudo apt-get install -y docker.io docker-compose-plugin
  sudo usermod -aG docker $USER && newgrp docker
cloudflared (the Cloudflare Tunnel client)

macOS (Homebrew):

bash
Copy code
brew update && brew install cloudflared
Ubuntu (if repo package not available, grab the .deb from Cloudflare’s site):

bash
Copy code
sudo apt-get update
sudo apt-get install -y cloudflared || true
1) Clone the repo
bash
Copy code
git clone https://github.com/KlausSchaefers/quant-ux
cd quant-ux/docker
2) Start the stack (Docker)
bash
Copy code
docker compose up -d
docker compose ps
What should be running

Frontend on 8082

WebSocket on 8086

Backend on 8080

MongoDB in the compose network

Sanity check

bash
Copy code
curl -I http://localhost:8082
# Expect HTTP/1.1 200/301-ish
Open http://localhost:8082 in your browser (local only for now).

3) Create two Quick Tunnels (two terminals)
Quick Tunnels print a random https://<something>.trycloudflare.com URL and stay alive as long as the command runs.

Terminal A — Frontend

bash
Copy code
cloudflared tunnel --hello-world   # quick sanity check; should print a URL
cloudflared tunnel --url http://localhost:8082
# Copy the printed https://<FE>.trycloudflare.com
Terminal B — WebSocket

bash
Copy code
cloudflared tunnel --url http://localhost:8086
# Copy the printed https://<WS>.trycloudflare.com
Keep both terminals running.

If a command looks “stuck” or doesn’t show a URL, hit Ctrl+C and use the Troubleshooting section below.

4) Point the frontend at the public WS URL
By default the UI tries ws://127.0.0.1:8086 which only works on your machine. Change it to the public wss:// URL from Terminal B.

Open quant-ux/docker/docker-compose.yml

Find the qux-fe service → environment: and set:

yaml
Copy code
QUX_WS_URL: "wss://<WS>.trycloudflare.com"  # paste your WS tunnel URL
TZ: "America/Los_Angeles"                   # optional but recommended
QUX_JWT_PASSWORD: "<long-random-string>"    # optional but recommended
Leave QUX_PROXY_URL as-is (container-internal).

Restart to pick up env changes:

bash
Copy code
docker compose down && docker compose up -d
5) Share it
Send friends the frontend URL from Terminal A:

cpp
Copy code
https://<FE>.trycloudflare.com
The app will open a WebSocket to:

perl
Copy code
wss://<WS>.trycloudflare.com    # what you set in QUX_WS_URL
6) Verify it’s live
Open the frontend public URL.

Browser DevTools → Network → WS:

You should see 101 Switching Protocols to wss://<WS>…

If it still tries ws://127.0.0.1:8086, you didn’t update QUX_WS_URL.

If WS shows 400/403/closed, make sure the WS tunnel (Terminal B) is running.

7) Keep tunnels alive (background options)
Quick Tunnels die when you stop the command.

nohup

bash
Copy code
nohup cloudflared tunnel --url http://localhost:8082 > /tmp/qux-fe.log 2>&1 &
nohup cloudflared tunnel --url http://localhost:8086 > /tmp/qux-ws.log 2>&1 &
tail -f /tmp/qux-fe.log
tail -f /tmp/qux-ws.log
tmux

bash
Copy code
tmux new -s qux-fe 'cloudflared tunnel --url http://localhost:8082'
# detach: Ctrl+B, then D
tmux new -s qux-ws 'cloudflared tunnel --url http://localhost:8086'
Stop them later

bash
Copy code
pkill -f 'cloudflared tunnel --url http://localhost:8082'
pkill -f 'cloudflared tunnel --url http://localhost:8086'
8) Troubleshooting
A) cloudflared prints cert errors or hangs with no URL

Kill it (Ctrl+C).

Ensure no default config is confusing it:

bash
Copy code
ls -la ~/.cloudflared /usr/local/etc/cloudflared 2>/dev/null
If present, temporarily move them:

bash
Copy code
mv ~/.cloudflared ~/.cloudflared.bak 2>/dev/null || true
sudo mv /usr/local/etc/cloudflared /usr/local/etc/cloudflared.bak 2>/dev/null || true
Force a clean run (ignore any config):

bash
Copy code
: > /tmp/empty.yml
cloudflared tunnel --config /tmp/empty.yml --url http://localhost:8082
B) Using Cloudflare WARP/VPN?

Turn it off and retry Quick Tunnels (they can conflict on some setups).

C) Frontend loads but features fail

Check containers:

bash
Copy code
docker compose ps
docker compose logs -f --tail=200
Hard restart:

bash
Copy code
docker compose down -v
docker compose up -d
D) “Address already in use” on 8082/8086

bash
Copy code
lsof -iTCP:8082 -sTCP:LISTEN
lsof -iTCP:8086 -sTCP:LISTEN
kill <PID>
E) Emails (invites/reset) don’t send

Set SMTP env vars in the backend service if you need email. Without them, everything else works; email won’t.

9) Optional: one persistent (named) tunnel with two hostnames
Requires a Cloudflare account and your domain on Cloudflare. Cleaner hostnames; not necessary for a quick share.

bash
Copy code
# 1) Auth (opens browser)
cloudflared tunnel login

# 2) Create tunnel
cloudflared tunnel create qux
Create ~/.cloudflared/config.yml:

yaml
Copy code
tunnel: qux
credentials-file: /Users/<you>/.cloudflared/<UUID>.json

ingress:
  - hostname: qux.example.com
    service: http://localhost:8082
  - hostname: qux-ws.example.com
    service: http://localhost:8086
  - service: http_status:404
Add DNS CNAMEs for qux and qux-ws to the tunnel in Cloudflare, then run:

bash
Copy code
cloudflared tunnel run qux
Update compose:

yaml
Copy code
QUX_WS_URL: "wss://qux-ws.example.com"
Restart Docker.

