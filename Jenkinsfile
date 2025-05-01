pipeline {
    agent { label "Jenkins-Agent" }

    environment {
        APP_NAME = "register-app-pipeline"
        IMAGE_TAG = "${BUILD_NUMBER}"
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

        stage("Update the Deployment Tags") {
            steps {
                sh """
                   echo "Before update:"
                   cat deployment.yaml
                   sed -i 's|${APP_NAME}:.*|${APP_NAME}:${IMAGE_TAG}|g' deployment.yaml
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
                       git commit -m "Updated Deployment Manifest to ${APP_NAME}:${IMAGE_TAG}" || echo "No changes to commit"
                       git push https://github.com/gokul-badrappan/gitops-register-app main
                    """
                }
            }
        }
    }
}
