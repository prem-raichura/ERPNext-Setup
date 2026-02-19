# 🚀 ERPNext v16 + HRMS v16 Docker Setup (Ubuntu)

This guide provides a complete **from-scratch setup** for running:

-   ERPNext v16
-   Frappe v16
-   HRMS v16
-   Docker Engine (Ubuntu)

------------------------------------------------------------------------

## 📦 Prerequisites

Install Docker & Git:

``` bash
sudo apt update
sudo apt install docker.io docker-compose-plugin git -y
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

## 1️⃣ Clone Official Frappe Docker

``` bash
git clone https://github.com/frappe/frappe_docker
cd frappe_docker
```

------------------------------------------------------------------------

## 2️⃣ Clone Custom Compose & ENV Files

``` bash
cd ..
git clone https://github.com/prem-raichura/ERPNext-Setup.git
cp ERPNext-Setup/compose.yaml frappe_docker/
cp ERPNext-Setup/example.env frappe_docker/.env
cd frappe_docker
```

------------------------------------------------------------------------

## 3️⃣ Create apps.json

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

## 4️⃣ Build Custom Docker Image

``` bash
docker build \
  --build-arg=PYTHON_VERSION=3.11 \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=version-16 \
  --build-arg=APPS_JSON_BASE64=$(base64 -w 0 apps.json) \
  --tag=custom-erpnext-hrms:v16 \
  --file=images/custom/Containerfile .
```

Wait for build to complete.

------------------------------------------------------------------------

## 5️⃣ Edit .env

``` bash
nano .env
```

Ensure:

``` env
CUSTOM_IMAGE=custom-erpnext-hrms
CUSTOM_TAG=v16
DB_ROOT_PASSWORD=StrongDBPassword123
```

------------------------------------------------------------------------

## 6️⃣ Start Containers

``` bash
docker compose up -d
docker ps
```

------------------------------------------------------------------------

## 7️⃣ Add Local Host Entry

``` bash
sudo nano /etc/hosts
```

Add:

    127.0.0.1 erp.localhost

------------------------------------------------------------------------

## 8️⃣ Create Site

``` bash
docker compose exec backend bench new-site erp.localhost \
  --db-root-password StrongDBPassword123 \
  --admin-password Admin@123
```

------------------------------------------------------------------------

## 9️⃣ Install ERPNext + HRMS

``` bash
docker compose exec backend bench --site erp.localhost install-app erpnext
docker compose exec backend bench --site erp.localhost install-app hrms
```

------------------------------------------------------------------------

## 🔟 Access ERPNext

Open in browser:

    http://erp.localhost:8080

Login with:

-   Username: Administrator
-   Password: Admin@123

------------------------------------------------------------------------

## 📁 Final Folder Structure

    frappe_docker/
    ├── compose.yaml
    ├── .env
    ├── apps.json
    └── images/custom/Containerfile

------------------------------------------------------------------------

## ⚙️ Recommended Versions

-   Python: 3.11
-   MariaDB: 10.6
-   Redis: 7
-   Docker: 24+
-   RAM: Minimum 6--8GB

------------------------------------------------------------------------

## 🎉 Done

You now have a complete Ubuntu Docker setup for ERPNext v16 + HRMS v16.

For production deployment (SSL, domain, backups), additional
configuration is required.
