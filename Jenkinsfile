pipeline {
    agent any

    environment {
        GIT_CREDENTIALS_ID = 'github-https'  // Ensure this credential exists in Jenkins
        GIT_REPO = 'https://github.com/Theramya10/jenkins.git'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    try {
                        checkout([$class: 'GitSCM', 
                            branches: [[name: '*/master']], 
                            doGenerateSubmoduleConfigurations: false, 
                            extensions: [], 
                            submoduleCfg: [], 
                            userRemoteConfigs: [[
                                credentialsId: GIT_CREDENTIALS_ID,
                                url: GIT_REPO
                            ]]
                        ])
                    } catch (Exception e) {
                        error("❌ Git Checkout Failed: ${e.message}")
                    }
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    sh 'sudo apt update && sudo apt install -y git terraform' // Adjust for your OS
                }
            }
        }

        stage('Terraform Init') {
            steps {
                script {
                    try {
                        sh 'terraform init -reconfigure' // Ensures proper backend setup
                    } catch (Exception e) {
                        error("❌ Terraform Init Failed: ${e.message}")
                    }
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                script {
                    try {
                        sh 'terraform plan -out=tfplan'
                    } catch (Exception e) {
                        error("❌ Terraform Plan Failed: ${e.message}")
                    }
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                script {
                    try {
                        sh 'terraform apply -auto-approve tfplan'
                    } catch (Exception e) {
                        error("❌ Terraform Apply Failed: ${e.message}")
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs for details.'
        }
    }
}
