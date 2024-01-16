pipeline {
    agent any
    
    tools{
        jdk 'jdk11'
        maven 'maven3.6'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }
    

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Git Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/jatin765/Demo-Shopping-Cart.git']])
            }
        }
        
        stage('Compile') {
            steps {
               sh 'mvn compile'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh '''$SCANNER_HOME:/bin/sonar-scanner -Dsonar.projectKey=Ekart  -Dsonar.projectName=Ekart \
                    -Dsonar.java.binaries=.'''
                }
            }
        }
        
        stage('Build and Tag Image') {
            steps {
                sh 'docker build -t app -f docker/Dockerfile  .'
                sh 'docker tag app jatin765/Ekart:latest'
            }
        }
        
        stage('Push Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                        sh 'docker push jatin765/Ekart:latest'
                    }
                }
            }
        }
        
        stage('Deploy container') {
            steps {
                sh 'docker run -d -p 80:8070 jatin765/Ekart:latest'
            }
        }
        
        stage('Deploy Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'k8', contextName: '', credentialsId: 'k8', namespace: 'webapp', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.52.4:6443') {
                    sh 'kubectl apply -f deployment.yml'    
                }
            }
        }
        
    }
}
