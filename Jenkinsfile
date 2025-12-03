pipeline {
    agent any

    environment {
        IMAGE_NAME = "madhavgirdhar/bt-frontend"
        VAULT_CREDS = "ansible-vault-pass"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/madhav8511/BT-Frontend.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    sh """
                        docker build -t ${IMAGE_NAME}:latest .
                    """
                }
            }
        }

       stage("Push to Docker Hub"){
            steps{
                echo "Pushing Image to Hub..."
                script{
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                        docker.image("${IMAGE_NAME}:latest").push()
                    }
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
