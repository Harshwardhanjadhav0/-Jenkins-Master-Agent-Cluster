# 🚀 Jenkins Cluster Setup Using Docker & Kubernetes (Manual, No Official Image)

## 👨‍💻 Project Overview

This guide covers the complete manual setup of a Jenkins Cluster (Master-Agent architecture) using Docker containers. The Jenkins master and agent are both installed from scratch on Ubuntu-based Docker containers — no official Jenkins image used. This setup is ideal for learning the internals of Jenkins and how it interacts with Docker networks.

---

## 🖥️ Step 1: Launch AWS EC2 Instance

- **Instance Type:** `t3.medium`
- **OS:** Ubuntu 22.04 LTS
- **Storage:** 20 GiB
- **Security Group:** Inbound Rule → `TCP` → Port `8080` (to allow Jenkins web UI)

---

## 🐳 Step 2: Install Docker (from Official DockerHub Docs)
### Update and install required dependencies
```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
```
---
### Add Docker GPG key
```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
---
### Add Docker apt repository
```bashecho \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo \"${UBUNTU_CODENAME:-$VERSION_CODENAME}\") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
---
### Install Docker
```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
---
### Post-install: run Docker as non-root
```bash
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```
---
### Test Docker
```bash
docker run hello-world
```
---
## 🧱 Step 3: Create Jenkins Docker Network & Master Container
### Create Docker network for Jenkins cluster
```bash
docker network create jenkins
```
---
### Launch Jenkins master container (Ubuntu base)
```bash
docker run -dit --name jenkins-master -p 8080:8080 --network jenkins ubuntu:22.04
```
---
### Enter container shell
```bash
docker exec -it jenkins-master bash
```
---
## ☕ Step 4: Install Java 18+ and Jenkins Manually
### Inside jenkins-master container
```bash
apt-get update
apt-get install -y wget curl git unzip nano software-properties-common
```
---
### Add OpenJDK PPA
```bash
add-apt-repository ppa:openjdk-r/ppa
apt-get update
apt-get install -y openjdk-18-jdk
```
---
### Verify Java
```bash
java -version
```
---
### Download Jenkins .war file
```bash
mkdir -p /opt/jenkins && cd /opt/jenkins
wget https://get.jenkins.io/war-stable/latest/jenkins.war
```
---
### Start Jenkins
```bash
java -jar jenkins.war --enable-future-java
```
---
## 🌐 Step 5: Access Jenkins UI
Open in browser: http://<EC2_PUBLIC_IP>:8080
Unlock with:
```bash
cat /root/.jenkins/secrets/initialAdminPassword
```
- Install suggested plugins
- Create admin user
- Access the Jenkins Dashboard
---
## 🤖 Step 6: Create Jenkins Agent (Docker Container)
### Launch Jenkins agent container
```bash
docker run -dit --name jenkins-agent-1 --network jenkins ubuntu:22.04
docker exec -it jenkins-agent-1 bash
```
---
### Inside agent container
```bash
apt update
apt install -y software-properties-common
add-apt-repository ppa:openjdk-r/ppa
apt update
apt install -y openjdk-18-jdk wget git curl unzip nano
```
---
### Setup Jenkins agent directory
```bash
mkdir -p /home/jenkins/agent
cd /home/jenkins/agent
```
---
### Download Jenkins agent jar
```bash
wget http://jenkins-master:8080/jnlpJars/agent.jar
```
---
## ⚙️ Step 7: Connect Agent in Jenkins UI
Navigate to:
Jenkins → Manage Jenkins → Nodes → New Node
Set configuration:
- Name: agent-1
- Type: Permanent Agent
- Remote root dir: /home/jenkins/agent
- Labels: agent-1
- Usage: Use this node as much as possible
- Launch Method: Launch agent by connecting it to the controller
- Jenkins will show a command like:
```bash
For example: java -jar agent.jar -url http://<JENKINS_MASTER_IP>:8080/ \
-secret <SECRET_KEY> -name "agent-1" -workDir "/home/jenkins/agent"
```
### Run it inside jenkins-agent-1 container:
```bash
cd /home/jenkins/agent
java -jar agent.jar -url http://jenkins-master:8080/ \
-secret <SECRET_KEY> -name "agent-1" -workDir "/home/jenkins/agent"
```
- ✅ Jenkins agent will show as "Connected" in the UI!
---
## 🧪 Step 8: Test Jenkins Agent with Job
- Create a new Freestyle Job
- Restrict to label: agent-1
- Add shell build step:
```bash
echo "Running on Jenkins Agent 1"
hostname
whoami
```
- Save and build → Check Console Output
---
### 📦 What's Next?
I'm extending this project to run Jenkins agents inside Kubernetes pods for dynamic scaling and high availability. 

### 💡 Final Thoughts
This project demonstrates:
- Manual Jenkins installation without official Docker image
- Secure, stable master-agent CI setup using Docker
- Real-world networking, Java installation, and agent bootstrapping
- Future-ready architecture for Kubernetes-native CI/CD

🔗 Stay connected for more DevOps builds and real-world automation!
Feel free to ⭐ the repo and try it yourself.

