pipeline {
    agent any
    tools {
        maven 'Maven3.10.0'
    }
    stages {
        stage('Git Checkout') {
            steps{
                git branch: 'main', credentialsId: 'gitlab_pat', url: 'https://github.com/udodi05/docker_local.git'
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
                withCredentials([usernamePassword(credentialsId: 'docker_hub_cred', passwordVariable: 'DH_TOKEN', usernameVariable: 'DH_USER')]) {
                  sh 'echo $DH_TOKEN | docker login -u $DH_USER --password-stdin'
                  sh 'docker push abusule/tomcat:latest'
                }
            }
        }
        stage('Deploy') {
            steps{
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2_key', keyFileVariable: 'EC_KEY', usernameVariable: 'EC_USER')]) {
                    sh 'ssh -i $EC_KEY -o StrictHostKeyChecking=no $EC_USER@15.157.61.120 "docker rm -f acada-web || true; docker pull abusule/tomcat:latest ;  docker run -d -p 8080:8080 --name acada-web abusule/tomcat:latest"'
                }
            }
        }
        
    }
}