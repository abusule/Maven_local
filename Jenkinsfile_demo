pipeline {
    agent any
    tools {
        maven 'maven3.9'
    }
    stages {
        stage('Code Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git_repo_2', url: 'https://github.com/udodi05/docker_local.git'
            }
        }
        stage('Package Code Artifacts') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'whoami'
                sh 'docker build -t kniru/tomcat:${BUILD_NUMBER} .'
                sh 'docker tag kniru/tomcat:${BUILD_NUMBER} kniru/tomcat:latest'
            }
        }
        stage('Push Image to Repo') {
            steps{
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin'
                    sh 'docker push kniru/tomcat:latest'
                }
            }
        }
        stage('Run Containers') {
            steps{
                sshagent(['agent_private_key']) {
                    
                    sh 'scp load_config/haproxy.cfg deploy.sh kniru@192.168.2.141:/opt/docker_config_files/'
                    sh 'ssh kniru@192.168.2.141 "bash /opt/docker_config_files/deploy.sh"'
                }
            }
        }
        // stage('Deploy to Tomcat') {
        //     steps{
        //         deploy adapters: [tomcat9(credentialsId: 'tomcat_password', path: '', url: 'http://192.168.2.141:8081')], contextPath: 'web-app', war: 'target/*.war'
        //         deploy adapters: [tomcat9(credentialsId: 'tomcat_password', path: '', url: 'http://192.168.2.141:8082')], contextPath: 'web-app', war: 'target/*.war'
        //         deploy adapters: [tomcat9(credentialsId: 'tomcat_password', path: '', url: 'http://192.168.2.141:8083')], contextPath: 'web-app', war: 'target/*.war'
        //         deploy adapters: [tomcat9(credentialsId: 'tomcat_password', path: '', url: 'http://192.168.2.141:8084')], contextPath: 'web-app', war: 'target/*.war'
        //         deploy adapters: [tomcat9(credentialsId: 'tomcat_password', path: '', url: 'http://192.168.2.141:8085')], contextPath: 'web-app', war: 'target/*.war'
        //         deploy adapters: [tomcat9(credentialsId: 'tomcat_password', path: '', url: 'http://192.168.2.141:8086')], contextPath: 'web-app', war: 'target/*.war'
        //     }
        // }
    }
}
