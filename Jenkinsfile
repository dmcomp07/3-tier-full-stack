pipeline {
    agent any
    
    tools {
        nodejs 'nodejs-test'
    }
    
    environment {
        SCANNER_HOME = tool "sonarqube"
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
                withSonarQubeEnv('sonaqube-api') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Campground -Dsonar.projectName=Campground"  // Corrected sonar property
                }
            }
        }
        stage('Docker build & Tag') {
            steps {
                script {
					
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {  // Removed extra space
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
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {  // Removed extra space
                        sh "docker push dmcomp07/camp:latest"
                    }
                }
            }
        }
        stage('Docker Deploy to Dev') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {  // Removed extra space
                        sh "docker run -d -p 3000:3000 dmcomp07/camp:latest"
                    }
                }
            }
        }
		stage('Deploy To EKS') {
				steps {
					withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'eks4', contextName: '', credentialsId: 'kube-secret', namespace: 'webapps', serverUrl: 'https://54F3D54B522DCA8FF9FF99A61DE57FEC.gr7.us-east-1.eks.amazonaws.com']]) {
					sh "kubectl apply -f Manifests/dss.yml"

				}
			}
		}
		stage('Verify Deployment') {
				steps {
					withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'eks4', contextName: '', credentialsId: 'kube-secret', namespace: 'webapps', serverUrl: 'https://54F3D54B522DCA8FF9FF99A61DE57FEC.gr7.us-east-1.eks.amazonaws.com']]) {
					sh "kubectl get svc -n webapps"
					sh "kubectl get pods -n webapps"


				}
			}
		}
		
		
    }
}
