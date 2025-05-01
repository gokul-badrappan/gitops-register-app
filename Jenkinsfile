pipeline {
    agent { label "Jenkins-Agent" }

    environment {
        APP_NAME = "register-app-pipeline"
    }

    stages {
        // 🚫 Removed "Cleanup Workspace" stage

        stage("Update the Deployment Tags") {
            steps {
                sh """
                   echo Before updating deployment.yaml:
                   cat deployment.yaml

                   sed -i 's|${APP_NAME}.*|${APP_NAME}:${IMAGE_TAG}|g' deployment.yaml

                   echo After updating deployment.yaml:
                   cat deployment.yaml
                """
            }
        }

        stage("Push the changed deployment file to Git") {
            steps {
                sh """
                   git config --global user.name "gokul-badrappan"
                   git config --global user.email "gokul0880@gmail.com"
                   git add deployment.yaml
                   git commit -m "Updated Deployment Manifest"
                """
                withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                    sh "git push https://github.com/gokul-badrappan/gitops-register-app main"
                }
            }
        }
    }
}
