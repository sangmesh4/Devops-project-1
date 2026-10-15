
<div align="center">

# 🚀 𝗗𝗲𝘃𝗢𝗽𝘀 𝗖𝗜/𝗖𝗗 𝗣𝗶𝗽𝗲𝗹𝗶𝗻𝗲
### *Git • GitHub • Docker • Jenkins*

![Git](https://img.shields.io/badge/Git-F05032?style=for-the-badge&logo=git&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04_LTS-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)

**Push code to GitHub → Jenkins is triggered by webhook → Docker builds & redeploys your website automatically. ⚡**

</div>

---

## 📑 Table of Contents

| # | Section |
|---|---------|
| 1️⃣ | [Project Overview](#1️⃣-project-overview) |
| 2️⃣ | [Architecture](#2️⃣-architecture) |
| 3️⃣ | [Infrastructure Setup](#3️⃣-infrastructure-setup) |
| 4️⃣ | [Dev Server Setup](#4️⃣-dev-server-setup) |
| 5️⃣ | [Jenkins Master Setup](#5️⃣-jenkins-master-setup) |
| 6️⃣ | [GitHub Webhook Configuration](#6️⃣-github-webhook-configuration) |
| 7️⃣ | [Create Jenkins Job](#7️⃣-create-jenkins-job) |
| 8️⃣ | [Test the CI/CD Pipeline](#8️⃣-test-the-cicd-pipeline) |
| 9️⃣ | [Jenkins Slave (Node 1) Setup](#9️⃣-jenkins-slave-node-1-setup) |

---

## 1️⃣ Project Overview

📌 This project builds an **automated deployment pipeline (CI/CD)** for a website, using **Git, GitHub, Docker and Jenkins**.

### 🎯 The Goal

> When you change the website's code and push it to GitHub, **the live site updates by itself**. Nobody has to log in to a server and redeploy anything. ✨

### 🧰 Tools Used

| Tool | Purpose |
|------|---------|
| 🐙 **Git & GitHub** | Version control & source code hosting |
| 🐳 **Docker** | Containerizing the Apache web server |
| 🛠️ **Jenkins** | Automation server for build & deploy |
| ☁️ **AWS EC2** | Hosting the servers |

### ⚙️ How It Works

| # | Stage | What Happens |
|:-:|-------|--------------|
| 1️⃣ | 🖥️ **Dev Server** | Download a website code (**Waso Strategy** from Tooplate), write a **Dockerfile** that packages the site into an **Apache web server container**, and push everything to a GitHub repository. |
| 2️⃣ | 🔔 **GitHub Webhook** | GitHub notifies Jenkins every time new code is pushed. |
| 3️⃣ | 🛠️ **Jenkins Master** | On notification, Jenkins pulls the latest code, then runs a shell script that **removes the old container**, **builds a fresh Docker image**, and **starts a new container on port 5000**. |
| 4️⃣ | 🌍 **Result** | The updated website is live at `http://<Jenkins-Master-Public-IP>:5000` within moments of your push. |
| 5️⃣ | 🤖 **Jenkins Slave** | A second server is set up with **Java and Docker** so it can act as an **agent node**. In larger setups, this lets Jenkins hand builds to other machines instead of running everything on the master. |

### 🎓 What You Learn

- 🐙 Version control with **Git and GitHub**
- 🐳 Containerizing an app with **Docker**
- 🔔 Triggering builds automatically with **webhooks**
- 📝 Writing a basic **Jenkins job**
- ☁️ Running a **Jenkins master-agent setup** on AWS EC2

> 💼 A great **beginner portfolio project**: it shows the full flow from code change to live deployment in just a few steps.

---

## 2️⃣ Architecture

```text
   👨‍💻 Developer
        │  git push
        ▼
   🐙 GitHub Repository
        │  Webhook (/github-webhook/)
        ▼
   🛠️ Jenkins Master  ──────►  🖥️ Jenkins Slave (Node 1)
        │
        │  docker build & run
        ▼
   🐳 Docker Container (Apache2)
        │  Port 5000 → 80
        ▼
   🌐 http://<Jenkins-Master-Public-IP>:5000
```

---

## 3️⃣ Infrastructure Setup

Launch **3 EC2 instances** with the following configuration:

| 🖥️ Server | Role |
|-----------|------|
| `dev server` | Developer machine – website source code & Dockerfile |
| `jenkins-master` | Jenkins controller – builds & deploys |
| `jenkins-slave` | Jenkins agent node |

> 💿 **AMI:** `Ubuntu Server 24.04 LTS (HVM), SSD Volume Type`

---

## 4️⃣ Dev Server Setup

### 📥 Step 1 – Download & extract the website template

```bash
cd /root

wget https://www.tooplate.com/zip-templates/2130_waso_strategy.zip

apt-get update -y

apt install unzip -y

unzip 2130_waso_strategy.zip
```

### 🐳 Step 2 – Create the Dockerfile

```bash
cd /root/2130_waso_strategy

vim Dockerfile
```

```dockerfile
FROM ubuntu
RUN apt-get update -y
RUN apt-get install -y apache2
COPY . /var/www/html
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

### 🐙 Step 3 – Push the code to GitHub

```bash
apt install git -y   # (optional)

git init

git add *

git commit -m "Initial commit Waso Strategy Website"

git remote add origin https://github.com/<your-username>/webserver-repo.git

git push -u origin master
```

---

## 5️⃣ Jenkins Master Setup

> 🖥️ Run on: **Ubuntu_Jenkins_Master**

### ☕ Step 1 – Install Java

```bash
apt update -y

apt install fontconfig openjdk-21-jre -y
```

### 🛠️ Step 2 – Install Jenkins

```bash
wget -O /etc/apt/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

apt update -y

apt install jenkins -y

systemctl start jenkins
```

### 🔐 Step 3 – Grant sudo access to the Jenkins user

```bash
vim /etc/sudoers
```

Add the following line:

```text
jenkins ALL=(ALL) NOPASSWD: ALL
```

### 🐳 Step 4 – Install Git & Docker

```bash
apt install git -y   # (optional)

apt install docker.io -y

systemctl start docker

systemctl enable docker
```

---

## 6️⃣ GitHub Webhook Configuration

🔗 On your GitHub repo:

**Settings ➜ Webhooks ➜ Add webhook**

```text
http://jenkins_public_ip:8080/github-webhook/
```

---

## 7️⃣ Create Jenkins Job

📝 Create **`job1`** in Jenkins with the following settings:

| ⚙️ Setting | ✅ Value |
|-----------|---------|
| **Source Code Management** | Git |
| **Build Trigger** | ☑️ GitHub hook trigger for GITScm polling |
| **Build Step** | Execute Shell |

### 💻 Execute Shell script

```bash
sudo docker rm -f container1 || true
sudo docker build -t webserver-image .
sudo docker run -itd --name container1 -p 5000:80 webserver-image
```

### 🌐 Access the website

```text
http://<Jenkins-Master-Public-IP>:5000
```

> 🔨 **Note:** Build manually for the first time.

---

## 8️⃣ Test the CI/CD Pipeline

✏️ Make changes after accessing the web server:

```bash
vim index.html

git add *

git commit -m "Updated index.html"

git push origin master
```

🎉 Jenkins will automatically detect the push, rebuild the image, and redeploy the container!

---

## 9️⃣ Jenkins Slave (Node 1) Setup

> 🖥️ Run on: **node 1 (slave)**

```bash
apt update -y

apt install fontconfig openjdk-21-jre -y

apt install git -y   # (optional)

apt install docker.io -y

systemctl start docker

systemctl enable docker
```

---

## ✅ Workflow Summary

| Step | Action |
|:----:|--------|
| 1️⃣ | Developer pushes code to GitHub |
| 2️⃣ | GitHub webhook notifies Jenkins |
| 3️⃣ | Jenkins pulls the latest code |
| 4️⃣ | Docker image is rebuilt |
| 5️⃣ | Old container removed, new one started on port **5000** |
| 6️⃣ | Website is live with the latest changes 🌍 |

---

<div align="center">


### ⭐ **Made with ❤️ for the DevOps community** ⭐

</div>
