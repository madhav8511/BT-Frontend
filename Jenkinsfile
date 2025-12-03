pipeline {
    agent any

    environment {
        IMAGE_NAME = "madhavgirdhar/bt-frontend"
        DOCKERHUB_CREDS = "dockerhub-creds"
        VAULT_CREDS = "ansible-vault-pass"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    sh """
                        docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} -t ${IMAGE_NAME}:latest .
                    """
                }
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKERHUB_CREDS}",
                    usernameVariable: "DOCKER_USER",
                    passwordVariable: "DOCKER_PASS"
                )]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                        docker push ${IMAGE_NAME}:latest
                        docker logout
                    """
                }
            }
        }

        stage('Deploy to Kubernetes using Ansible') {
            steps {
                withCredentials([string(
                    credentialsId: "${VAULT_CREDS}",
                    variable: "VAULT_PASS"
                )]) {
                    sh '''
                        echo "$VAULT_PASS" > .vault_pass

                        ansible-playbook playbook.yml \
                          -i inventory.ini \
                          --vault-password-file .vault_pass \
                          --extra-vars "app_version=${BUILD_NUMBER}"

                        rm -f .vault_pass
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Deployment successful! Frontend version ${BUILD_NUMBER} deployed."
        }
        failure {
            echo "❌ Deployment failed. Check Jenkins logs."
        }
    }
}
