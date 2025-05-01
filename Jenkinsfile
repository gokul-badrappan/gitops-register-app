pipeline {
    agent any
    environment {
        REGISTRY = 'gcr.io'
        IMAGE_NAME = 'gokul0880/register-app-pipeline'
        TAG = 'latest'
    }
    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()  // Cleans up the workspace before starting the build
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/gokul-badrappan/register-app-pipeline.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    docker.build("${IMAGE_NAME}:${TAG}")
                }
            }
        }

        stage('Push to Registry') {
            steps {
                script {
                    // Push the image to the container registry
                    docker.withRegistry("https://${REGISTRY}", 'gcr-token') {
                        docker.image("${IMAGE_NAME}:${TAG}").push()
                    }
                }
            }
        }

        stage('Trivy Scan for Vulnerabilities') {
            steps {
                script {
                    // Run Trivy to scan the Docker image for vulnerabilities
                    sh 'docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${IMAGE_NAME}:${TAG} --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table --skip-java'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Apply Kubernetes manifests for deployment
                    sh 'kubectl apply -f k8s/deployment.yaml'
                }
            }
        }

        stage('Cleanup Docker Images') {
            steps {
                script {
                    // Clean up Docker images to free up space
                    sh 'docker system prune -f'
                }
            }
        }
    }

    post {
        success {
            echo 'Build and deployment succeeded!'
        }
        failure {
            echo 'Build or deployment failed.'
        }
        always {
            cleanWs()  // Clean workspace again after the pipeline finishes
        }
    }
}
