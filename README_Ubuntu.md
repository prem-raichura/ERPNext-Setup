# Custom ERPNext & HRMS Docker Setup (Ubuntu 24.04)

This guide explains how to deploy ERPNext v16 & HRMS v16 on Ubuntu 24.04
using your custom Docker Compose configuration from:

-   compose.yaml\
    https://github.com/prem-raichura/ERPNext-Setup/blob/main/compose.yaml

-   example.env\
    https://github.com/prem-raichura/ERPNext-Setup/blob/main/example.env

This setup supports firewall-restricted environments by building the
Docker image on a machine with internet access and transferring it to
the Ubuntu server.

------------------------------------------------------------------------

# 🧱 Architecture Overview

Machine A (Internet Access) → Clone frappe_docker → Build custom image →
Export image (.tar)

Machine B (Ubuntu Server -- Offline) → Install Docker → Copy
compose.yaml + .env → Load image → Start services → Create site →
Install apps

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

## 4️⃣ Export Image

``` bash
docker save custom-erpnext-hrms:v16 -o erpnext-v16-offline.tar
```

Transfer `erpnext-v16-offline.tar` to your Ubuntu server.

------------------------------------------------------------------------

# 🖥 PART 3 --- Back on Ubuntu Server

## 1️⃣ Create Project Directory

``` bash
mkdir erpnext
cd erpnext
```

## 2️⃣ Copy Required Files

Download from your GitHub repo:

``` bash
wget https://raw.githubusercontent.com/prem-raichura/ERPNext-Setup/main/compose.yaml
wget https://raw.githubusercontent.com/prem-raichura/ERPNext-Setup/main/example.env
mv example.env .env
```

(If firewall blocks raw access, download manually on another machine and
transfer them.)

------------------------------------------------------------------------

## 3️⃣ Edit .env

``` bash
nano .env
```

Ensure the following values are set:

``` env
CUSTOM_IMAGE=custom-erpnext-hrms
CUSTOM_TAG=v16
ERPNEXT_VERSION=v16

DB_PASSWORD=StrongDBPassword123
PULL_POLICY=never
RESTART_POLICY=unless-stopped
```

Save and exit.

------------------------------------------------------------------------

## 4️⃣ Load Docker Image

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

## 5️⃣ Start Services

``` bash
docker compose up -d
```

Check containers:

``` bash
docker ps
```

You should see:

-   configurator
-   backend
-   frontend
-   websocket
-   queue-short
-   queue-long
-   scheduler
-   mariadb
-   redis-cache
-   redis-queue

------------------------------------------------------------------------

## 6️⃣ Create ERPNext Site

``` bash
docker compose exec backend bench new-site erp.localhost   --db-root-password StrongDBPassword123   --admin-password Admin@123
```

------------------------------------------------------------------------

## 7️⃣ Install Apps

``` bash
docker compose exec backend bench --site erp.localhost install-app erpnext
docker compose exec backend bench --site erp.localhost install-app hrms
```

------------------------------------------------------------------------

## 8️⃣ Access ERPNext

Open in browser:

http://SERVER-IP:8080

Login:

Username: Administrator\
Password: Admin@123

------------------------------------------------------------------------

# 📁 Final Structure

    erpnext/
    ├── compose.yaml
    ├── .env
    └── erpnext-v16-offline.tar (optional after load)

------------------------------------------------------------------------

# ✅ Result

✔ Uses your exact GitHub compose.yaml\
✔ Uses your example.env configuration\
✔ Fully offline-capable\
✔ Custom ERPNext v16 + HRMS v16 image\
✔ Production-ready architecture

For SSL, domain, and backup configuration, additional setup is required.
