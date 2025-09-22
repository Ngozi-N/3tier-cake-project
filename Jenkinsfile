pipeline {
    agent any
    
    tools {
        nodejs 'node24'
        //dockerTool 'docker'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/Ngozi-N/3tier-cake-project.git'
            }
        }
        
        stage('Install Package Dependencies') {
            steps {
                sh "npm install"
            }
        }
        
        stage('Unit Tests') {
            steps {
                sh 'npm test'
            }
        }
        
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs-report.html ."
            }
        }
        
        stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonar') {
                sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Engee -Dsonar.projectName=Engee"
                }
            }
        }
        
        stage('Docker Build & Tag') {
            steps {
                    sh "docker build -t ngozin/bakerylocation:latest ."
            }
        }
        
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o fs-report.html ngozin/bakerylocation:latest"
            }
        }
        
        stage('Docker Push Image') {
            steps {
                withDockerRegistry([credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/']) {
                        sh "docker push ngozin/bakerylocation:latest"
                }
            }
        }
        
        stage('Docker Deploy to Dev') {
            steps {
                sh '''
                  docker ps -q --filter "name=bakerylocation" | grep -q . && docker rm -f bakerylocation || true
                  docker run -d --name bakerylocation -p 3000:3000 ngozin/bakerylocation:latest
                '''
            }
        }
    }
}