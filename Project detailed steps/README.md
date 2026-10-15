<div align="center">

# 🚀 𝗗𝗲𝘃𝗢𝗽𝘀 𝗖𝗜/𝗖𝗗 𝗣𝗿𝗼𝗷𝗲𝗰𝘁
### 𝘎𝘪𝘵 • 𝘎𝘪𝘵𝘏𝘶𝘣 • 𝘋𝘰𝘤𝘬𝘦𝘳 • 𝘑𝘦𝘯𝘬𝘪𝘯𝘴

*Automated deployment of a static website using a Jenkins Master–Slave architecture, Docker containers and GitHub Webhooks.*

![Git](https://img.shields.io/badge/Git-F05032?style=for-the-badge&logo=git&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04_LTS-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)

</div>

---

## 📑 𝗧𝗮𝗯𝗹𝗲 𝗼𝗳 𝗖𝗼𝗻𝘁𝗲𝗻𝘁𝘀

| # | Section |
|---|---------|
| 1️⃣ | [Project Overview](#1️⃣-project-overview) |
| 2️⃣ | [Architecture & Workflow](#2️⃣-architecture--workflow) |
| 3️⃣ | [Infrastructure Setup](#3️⃣-infrastructure-setup) |
| 4️⃣ | [Dev Server Setup](#4️⃣-dev-server-setup) |
| 5️⃣ | [Jenkins Master Setup](#5️⃣-jenkins-master-setup) |
| 6️⃣ | [GitHub Webhook Configuration](#6️⃣-github-webhook-configuration) |
| 7️⃣ | [Job 1 – Freestyle Project](#7️⃣-job-1--freestyle-project) |
| 8️⃣ | [Testing the Pipeline](#8️⃣-testing-the-pipeline) |
| 9️⃣ | [Jenkins Slave (Node) Setup](#9️⃣-jenkins-slave-node-setup) |
| 🔟 | [Job 2 – Multi-Configuration Project](#-job-2--multi-configuration-project) |
| 1️⃣1️⃣ | [Troubleshooting & Notes](#1️⃣1️⃣-troubleshooting--notes) |

---

## 1️⃣ 𝗣𝗿𝗼𝗷𝗲𝗰𝘁 𝗢𝘃𝗲𝗿𝘃𝗶𝗲𝘄

This project demonstrates a complete **CI/CD workflow**: every time code is pushed to GitHub, Jenkins automatically rebuilds a Docker image and redeploys the website container, on the master node and on a slave node.

| 🧰 Tool | 🎯 Purpose |
|---|---|
| **Git & GitHub** | Version control and source code hosting (also triggers the webhook) |
| **Docker** | Packages the website + Apache web server into a portable container |
| **Jenkins** | Automates build and deployment (Master = controller, Slave = agent) |

---

## 2️⃣ 𝗔𝗿𝗰𝗵𝗶𝘁𝗲𝗰𝘁𝘂𝗿𝗲 & 𝗪𝗼𝗿𝗸𝗳𝗹𝗼𝘄

```
 👨‍💻 Developer
     │  git push
     ▼
 🐙 GitHub Repository ──── Webhook ────►  🏗️ Jenkins Master (port 8080)
                                               │
                              ┌────────────────┴────────────────┐
                              ▼                                 ▼
                    🐳 Docker on Master                 🐳 Docker on Slave
                    (container1 → :5000)                (container1 → :5000)
```

**🔄 Flow:** Push code ➜ GitHub webhook fires ➜ Jenkins job starts ➜ old container removed ➜ new image built ➜ new container runs.

---

## 3️⃣ 𝗜𝗻𝗳𝗿𝗮𝘀𝘁𝗿𝘂𝗰𝘁𝘂𝗿𝗲 𝗦𝗲𝘁𝘂𝗽

Launch **3 EC2 instances** with the following AMI:

> 🖥️ **AMI:** Ubuntu Server 24.04 LTS (HVM), SSD Volume Type

| 🏷️ Instance | 🎭 Role |
|---|---|
| `dev server` | Prepares the website, Dockerfile and pushes code to GitHub |
| `jenkins-master` | Jenkins controller + Docker (runs Job 1 and Job 2) |
| `jenkins-slave` | Jenkins agent node + Docker (runs builds sent by master) |

> 🔐 **Security Group tip:** Open ports `22` (SSH), `8080` (Jenkins) and `5000` (Website) on the Jenkins master/slave instances.

---

## 4️⃣ 𝗗𝗲𝘃 𝗦𝗲𝗿𝘃𝗲𝗿 𝗦𝗲𝘁𝘂𝗽

### 📥 Step 1 – Download the website template

```bash
cd /root

wget https://www.tooplate.com/zip-templates/2130_waso_strategy.zip

apt-get update -y      # (refresh package list)

apt install unzip -y   # (tool required to extract the .zip file)

unzip 2130_waso_strategy.zip
```

### 🐳 Step 2 – Create the Dockerfile

```bash
cd /root/2130_waso_strategy

vim Dockerfile
```

```dockerfile
FROM ubuntu                                              # (base image)
RUN apt-get update -y                                    # (update packages inside image)
RUN apt-get install -y apache2                           # (install Apache web server)
COPY . /var/www/html                                     # (copy website files to Apache's web root)
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]         # (run Apache in foreground so container stays alive)
```

> ⚠️ Remove the `# (...)` comments if your Docker version complains about inline comments — they are only here for explanation.

### 🌱 Step 3 – Push the code to GitHub

```bash
apt install git -y     # (optional – install only if Git is not present)

git init               # (initialize a local Git repository)

git add *              # (stage all files)

git commit -m "Initial commit Waso Strategy Website"

git remote add origin https://github.com/<your-username>/webserver-repo.git   # (link to your GitHub repo)

git push -u origin master   # (push code; use `main` if your default branch is main)
```

---

## 5️⃣ 𝗝𝗲𝗻𝗸𝗶𝗻𝘀 𝗠𝗮𝘀𝘁𝗲𝗿 𝗦𝗲𝘁𝘂𝗽

> 💻 Run on: **Ubuntu_Jenkins_Master**

### ☕ Step 1 – Install Java (Jenkins prerequisite)

```bash
apt update -y

apt install fontconfig openjdk-21-jre -y
```

### 📦 Step 2 – Add Jenkins repository & install

```bash
wget -O /etc/apt/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
# (downloads the repository's security key)

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
# (registers the Jenkins package repository)

apt update -y

apt install jenkins -y

systemctl start jenkins    # (start the Jenkins service)
```

### 🔑 Step 3 – Give Jenkins sudo access

```bash
vim /etc/sudoers
```

Add this line at the end:

```
jenkins ALL=(ALL) NOPASSWD: ALL
```

> 💡 **Why?** Jenkins runs shell commands as the `jenkins` user. This lets it run `sudo docker ...` without a password prompt.

### 🐳 Step 4 – Install Docker (and Git)

```bash
apt install git -y            # (optional)

apt install docker.io -y

systemctl start docker

systemctl enable docker       # (auto-start Docker on every reboot)
```

---

## 6️⃣ 𝗚𝗶𝘁𝗛𝘂𝗯 𝗪𝗲𝗯𝗵𝗼𝗼𝗸 𝗖𝗼𝗻𝗳𝗶𝗴𝘂𝗿𝗮𝘁𝗶𝗼𝗻

On your GitHub repository:

**Settings ➜ Webhooks ➜ Add webhook**

```
Payload URL: http://<jenkins_public_ip>:8080/github-webhook/
```

> 🔔 **What it does:** GitHub notifies Jenkins instantly whenever code is pushed, so builds start automatically. (Keep the trailing `/`.)

---

## 7️⃣ 𝗝𝗼𝗯 𝟭 – 𝗙𝗿𝗲𝗲𝘀𝘁𝘆𝗹𝗲 𝗣𝗿𝗼𝗷𝗲𝗰𝘁

Create **`job1`** in Jenkins:

1. 📂 **Source Code Management** ➜ select **Git** (add your repository URL)
2. ⚡ **Build Triggers** ➜ check **GitHub hook trigger for GITScm polling**
3. 🛠️ **Build Step** ➜ **Execute Shell**

```bash
sudo docker rm -f container1 || true                          # (remove old container; `|| true` ignores error if none exists)
sudo docker build -t webserver-image .                        # (build a fresh image from the Dockerfile)
sudo docker run -itd --name container1 -p 5000:80 webserver-image   # (run container; host port 5000 → container port 80)
```

4. ▶️ **Build manually for the first time.**

🌐 **Access the website:** `http://<Jenkins-Master-Public-IP>:5000`

---

## 8️⃣ 𝗧𝗲𝘀𝘁𝗶𝗻𝗴 𝘁𝗵𝗲 𝗣𝗶𝗽𝗲𝗹𝗶𝗻𝗲

Make a change on the **dev server** after viewing the site:

```bash
vim index.html

git add *

git commit -m "Updated index.html"

git push origin master
```

✅ Jenkins should trigger automatically and the updated site will appear at port `5000` (refresh the browser).

---

## 9️⃣ 𝗝𝗲𝗻𝗸𝗶𝗻𝘀 𝗦𝗹𝗮𝘃𝗲 (𝗡𝗼𝗱𝗲) 𝗦𝗲𝘁𝘂𝗽

### 🖥️ Part A – Prepare the slave instance

> 💻 Run on: **node 1 (slave)**

```bash
apt update -y

apt install fontconfig openjdk-21-jre -y   # (Java is required to run the Jenkins agent)

apt install git -y                          # (optional)

apt install docker.io -y

systemctl start docker

systemctl enable docker
```

### 🔗 Part B – Add the node in Jenkins

**Jenkins Dashboard ➜ Manage Jenkins ➜ Nodes ➜ Add new node**

| ⚙️ Setting | ✏️ Value |
|---|---|
| **Node name** | `Jenkins-slave1` (select **Permanent Agent**) |
| **Remote root directory** | Copy the directory path from the field (workspace location on the slave) |
| **Labels** | `Prod` |
| **Launch method** | *Launch agent by connecting it to the controller* |

Then click **Save**.

> 🔴 The node will show **offline** at first — this is expected.

**Bring the node online:**

1. Open the newly created node.
2. Copy the commands under **"Run from agent command line (Unix)"**.
3. Run them on the **slave instance**. Use the 2nd command with `nohup` and `&` so the agent keeps running in the background:

```bash
nohup <copied-agent-command> &
```

> 🟢 Refresh the Nodes page — the node should now be online.

### 🎯 Part C – Run Job 1 on the slave

**Jenkins ➜ job1 ➜ Configure**

1. ✔️ Select **Restrict where this project can be run**
2. 🏷️ **Label Expression:** `Jenkins-slave1`
3. 💾 **Apply & Save**
4. ▶️ Build manually the first time

> 📌 Now changes are deployed **only on the slave**.

---

## 🔟 𝗝𝗼𝗯 𝟮 – 𝗠𝘂𝗹𝘁𝗶-𝗖𝗼𝗻𝗳𝗶𝗴𝘂𝗿𝗮𝘁𝗶𝗼𝗻 𝗣𝗿𝗼𝗷𝗲𝗰𝘁

> 🌍 Deploys to **both master and slave simultaneously.**

**Jenkins ➜ New Item ➜ Create `job2`**

1. 🧩 Choose **Multi-configuration project**
2. 📂 **Source Code Management** ➜ Git ➜ add the repository URL
3. ⚡ **Triggers** ➜ **GitHub hook trigger for GITScm polling**
4. 🧮 **Configuration Matrix** ➜ **Add axis ➜ Label expression** (also called *Label Expressions*)
5. 🖧 Add the node names: **`Built-in`** and **`Jenkins-slave1`** *(Built-in = the master node)*
6. 🛠️ **Build Step ➜ Execute Shell**

```bash
sudo docker rm -f container1 || true
sudo docker build -t webserver-image .
sudo docker run -itd --name container1 -p 5000:80 webserver-image
```

7. 💾 **Apply & Save**

### ✅ Verify

Update `index.html` and push to GitHub, then check both:

| 🌐 Server | 🔗 URL |
|---|---|
| Jenkins Master | `http://<Jenkins-Master-IP>:5000` |
| Jenkins Slave (Node) | `http://<Node-IP>:5000` |

---

## 1️⃣1️⃣ 𝗧𝗿𝗼𝘂𝗯𝗹𝗲𝘀𝗵𝗼𝗼𝘁𝗶𝗻𝗴 & 𝗡𝗼𝘁𝗲𝘀

| ❗ Issue | 💡 Fix |
|---|---|
| Webhook not triggering | Check port `8080` is open in the Security Group and the payload URL ends with `/github-webhook/` |
| `permission denied` on Docker | Confirm the `jenkins ALL=(ALL) NOPASSWD: ALL` line in `/etc/sudoers` |
| Node stays offline | Re-run the agent command on the slave; ensure Java is installed and the agent port is reachable |
| Port 5000 not reachable | Open port `5000` in the instance's Security Group |
| `git push origin master` fails | Your default branch may be `main` — use `git push origin main` |
| Job runs on wrong node | Check the **label expression** in job configuration |

---

<div align="center">

### 🌟 *Built with ❤️ using Git • GitHub • Docker • Jenkins For Devops community* ⭐

</div>
