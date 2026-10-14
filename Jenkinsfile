pipeline {
    agent any
    tools {
        maven 'Maven3.10.0'
    }
    stages {
        stage('Git Checkout') {
            steps{
                git branch: 'main', url: 'https://github.com/abusule/Maven_local.git'
            }
        }
        stage('Sonar Scan') {
            steps{
              withCredentials([string(credentialsId: 'sonar_token', variable: 'SONAR_TOKEN')]) {
                sh "mvn clean org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=acada-webapp -Dsonar.projectName='acada-webapp' -Dsonar.host.url=http://35.183.105.208:9000 -Dsonar.token=$SONAR_TOKEN"
                }  
            }
        }
        stage('Maven Build') {
            steps{
                sh 'mvn clean package'
            }
        }
        stage('Docker Build & Push') {
            steps{
                sh 'docker build -t abusule/tomcat:latest .'
                withCredentials([usernamePassword(credentialsId: 'dh_token_cred', passwordVariable: 'dh_token', usernameVariable: 'dh_user')]) {
                  sh 'echo $DH_TOKEN | docker login -u $DH_USER --password-stdin'
                  sh 'docker push abusule/tomcat:latest'
                }
            }
        }
        stage('Deploy') {
            steps{
                withCredentials([sshUserPrivateKey(credentialsId: 'aws_key', keyFileVariable: 'aws_key', usernameVariable: 'aws_user')]) {
                    sh 'ssh -i $aws_key -o StrictHostKeyChecking=no $aws_user@3.21.169.109 "docker rm -f acada-web || true; docker pull abusule/tomcat:latest ;  docker run -d -p 8080:8080 --name acada-web abusule/tomcat:latest"'
                }
            }
        }
        
    }
}