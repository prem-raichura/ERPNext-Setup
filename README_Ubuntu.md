# 🚀 ERPNext v16 + HRMS v16 Offline Docker Setup (Ubuntu 24.04)

This guide explains how to install ERPNext + HRMS on an Ubuntu server
when GitHub access is blocked by a firewall.

We use this method:

✅ Build image on internet machine\
✅ Export Docker image\
✅ Transfer to Ubuntu server\
✅ Load and run locally

No GitHub access required on the server.

------------------------------------------------------------------------

# 🧱 Architecture Overview

Machine A (Internet Access) → Build Docker image → Export image (.tar
file)

Machine B (Your Ubuntu Server) → Load image → Run Docker Compose →
Create site → Install apps

------------------------------------------------------------------------

# 🖥 PART 1 --- Setup Ubuntu Server (Offline Machine)

## 1️⃣ Install Docker (Ubuntu 24.04)

``` bash
sudo apt update
sudo apt install ca-certificates curl gnupg -y

sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg]   https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" |   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

Enable Docker:

``` bash
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
```

Logout and login again.

Verify:

``` bash
docker --version
docker compose version
```

Do NOT build anything on this server.

------------------------------------------------------------------------

# 💻 PART 2 --- On Internet Machine

## 1️⃣ Clone Frappe Docker

``` bash
git clone https://github.com/frappe/frappe_docker
cd frappe_docker
```

## 2️⃣ Create apps.json

``` bash
nano apps.json
```

Paste:

``` json
[
  {
    "url": "https://github.com/frappe/erpnext",
    "branch": "version-16"
  },
  {
    "url": "https://github.com/frappe/hrms",
    "branch": "version-16"
  }
]
```

------------------------------------------------------------------------

## 3️⃣ Build Custom Image

``` bash
docker build \
  --build-arg=PYTHON_VERSION=3.11 \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=version-16 \
  --build-arg=APPS_JSON_BASE64=$(base64 -w 0 apps.json) \
  --tag=custom-erpnext-hrms:v16 \
  --file=images/custom/Containerfile .
```

------------------------------------------------------------------------

## 4️⃣ Export Docker Image

``` bash
docker save custom-erpnext-hrms:v16 -o erpnext-v16-offline.tar
```

Transfer `erpnext-v16-offline.tar` to your Ubuntu server.

------------------------------------------------------------------------

# 🖥 PART 3 --- Back on Ubuntu Server

## 1️⃣ Load Docker Image

``` bash
docker load -i erpnext-v16-offline.tar
```

Verify:

``` bash
docker images
```

You should see:

custom-erpnext-hrms v16

------------------------------------------------------------------------

## 2️⃣ Create compose.yaml

``` bash
nano compose.yaml
```

Paste:

``` yaml
version: "3.9"

services:

  mariadb:
    image: mariadb:10.6
    environment:
      MYSQL_ROOT_PASSWORD: admin
    volumes:
      - mariadb-data:/var/lib/mysql

  redis-cache:
    image: redis:7

  redis-queue:
    image: redis:7

  backend:
    image: custom-erpnext-hrms:v16
    depends_on:
      - mariadb
      - redis-cache
      - redis-queue
    volumes:
      - sites:/home/frappe/frappe-bench/sites

  scheduler:
    image: custom-erpnext-hrms:v16
    command: bench schedule
    depends_on:
      - backend
    volumes:
      - sites:/home/frappe/frappe-bench/sites

  worker:
    image: custom-erpnext-hrms:v16
    command: bench worker
    depends_on:
      - backend
    volumes:
      - sites:/home/frappe/frappe-bench/sites

  frontend:
    image: custom-erpnext-hrms:v16
    command: nginx-entrypoint.sh
    ports:
      - "8080:8080"
    depends_on:
      - backend
    volumes:
      - sites:/home/frappe/frappe-bench/sites

volumes:
  mariadb-data:
  sites:
```

------------------------------------------------------------------------

## 3️⃣ Start Containers

``` bash
docker compose up -d
```

------------------------------------------------------------------------

## 4️⃣ Create ERPNext Site

``` bash
docker compose exec backend bench new-site erp.localhost   --db-root-password admin   --admin-password Admin@123
```

------------------------------------------------------------------------

## 5️⃣ Install Apps

``` bash
docker compose exec backend bench --site erp.localhost install-app erpnext
docker compose exec backend bench --site erp.localhost install-app hrms
```

------------------------------------------------------------------------

## 6️⃣ Access ERPNext

Open in browser:

http://SERVER-IP:8080

Login:

Username: Administrator\
Password: Admin@123

------------------------------------------------------------------------

# ✅ Result

✔ Fully offline-compatible\
✔ No GitHub access required\
✔ Custom ERPNext + HRMS image\
✔ Works on Ubuntu 24.04

------------------------------------------------------------------------

For production (SSL, domain, backups), additional configuration is
required.
