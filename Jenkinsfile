pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/Theramya10/jenkins' // Update with your repo URL
        GIT_BRANCH = 'master' // Update if needed
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    git branch: "${GIT_BRANCH}", url: "${GIT_REPO}"
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    sh '''
                        if [ -f /etc/debian_version ]; then
                            echo "" | sudo -S apt update
                        elif [ -f /etc/redhat-release ]; then
                            echo "" | sudo -S yum update -y
                        elif [ -f /etc/alpine-release ]; then
                            echo "" | sudo -S apk update
                        elif grep -qi "amazon linux" /etc/os-release; then
                            echo "" | sudo -S dnf update -y
                        else
                            echo "Unsupported OS"
                            exit 1
                        fi
                    '''
                }
            }
        }

        stage('Terraform Init') {
            steps {
                script {
                    sh 'terraform init'
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                script {
                    sh 'terraform plan -out=tfplan'
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                script {
                    sh 'terraform apply -auto-approve tfplan'
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs for details."
        }
    }
}
