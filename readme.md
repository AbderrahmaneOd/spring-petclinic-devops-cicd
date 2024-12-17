# Spring Petclinic CI/CD Pipeline

This project demonstrates a CI/CD pipeline for the [Spring Petclinic application](https://github.com/spring-projects/spring-petclinic) using Jenkins, Maven, SonarQube, Nexus, and Tomcat. The pipeline automates building, testing, code analysis, artifact storage, and deployment using Jenkins Pipeline as Code.


## 📋 Table of Contents

1. [Architecture](#-architecture)
2. [Technologies](#-technologies)
3. [Prerequisites](#-prerequisites)
4. [Pipeline Stages](#-pipeline-stages)
5. [Setup Instructions](#-setup-instructions)
6. [Jenkinsfile Explanation](#-jenkinsfile-breakdown)
7. [Future Enhancements](#-future-enhancements)

## 🏗️ Architecture

The project utilizes a modular, scalable architecture with dedicated Virtual Machines (VMs) for different components:

### Components
- **Jenkins VM**: 
  - Orchestrates the entire CI/CD pipeline
  - Runs build and test processes
  - Integrates with SonarQube and Nexus

- **SonarQube**: 
  - Performs static code analysis
  - Ensures code quality and identifies potential issues

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

- Jenkins with:
  - JDK 17
  - Maven

- Installed and configured:
  - SonarQube
  - Nexus Repository Manager
  - Tomcat Server
  - Maven
  - JDK 17

## 🔄 Pipeline Stages

![Global CI/CD pipeline](/images/cicd-pipeline.png)

The CI/CD pipeline follows these key stages:

1. **Git Checkout**: Clone repository from GitHub
2. **Compile**: Compile project using Maven
3. **Test**: Run unit tests
4. **SonarQube Analysis**: Analyze code quality
5. **Build**: Package application as WAR
6. **Deploy to Nexus**: Store artifacts
7. **Copy to Tomcat**: Transfer WAR to server
8. **Deploy WebApp**: Activate application

## 🛠 Setup Instructions

### 1. Jenkins Configuration

1. Install Maven and JDK 17 in Jenkins Global Tool Configuration
2. Add SSH credentials for Tomcat server access
3. Add credentials in `~/.m2/settings.xml` to connect to Nexus Repository
4. Create a new Jenkins Pipeline project
5. Copy the `Jenkinsfile` file to your Jenkins project

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
