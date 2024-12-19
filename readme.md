# Spring Petclinic CI/CD Pipeline

This project demonstrates a CI/CD pipeline for the [Spring Petclinic application](https://github.com/spring-projects/spring-petclinic) using Jenkins, Maven, SonarQube, Nexus, and Tomcat. The pipeline automates building, testing, code analysis, artifact storage, and deployment using Jenkins Pipeline as Code.


## 📋 Table of Contents
1. [Architecture](#-architecture)
2. [Technologies](#-technologies)
3. [Prerequisites](#-prerequisites)
4. [Infrastructure Setup](#-infrastructure-setup)
5. [Pipeline Stages](#-pipeline-stages)
6. [Configuration Steps](#-configuration-steps)
7. [Jenkinsfile Explanation](#-jenkinsfile-breakdown)
8. [Future Enhancements](#-future-enhancements)

## 🏗️ Architecture
The project utilizes a modular, scalable architecture with three dedicated Virtual Machines (VMs):

### Components
- **Jenkins VM**: 
  - Orchestrates the entire CI/CD pipeline
  - Runs build and test processes
  - Hosts SonarQube (running as Docker container)
  - Integrates with SonarQube and Nexus
- **Nexus VM**: 
  - Manages and stores build artifacts
  - Provides artifact versioning and distribution
- **Tomcat VM**:
  - Hosts the application server
  - Receives and deploys the final WAR file

## 🛠️ Technologies
- **CI/CD**: Jenkins
- **Build Tool**: Maven
- **Code Quality**: SonarQube
- **Artifact Repository**: Nexus
- **Application Server**: Tomcat
- **Testing**: JUnit
- **Java**: JDK 17

## 📋 Prerequisites
Before setting up the pipeline, ensure you have:
- Three separate VMs for:
  - Jenkins (with Docker for SonarQube)
  - Nexus Repository Manager
  - Tomcat Server
- Jenkins with:
  - JDK 17
  - Maven
  - Docker and Docker Compose
 
## 🔄 Pipeline Stages
The CI/CD pipeline follows these key stages:
![Global CI/CD pipeline](/images/cicd-pipeline.png)

1. **Git Checkout**: Clone repository from GitHub
2. **Compile**: Compile project using Maven
3. **Test**: Run unit tests
4. **SonarQube Analysis**: Analyze code quality
5. **Build**: Package application as WAR
6. **Deploy to Nexus**: Store artifacts
7. **Copy to Tomcat**: Transfer WAR to server
8. **Deploy WebApp**: Activate application

## 🔧 Infrastructure Setup

### Jenkins Installation
1. Update system packages:
```bash
sudo apt update
```

2. Install Jenkins:
```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

### Nexus Installation
1. Download and install Nexus:
```bash
cd /opt
wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
tar -zxvf latest-unix.tar.gz
sudo mv nexus-3.* nexus
sudo mv nexus /opt/nexus

```

2. Create systemd service:
```bash
sudo vim /etc/systemd/system/nexus.service
```

3.  Add configuration for Nexus service
```bash
[Unit]
Description=nexus service
After=network.target

[Service]
Type=forking
LimitNOFILE=65536
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
#User=nexus
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

To enable nexus service at system startup

```bash
sudo systemctl enable nexus
```

To start nexus service using systemctl
```bash
sudo systemctl start nexus
```
THe default username is `admin` and the default password is located in:
```bash
cat /opt/sonatype-work/nexus3/admin.password
```

### Tomcat Installation
1. Install Tomcat:
```bash
wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.15/bin/apache-tomcat-10.1.15.tar.gz
```
2. Create a dedicated user to run Apache Tomcat:
```bash
useradd -r -m -U -d /opt/tomcat -s /bin/false tomcat 
```
3. Extract the downloaded file and move it to `/opt` directory:
```bash
tar xzf apache-tomcat-10.1.15.tar.gz
mv apache-tomcat-10.1.15/* /opt/tomcat/
```

### SonarQube Setup
Create a `docker-compose.yaml` file on the Jenkins VM:

```yaml
services:
  sonarqube:
    image: sonarqube:lts-community
    depends_on:
      - sonar_db
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://sonar_db:5432/sonar
      SONAR_JDBC_USERNAME: sonar
      SONAR_JDBC_PASSWORD: sonar
    ports:
      - "9000:9000"

  sonar_db:
    image: postgres:17.2-alpine3.21
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonar
      POSTGRES_DB: sonar
```

## ⚙️ Configuration Steps

### 1. Maven Settings Configuration
Update your `~/.m2/settings.xml` in the Jenkins VM:

```xml
<server>
    <id>deployment</id>
    <username>username</username>     <!-- Nexus username -->
    <password>password</password>     <!-- Nexus password -->
</server>
```

### 2. Project POM Configuration
Update the `pom.xml` with Nexus repository information:

```xml
<distributionManagement>
  <repository>
    <id>deployment</id>
    <name>Internal Releases</name>
    <url>http://<your-tomcat-server-ip>:8081/repository/spring-petclinic-release/</url>
  </repository>
  <snapshotRepository>
    <id>deployment</id>
    <name>Internal Snapshot Releases</name>
    <url>http://<your-tomcat-server-ip>:8081/repository/spring-petclinic-snap/</url>
  </snapshotRepository>
</distributionManagement>
```


### 2. Jenkinsfile Environment Variables

Update the following in the Jenkinsfile:
- `REMOTE_USER`: Tomcat server username
- `REMOTE_HOST`: Tomcat server hostname
- `REMOTE_PATH`: Temporary file transfer location
- `TOMCAT_PATH`: Tomcat webapps directory
- SonarQube authentication token



## 🚧 Future Enhancements

- [ ] Containerize infrastructure components using Docker
- [ ] Integrate additional security scanning
- [ ] Create staging and production environment deployments
