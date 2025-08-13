# 🚀 Continuous Integration Pipeline Setup of Java Application Project on AWS EC2 with Jenkins, Docker, Maven, Trivy and SonarQube

Welcome to the **CI Pipeline Lab**! This project demonstrates how to set up a continuous integration pipeline of a java application using **Jenkins**, **Docker**, **SonarQube**, and **Trivy** on an **AWS Ubuntu EC2 instance**. Follow these steps carefully to get your CI environment up and running.

---

## 🖥️ Step 1: Create the EC2 Instance

- **EC2 Type:** Ubuntu `t2.medium`
- **EBS Volume:** 30 GB
- **Region:** `us-east-1`

![Instance 1](Images/Instance-1.png)

![Instance 2](Images/instance-2.png)

> Launch an EC2 instance in your AWS account. We choose `t2.medium` for moderate compute power and a 30 GB volume for storage.

![Instance 3](Images/instance-3.png)

---

## 🔗 Step 2: Connect to EC2 using the AWS Console and Login as Root

```bash
sudo su
```

![Connect](Images/connect-1.png)

---

## 🛠️ Step 3: Install Jenkins on Ubuntu

```bash
apt update -y
apt upgrade -y
apt install openjdk-17-jre -y

```

> Updates the package lists, upgrades existing packages, and installs Java 17 runtime, which Jenkins requires.

![Jenkins 1](Images/jenkins-1.png)

```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null

```

> Downloads and adds the Jenkins GPG (GNU Privacy Guard) key for secure package verification.

![Jenkins 2](Images/jenkins-2.png)

```bash
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

```

> Adds the Jenkins repository to Ubuntu so you can install the latest stable version.

![Jenkins 3](Images/jenkins-3.png)

```bash
apt-get update -y
apt-get install jenkins -y

```

> Updates the package lists again and installs Jenkins.

![Jenkins 4](Images/jenkins-4.png)

---

## 🔒 Step 4: Update Security Group

- Edit the inbound rules for your EC2 instance
- Add **All traffic** from **Anywhere**

> This allows Jenkins and other tools to be accessed from your local browser.

![Inbound Rule](Images/inboundrule.png)

---

## 🌐 Step 5: Access Jenkins Console

Open your browser and go to:

```
http://<EC2_PUBLIC_IP>:8080

```

> Jenkins web interface runs on port `8080`.

![Jenkins Console](Images/jenkinsconsole.png)

---

## 🔑 Step 6: Get Administrator Password

- On aws console paste the command below

```bash
cat /var/lib/jenkins/secrets/initialAdminPassword

```

![Secret key](Images/secretjenkins.png)

> Retrieves the initial password required to unlock Jenkins.

![Secret key 1](Images/secretjenkins-1.png)

---

## ⚙️ Step 7: Install Suggested Plugins

> Jenkins will recommend essential plugins. Install all suggested plugins for smoother setup.

![Install Plugin](Images/installplugin.png)

![Install Plugin](Images/installplugin-1.png)

---

## 👤 Step 8: Create First User

> Create an admin user to manage Jenkins securely.

![Create Admin](Images/createadmin.png)

![Create Admin](Images/createadmin-1.png)

![Start](Images/start.png)

![Success](Images/successjenkins.png)

---

## 📦 Step 9: Create a Pipeline Job

> Navigate to Jenkins Dashboard -> New Item -> Pipeline and create your first CI pipeline.

---

## 🔗 Step 10: Configure SCM (GitHub)

- Repository URL: [Java-App-Sample-1](https://github.com/mirjSolution/Java-App-Sample-1)

> Jenkins will pull source code from GitHub to run your CI pipeline.

---

## 🔌 Step 11: Install Plugins

Navigate to:

```
Dashboard -> Manage Jenkins -> Available Plugins

```

Install plugins for:

- SonarQube: Sonar Gerrit, SonarQube Scanner, SonarQube Generic Coverage, Sonar Quality Gates, Quality Gates
- Artifactory/JFrog

> These plugins integrate code quality and artifact management into your pipeline.

---

## 🐳 Step 12: Setup Docker

```bash
apt update -y
apt install apt-transport-https ca-certificates curl software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable" -y
apt update -y
apt-cache policy docker-ce -y
apt install docker-ce -y
systemctl status docker
chmod 777 /var/run/docker.sock
docker -v

```

> Installs Docker CE (Community Edition), checks its status, allows non-root access to Docker, and verifies the installation.

---

## 🟢 Step 13: Install SonarQube

```bash
docker run -d --name sonarqube -p 9000:9000 -p 9092:9092 sonarqube

```

> Runs SonarQube in a Docker container, exposing ports `9000` and `9092` for web access.

### Start Container if Down

```bash
docker ps -a
docker start <containerID>

```

> Lists all Docker containers and starts SonarQube if it’s not running.

### Login to SonarQube

- **Username:** `admin`
- **Password:** `admin`

> Access the SonarQube dashboard to manage code quality.

### Create Sonar Token for Jenkins

Navigate: `Sonar Dashboard -> Administration -> My Account -> Security -> Create token`

> Generate a token for Jenkins integration.

### Integrate SonarQube with Jenkins

Navigate: `Sonar Dashboard -> Administration -> Configuration -> Webhooks -> Add` and add:

```
http://<EC2_IP>:8080/sonarqube-webhook/

```

> Enables automatic SonarQube scans after Jenkins pipeline runs.

---

## 📦 Step 14: Install Maven

```bash
apt update -y
apt install maven -y
mvn -version

```

> Maven is a build automation tool for Java. This installs it and verifies the version.

---

## 🔍 Step 15: Install Trivy for Docker Image Scanning

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy

```

> Trivy scans Docker images for vulnerabilities to ensure secure deployments.

---

## 🔧 Step 16: Configure Jenkins Integrations

### Add SonarQube

- Manage Jenkins -> Configure System -> SonarQube servers
- Add URL and token (from Step 13.3)
- Name token: `sonarqube-api`

### Add Docker Hub Credentials

- Jenkins Dashboard -> Manage Jenkins -> Credentials -> System -> Global credentials
- Add Docker Hub credentials
- Name: `docker`

### Add Jenkins Shared Library

- Manage Jenkins -> Configure System -> Global Pipeline Library
- Name: `my-shared-library`
- Default Version: `main`
- Git: [JENKINS-SHARED-LIBRARY](https://github.com/mirjSolution/JENKINS-SHARED-LIBRARY)

> These configurations integrate all tools into Jenkins for a complete CI/CD pipeline.

---

## ✅ Step 17: Run Pipeline and Verify

- Check Jenkins logs for build status
- Verify Trivy scans for vulnerabilities
- Review SonarQube dashboard for code quality report

> After running the pipeline, ensure all integrated tools work correctly and the CI/CD process is seamless.

---

### 📚 References

- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Docker Documentation](https://docs.docker.com/)
- [SonarQube Documentation](https://docs.sonarqube.org/latest/)
- [Trivy Documentation](https://aquasecurity.github.io/trivy/)

---

💡 **Tip:** Always keep your EC2 security group and Docker containers secured when exposing services to the internet.
