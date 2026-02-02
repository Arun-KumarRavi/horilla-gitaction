# 🚀 Horilla CI/CD Deployment Guide

This guide provides a comprehensive, step-by-step walkthrough to set up a professional CI/CD pipeline for the Horilla HRMS. We will go from a local environment to a live production server on AWS EC2 using GitHub Actions and Docker.

---

## 🛠 Prerequisites

Before starting, ensure you have the following:
1.  **A Docker Hub Account**: [Create one here](https://hub.docker.com/signup).
2.  **An AWS Account**: With an active **Ubuntu EC2 Instance**.
3.  **Local Git**: Installed and configured on your machine.
4.  **SSH Access**: Your `.pem` key file for the EC2 instance.

---

## 🏗 Step 1: Prepare the EC2 Instance

Connect to your Ubuntu EC2 via SSH and run these commands to install the necessary infrastructure.

### 1.1 Install Docker
```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
# Allow the 'ubuntu' user to run docker commands without sudo
sudo usermod -aG docker ubuntu
```
> [!IMPORTANT]
> **Logout and log back in** after running the `usermod` command for the changes to take effect.

### 1.2 Install Docker Compose
```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
# Verify installation
docker-compose --version
```

---

## 🔐 Step 2: Configure GitHub Secrets

This is the most critical step for security. Do **not** hardcode passwords in your workflow files.

1.  Navigate to your GitHub Repository.
2.  Go to **Settings > Secrets and variables > Actions**.
3.  Click **New repository secret** and add the following:

| Secret Name | How to get it |
| :--- | :--- |
| `DOCKERHUB_USERNAME` | Your Docker Hub ID. |
| `DOCKERHUB_TOKEN` | Create at [Docker Hub Settings > Security](https://hub.docker.com/settings/security). |
| `EC2_HOST` | The **Public IPv4 address** of your EC2 instance. |
| `EC2_USER` | Usually `ubuntu`. |
| `EC2_SSH_KEY` | Open your `.pem` file and copy **everything** (including the BEGIN/END headers). |

---

## 🛡 Step 3: Expose Horilla (AWS Security Group)

Your EC2 instance blocks all web traffic by default. You must "Expose" port 8000.

1.  Open the **AWS EC2 Console**.
2.  Select your instance and look for the **Security** tab.
3.  Click on the active **Security Group**.
4.  Click **Edit inbound rules** and add these two:
    - **Rule 1**: type `Custom TCP`, Port `8000`, Source `0.0.0.0/0`.
    - **Rule 2**: Type `SSH`, Port `22`, Source `0.0.0.0/0` (or your specific IP).

---

## 💻 Step 4: The Local Git Workflow

Now, push your code to trigger the automation.

```bash
# 1. Initialize git (if not done)
git init

# 2. Add your GitHub as the remote
git remote add origin https://github.com/<YOUR_USERNAME>/<YOUR_REPO_NAME>.git

# 3. Add files and push
git add .
git commit -m "Deploy: Setup CI/CD pipeline"
git branch -M master
git push -u origin master
```

---

## 🔄 Step 5: How the Automation Works

The moment you push to `master`, GitHub Actions performs these steps:

1.  **Checkout**: Grabs your latest code.
2.  **Build**: Creates a Docker Image of the Django app.
3.  **Push**: Sends that image to your Docker Hub repository.
4.  **Deploy (SSH)**:
    - GitHub logs into your EC2 automatically.
    - It creates/updates a `docker-compose.yml` on the server.
    - It pulls the new image and restarts the containers.

---

## 🏁 Step 6: Verify and Access

1.  Monitor the **Actions** tab in GitHub until you see a green checkmark ✅.
2.  Open your browser and navigate to: `http://<YOUR_EC2_IP>:8000`.
3.  **Initial Login**:
    - **Username**: `admin`
    - **Password**: `admin`

---

## 📋 Troubleshooting

- **CSS/Images not loading?** Horilla uses WhiteNoise to serve static files within the container. If they don't load, check the logs with `docker logs <container_id>`.
- **Permission Denied (SSH)?** Ensure the `EC2_SSH_KEY` in GitHub Secrets is exactly the content of your private key.
- **Port 8000 timing out?** Double-check the **AWS Inbound Rules** and make sure nothing is blocking port 8000.
