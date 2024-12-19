# Spring Petclinic CI/CD Pipeline

This project demonstrates a CI/CD pipeline for the [Spring Petclinic application](https://github.com/spring-projects/spring-petclinic) using Jenkins, Maven, SonarQube, Nexus, and Tomcat. The pipeline automates building, testing, code analysis, artifact storage, and deployment using Jenkins Pipeline as Code.


## 📋 Table of Contents
1. [Architecture](#-architecture)
2. [Technologies Stack](#-technologies-stack)
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

## 🛠️ Technologies Stack

| Category | Technology |
|----------|------------|
| CI/CD Platform | Jenkins |
| Build System | Maven |
| Code Analysis | SonarQube |
| Artifact Repository | Nexus |
| Application Server | Tomcat |
| Testing Framework | JUnit |
| Runtime | Java 17 |
 
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

### Prerequisites
- Ubuntu/Debian-based system
- Docker and Docker Compose
- Java Development Kit 17
- Maven 3.8+

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
rm latest-unix.tar.gz
sudo mv nexus-3.* nexus
```

2. Create systemd service:
```bash
sudo nano /etc/systemd/system/nexus.service
```

3. Add configuration for Nexus service:
```ini
[Unit]
Description=nexus service
After=network.target

[Service]
Type=forking
LimitNOFILE=65536
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

4. Enable and start Nexus:
```bash
sudo systemctl enable nexus
sudo systemctl start nexus
```

5. Get the default admin password:
```bash
cat /opt/nexus/sonatype-work/nexus3/admin.password
```

### Tomcat Installation
1. Download and install Tomcat:
```bash
# Download and install Tomcat 11
wget https://dlcdn.apache.org/tomcat/tomcat-11/v11.0.2/bin/apache-tomcat-11.0.2.tar.gz

# Create service user
sudo useradd -r -m -U -d /opt/tomcat -s /bin/false tomcat

# Extract and configure
tar xzf apache-tomcat-11.0.2.tar.gz
sudo mv apache-tomcat-11.0.2/* /opt/tomcat/

# Set permissions
sudo chown -R tomcat: /opt/tomcat
```

2. Configure Tomcat users (`/opt/tomcat/conf/tomcat-users.xml`):
```xml
<tomcat-users>
    <role rolename="manager-gui" />
    <user username="manager" password="password" roles="manager-gui" />
    <role rolename="admin-gui" />
    <user username="admin" password="password" roles="manager-gui,admin-gui" />
</tomcat-users>
```

3. Create Tomcat service (`/etc/systemd/system/tomcat.service`):
```ini
[Unit]
Description=Apache Tomcat 11 Web Application Server
After=network.target
 
[Service]
Type=forking
 
User=tomcat
Group=tomcat
 
Environment="JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64"
Environment="CATALINA_HOME=/opt/tomcat"
Environment="CATALINA_BASE=/opt/tomcat"
Environment="CATALINA_PID=/opt/tomcat/temp/tomcat.pid"
Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"
 
ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh
 
[Install]
WantedBy=multi-user.target
```

4. Start Tomcat service:
```bash
sudo systemctl daemon-reload
sudo systemctl start tomcat
sudo systemctl status tomcat
```

### SonarQube Setup
Create `docker-compose.yaml`:

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

### Maven Settings

Update `~/.m2/settings.xml`:
```xml
<server>
    <id>deployment</id>
    <username>username</username>     <!-- Nexus username -->
    <password>password</password>     <!-- Nexus password -->
</server>
```

### Project Configuration
Update the `pom.xml` with your Nexus Repository information:

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

### Nexus Repository Configuration

1. Access Nexus web interface
2. Create Release Repository:
   - Type: maven2 (hosted)
   - Name: spring-petclinic-release
   - Version Policy: Release
3. Create Snapshot Repository:
   - Type: maven2 (hosted)
   - Name: spring-petclinic-snap
   - Version Policy: Snapshot

### Jenkinsfile Environment Variables

Update the following in the Jenkinsfile:
- `REMOTE_USER`: Tomcat server username
- `REMOTE_HOST`: Tomcat server hostname
- `REMOTE_PATH`: Temporary file transfer location
- `TOMCAT_PATH`: Tomcat webapps directory
- SonarQube authentication token



## 🚧 Future Enhancements

- [ ] Docker containerization of all components
- [ ] Integrate additional security scanning
- [ ] Create staging and production environment deployments
