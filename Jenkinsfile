pipeline {
    agent any
    
    tools {
        nodejs 'nodejs-test'
    }
    
    environment {
        SCANNER_HOME = tool "sonar-scanner"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/dmcomp07/3-tier-full-stack.git'
            }
        }
        stage('Install Dependencies') {  // Typo corrected from "Depedencies"
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
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Campground -Dsonar.projectName=Campground"  // Corrected sonar property
                }
            }
        }
        stage('Docker build & Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-id', toolName: 'docker') {  // Removed extra space
                        sh "docker build -t dmcomp07/camp:latest ."
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image-report.html dmcomp07/camp:latest"  // Changed report file name to avoid overwriting
            }
        }
        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred1', toolName: 'docker') {  // Removed extra space
                        sh "docker push dmcomp07/camp:latest"
                    }
                }
            }
        }
        stage('Docker Deploy to Dev') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-id', toolName: 'docker') {  // Removed extra space
                        sh "docker run -d -p 3000:3000 dmcomp07/camp:latest"
                    }
                }
            }
        }
    }
}
