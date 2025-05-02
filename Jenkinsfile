pipeline {
    agent { label "Jenkins-Agent" }

    environment {
        APP_NAME = "gokul0880/register-app-pipeline"
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

        stage("Get Latest Image Tag from Docker Hub") {
            steps {
                script {
                    def tagsJson = sh(
                        script: 'curl -s https://hub.docker.com/v2/repositories/gokul0880/register-app-pipeline/tags?page_size=1',
                        returnStdout: true
                    ).trim()

                    def tag = sh(
                        script: "echo '${tagsJson}' | jq -r '.results[0].name'",
                        returnStdout: true
                    ).trim()

                    env.IMAGE_TAG = tag
                    echo "Latest tag fetched: ${env.IMAGE_TAG}"
                }
            }
        }

        stage("Update the Deployment Tags") {
            steps {
                sh """
                    echo "Before update:"
                    cat deployment.yaml

                    sed -i "s|image:.*|image: ${APP_NAME}:${IMAGE_TAG}|g" deployment.yaml

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
                        git commit -m "Auto-updated to latest tag ${IMAGE_TAG}" || echo "No changes to commit"
                        git push https://github.com/gokul-badrappan/gitops-register-app main
                    """
                }
            }
        }
    }
}
