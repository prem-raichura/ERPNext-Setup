# 🚀 ERPNext v16 & HRMS v16 Docker Setup (Ubuntu 24.04)

This guide explains how to deploy ERPNext v16 and HRMS v16 on Ubuntu
24.04 using Docker.

This setup supports:

-   Fully offline deployment
-   Custom Docker image build
-   Firewall-restricted environments
-   Production-ready architecture
-   Local repository-based build (no second PC required)

------------------------------------------------------------------------

# 📌 Architecture Overview

You have two possible methods:

## Method 1 --- With Internet (Single Machine)

1.  Clone repositories
2.  Build Docker image
3.  Save image as `.tar`
4.  Use offline forever

## Method 2 --- Fully Offline Deployment

1.  Build image once (with internet)
2.  Export `.tar`
3.  Transfer to offline Ubuntu server
4.  Load image
5.  Start services
6.  Create site
7.  Install apps

------------------------------------------------------------------------

# 🖥 PART 1 --- Install Docker (Ubuntu 24.04)

``` bash
sudo apt update
sudo apt upgrade -y
```

Install Docker:

``` bash
sudo apt install ca-certificates curl gnupg -y

sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

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

# 💻 PART 2 --- Build Custom ERPNext + HRMS Image

## 1️⃣ Clone Required Repositories

``` bash
git clone -b version-16 https://github.com/frappe/frappe
git clone -b version-16 https://github.com/frappe/erpnext
git clone -b version-16 https://github.com/frappe/hrms
```

## 2️⃣ Clone frappe_docker

``` bash
git clone https://github.com/frappe/frappe_docker
cd frappe_docker
```

## 3️⃣ Add Local Apps

``` bash
mkdir local-apps
cp -r ../frappe local-apps/
cp -r ../erpnext local-apps/
cp -r ../hrms local-apps/
```

## 4️⃣ Create apps.json

``` json
[
  {
    "url": "/workspace/local-apps/erpnext",
    "branch": "version-16"
  },
  {
    "url": "/workspace/local-apps/hrms",
    "branch": "version-16"
  }
]
```

## 5️⃣ Modify Containerfile

Add before bench init:

    COPY local-apps /workspace/local-apps

## 6️⃣ Build Custom Image

``` bash
docker build \
  --build-arg=PYTHON_VERSION=3.11 \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=version-16 \
  --build-arg=APPS_JSON_BASE64=$(base64 -w 0 apps.json) \
  --tag=custom-erpnext-hrms:v16 \
  --file=images/custom/Containerfile .
```

## 7️⃣ Export Image

``` bash
docker save custom-erpnext-hrms:v16 -o erpnext-v16-offline.tar
```

------------------------------------------------------------------------

# 🖥 PART 3 --- Offline Server Setup

## 1️⃣ Create Project Directory

``` bash
mkdir erpnext
cd erpnext
```

## 2️⃣ Download compose.yaml & example.env

``` bash
wget https://raw.githubusercontent.com/prem-raichura/ERPNext-Setup/main/compose.yaml
wget https://raw.githubusercontent.com/prem-raichura/ERPNext-Setup/main/example.env
mv example.env .env
```

## 3️⃣ Edit .env

``` env
CUSTOM_IMAGE=custom-erpnext-hrms
CUSTOM_TAG=v16
ERPNEXT_VERSION=v16

DB_PASSWORD=StrongDBPassword123
PULL_POLICY=never
RESTART_POLICY=unless-stopped
```

## 4️⃣ Load Docker Image

``` bash
docker load -i erpnext-v16-offline.tar
docker images
```

## 5️⃣ Start Services

``` bash
docker compose up -d
docker ps
```

## 6️⃣ Create ERPNext Site

``` bash
docker compose exec backend bench new-site erp.localhost   --db-root-password StrongDBPassword123   --admin-password Admin@123
```

## 7️⃣ Install Apps

``` bash
docker compose exec backend bench --site erp.localhost install-app erpnext
docker compose exec backend bench --site erp.localhost install-app hrms
```

------------------------------------------------------------------------

# 🌐 Access ERPNext

Open in browser:

http://SERVER-IP:8080

Login:

Username: Administrator\
Password: Admin@123

------------------------------------------------------------------------

# 📁 Final Project Structure

    erpnext/
    ├── compose.yaml
    ├── .env
    └── erpnext-v16-offline.tar

------------------------------------------------------------------------

# 🔒 Production Recommendations

-   Use domain + reverse proxy (Nginx)
-   Enable SSL (Let's Encrypt)
-   Configure automatic backups
-   Change Administrator password immediately
-   Restrict firewall ports

------------------------------------------------------------------------

# ✅ Result

✔ ERPNext v16\
✔ HRMS v16\
✔ Custom Docker image\
✔ Fully offline-capable\
✔ Ubuntu 24.04 compatible\
✔ Production-ready structure

------------------------------------------------------------------------

---

## 🧑‍💻 Author

[**Prem Raichura**](https://portfolio-prem-raichura.vercel.app/)
* **GitHub:** [prem-raichura](https://github.com/prem-raichura)
* **LinkedIn:** [prem-raichura](https://www.linkedin.com/in/prem-raichura/)
