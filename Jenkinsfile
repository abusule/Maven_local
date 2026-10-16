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
              withCredentials([string(credentialsId: 'SONAR_SECRET', variable: 'sonar_secret')]) {
                sh "mvn clean org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=devopswep -Dsonar.projectName='devopswep' -Dsonar.host.url=http://13.58.198.47:9000 -Dsonar.token=$SONAR_TOKEN"
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
                withCredentials([usernamePassword(credentialsId: 'DH_TOKEN', passwordVariable: 'DH_TOKEN', usernameVariable: 'DH_USER')]) {
                  sh 'echo $DH_TOKEN | docker login -u $DH_USER --password-stdin'
                  sh 'docker push abusule/tomcat:latest'
                }
            }
        }
        stage('Deploy') {
            steps{
                withCredentials([sshUserPrivateKey(credentialsId: 'AWS_KEY', keyFileVariable: 'AWS_KEY', usernameVariable: 'AWS_USER')]) {
                    sh 'ssh -i $AWS_KEY -o StrictHostKeyChecking=no $AWS_USER@18.216.228.129 "docker rm -f devopsweb-app || true; docker pull abusule/tomcat:latest ;  docker run -d -p 8080:8080 --name devopsweb-app abusule/tomcat:latest"'
                }
            }
        }
        
    }
}