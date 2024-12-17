# Spring Petclinic with CI/CD

This project demonstrates a CI/CD pipeline for the Spring Petclinic application using Jenkins, Maven, SonarQube, Nexus, and Tomcat. The pipeline is defined using Jenkins Pipeline as Code and includes stages for building, testing, code analysis, and deployment.

## Project Structure

- **Jenkins Master VM**: Hosts Jenkins and SonarQube.
- **Nexus VM**: Hosts Nexus for artifact management.
- **Tomcat VM**: Hosts the Tomcat server for deploying the application.

## Prerequisites

- Jenkins
- SonarQube
- Nexus
- Tomcat
- Maven
- JDK 17

## Pipeline Overview

The CI/CD pipeline is defined in the `Jenkinsfile` and consists of the following stages:

1. **Git Checkout**: Clones the repository from GitHub.
2. **Compile**: Compiles the project using Maven.
3. **Test**: Runs the unit tests.
4. **SonarQube Analysis**: Analyzes the code quality using SonarQube.
5. **Build**: Packages the application as a WAR file.
6. **Deploy Artifacts to Nexus**: Deploys the built artifacts to Nexus.
7. **Copy WAR to Tomcat Server**: Copies the WAR file to the Tomcat server.
8. **Deploy WebApp**: Deploys the WAR file on the Tomcat server.

## Jenkinsfile

```groovy
pipeline {
    agent any 
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        REMOTE_USER = 'your-tom-cat-server-user'
        REMOTE_HOST = 'your-tom-cat-server-host'
        REMOTE_PATH = '/tmp'
        TOMCAT_PATH = '/opt/tomcat/webapps'
    }
    
    stages {
        
        stage("Git Checkout") {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/AbderrahmaneOd/spring-petclinic-jenkins'
            }
        }
        
        stage("Compile") {
            steps {
                sh "mvn clean compile"
            }
        }
        
        stage("Test") {
            steps {
                sh "mvn test"
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                sh ''' 
                mvn sonar:sonar \
                    -Dsonar.projectKey=Petclinic-SpringBoot \
                    -Dsonar.projectName='Petclinic-SpringBoot' \
                    -Dsonar.host.url=http://localhost:9000 \
                    -Dsonar.token=<your-auth-token>
                '''
            }
        }
        
        stage("Build") {
            steps {
                sh "mvn clean package"
            }
        }
        
        stage("Deploy Artifacts to Nexus") {
            steps {
                sh 'mvn deploy -DskipTests'
            }
        }
        
        stage('Copy War to Tomcat Server') {
            steps {
                sshagent(['tomcat-server']) {
                    sh 'scp -o StrictHostKeyChecking=no **/*.war ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_PATH}'
                }
            }
        }
        
        stage("Deploy WebApp") {
            steps {
                sshagent(['tomcat-server']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} '
                        sudo mv ${REMOTE_PATH}/*.war ${TOMCAT_PATH} && 
                        sudo chown -R tomcat:tomcat ${TOMCAT_PATH}
                    '
                    """
                }
            }
        }
    }
}
