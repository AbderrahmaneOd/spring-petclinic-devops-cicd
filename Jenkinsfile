pipeline {
    agent any 
    
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        REMOTE_USER = 'abderrahmane'
        REMOTE_HOST = '20.199.75.233'
        REMOTE_PATH = '/tmp'
        TOMCAT_PATH = '/opt/tomcat/webapps'
    }
    
    
    stages{
        
        stage("Git Checkout"){
        steps{
            git branch: 'main', changelog: false, poll: false, url: 'https://github.com/AbderrahmaneOd/spring-petclinic-jenkins'
        }
    }
    
    
        stage("Compile"){
            steps{
                sh "mvn clean compile"
            }
        }
        
         stage("Test"){
            steps{
                sh "mvn test"
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                sh ''' mvn sonar:sonar \
                        -Dsonar.projectKey=Petclinic-SpringBoot \
                        -Dsonar.projectName='Petclinic-SpringBoot' \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.token=sqp_b8b93b21e34210008a3c6b7a1bcc3a6e0799b546
                     '''
            }
        }
        
       
         stage("Build"){
            steps{
                sh " mvn clean package"
            }
        }
        
        
        
        stage("Deploy Artifacts to Nexus"){
            steps{
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