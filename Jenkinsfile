pipeline {
    agent any
    environment {
        REGISTRY = 'gcr.io'
        IMAGE_NAME = 'gokul0880/register-app-pipeline'
        TAG = 'latest'
    }

    stages {
        stage('Prepare Workspace') {
            steps {
                cleanWs()
                // Show directory structure for debugging
                sh 'ls -la'
                sh 'ls -la docker-context/'
                script {
                    // Ensure WAR file is available
                    if (!fileExists('docker-context/app.war')) {
                        error "app.war not found in docker-context/. Make sure it is available before running this pipeline."
                    }
                    if (!fileExists('docker-context/Dockerfile')) {
                        error "Dockerfile not found in docker-context/. Please include Dockerfile."
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${TAG}", "docker-context/")
                }
            }
        }

        stage('Push to Registry') {
            steps {
                script {
                    docker.withRegistry("https://${REGISTRY}", 'gcr-token') {
                        docker.image("${IMAGE_NAME}:${TAG}").push()
                    }
                }
            }
        }

        stage('Trivy Scan for Vulnerabilities') {
            steps {
                script {
                    sh 'docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${IMAGE_NAME}:${TAG} --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table --skip-java'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml'
            }
        }

        stage('Cleanup Docker Images') {
            steps {
                sh 'docker system prune -f'
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
            cleanWs()
        }
    }
}
