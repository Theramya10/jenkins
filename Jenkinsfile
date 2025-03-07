pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/Theramya10/jenkins', branch: 'master'
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
                    sh 'terraform plan'
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                script {
                    sh 'terraform apply -auto-approve'
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
