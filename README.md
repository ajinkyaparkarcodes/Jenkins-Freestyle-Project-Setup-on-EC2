# Jenkins-Tomcat-EC2-Deployment

This project sets up a complete CI/CD pipeline using Jenkins on AWS EC2 (Amazon Linux 2) to deploy a sample Maven web application to Apache Tomcat running on the same EC2 instance.

---

## 🔧 Technologies Used

- Amazon EC2 (Amazon Linux 2)
- Jenkins
- Apache Tomcat 9
- Git & GitHub
- Maven
- Java 17 (Amazon Corretto)

---

## 🚀 Setup Steps

### 1. Launch EC2 Instance

- Use Amazon Linux 2 AMI (t2.micro - Free Tier eligible)
- Configure Security Group:
  - Port 22 – SSH
  - Port 8080 – Jenkins
  - Port 9090 – Tomcat

---

### 2. Install Java & Jenkins

```bash
sudo su
sudo rpm --import https://yum.corretto.aws/corretto.key
sudo curl -Lo /etc/yum.repos.d/corretto.repo https://yum.corretto.aws/corretto.repo
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum install java-17-amazon-corretto -y
sudo yum install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

---

### 3. Setup Jenkins UI

- Visit `http://<EC2-PUBLIC-IP>:8080`
- Unlock Jenkins:
  ```bash
  sudo cat /var/lib/jenkins/secrets/initialAdminPassword
  ```
- Install suggested plugins and create an admin user

---

### 4. Install Git, JDK & Maven

```bash
sudo yum install git -y
```

- Jenkins → Manage Jenkins → Global Tool Configuration:
  - Add JDK and Maven

---

### 5. Install Required Plugins

Jenkins → Manage Plugins:
- ✅ `Deploy to container`
- ✅ `GitHub Integration`

---

### 6. Setup Apache Tomcat

```bash
wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.106/bin/apache-tomcat-9.0.106.tar.gz
tar -xvf apache-tomcat-9.0.106.tar.gz
cd apache-tomcat-9.0.106/
```

Edit `conf/server.xml`:
```xml
<Connector port="9090" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
```

Edit `conf/tomcat-users.xml`:
```xml
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<role rolename="admin-gui"/>
<user username="admin" password="admin" roles="manager-gui,admin-gui,manager-script"/>
```

Edit `webapps/manager/META-INF/context.xml`:
```xml
<Valve className="org.apache.catalina.valves.RemoteAddrValve" allow=".*" />
```

Restart Tomcat:
```bash
cd bin/
./shutdown.sh
./startup.sh
```

---

### 7. Create Jenkins Freestyle Project

- Go to Jenkins Dashboard → New Item → Freestyle Project
- Name: `Maven-WebApp-Build`
- Source Code Management:
  - Git Repository: `https://github.com/ajinkyaparkarcodes/webapp-maven-template.git`
  - Branch: `main`
- Build Step:
  - Goals: `clean package`
- Post-build Action:
  - Deploy WAR to container
    - WAR Files: `**/*.war`
    - Container: Tomcat 9.x
    - URL: `http://<EC2-IP>:9090`
    - Credentials: `admin / admin`

---

### 8. Enable GitHub Webhook

- GitHub → Repo → Settings → Webhooks → Add:
  - Payload URL: `http://<EC2-IP>:8080/github-webhook/`
  - Content Type: `application/json`
  - Events: Just the push event
  - Save

- Jenkins Job → Configure:
  - ✅ Check: "GitHub hook trigger for GITScm polling"

---

### ✅ Final Test

Visit in browser:
```
http://<EC2-IP>:9090/Maven-WebApp-Build
```

---

## 📂 Sample Git Repo

[Maven WebApp Template](https://github.com/ajinkyaparkarcodes/webapp-maven-template.git)

---
