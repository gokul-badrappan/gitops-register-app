pipeline {
    agent { label "Jenkins-Agent" }

    environment {
        APP_NAME = "register-app-pipeline"
    }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main',
                    credentialsId: 'github',
                    url: 'https://github.com/gokul-badrappan/gitops-register-app'
            }
        }

        stage("Fetch Latest Docker Image Tag") {
            steps {
                script {
                    def latestTag = sh(
                        script: """
                            curl -s https://registry.hub.docker.com/v2/repositories/gokul0880/${APP_NAME}/tags \\
                            | jq -r '.results[].name' \\
                            | grep -E '^[0-9]+$' \\
                            | sort -nr \\
                            | head -n 1
                        """,
                        returnStdout: true
                    ).trim()
                    env.IMAGE_TAG = latestTag
                    echo "Fetched latest image tag: ${env.IMAGE_TAG}"
                }
            }
        }

        stage("Update the Deployment Tags") {
            steps {
                sh """
                    echo "Before update:"
                    cat deployment.yaml
                    sed -i "s|image:.*|image: gokul0880/${APP_NAME}:${IMAGE_TAG}|g" deployment.yaml
                    echo "After update:"
                    cat deployment.yaml
                """
            }
        }

        stage("Push the changed deployment file to Git") {
            steps {
                withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                    sh """
                        git config --global user.name "gokul-badrappan"
                        git config --global user.email "gokul0880@gmail.com"
                        git add deployment.yaml
                        git commit -m "CD: Updated Deployment Manifest to ${APP_NAME}:${IMAGE_TAG}" || echo "No changes to commit"
                        git push https://github.com/gokul-badrappan/gitops-register-app main
                    """
                }
            }
        }
    }
}
