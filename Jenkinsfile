pipeline {
    agent any
    
    tools {
        nodejs 'node21'
    }
    
    environment {
        SCANNER_HOME= tool "sonar-scanner"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/Shubham-Stunner/3-Tier-Full-Stack.git'
            }
        }
        stage('Install Depedencies') {
            steps {
                sh "npm install"
            }
        }
        stage('Unit Test') {
            steps {
                sh "npm test"
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
                    sh " $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Campground -DsonarprojectName=Campground"
                }
            }
            
        }
        stage('Docker build & Tag') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred1', toolName: 'docker ') {
                        sh "docker build -t stunnershubham/camp:latest ."
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o fs-report.html stunnershubham/camp:latest"
            }
        }
        stage('Docker Push Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred1', toolName: 'docker ') {
                        sh "docker push stunnershubham/camp:latest"
                    }
                }
            }
        }
        stage('Docker Deploy to Dev') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred1', toolName: 'docker ') {
                        sh "docker run -d -p 3000:3000 stunnershubham/camp:latest"
                    }
                }
            }
        }
    }
}
