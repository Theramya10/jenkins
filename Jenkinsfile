pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'manster', credentialsId: 'github-ssh-key', url: 'git@github.com:Theramya10/jenkins.git'
            }
        }
        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }
        stage('Terraform Plan') {
            steps {
                sh 'terraform plan -out=tfplan'
            }
        }
        stage('Terraform Apply') {
            steps {
                sh 'terraform apply -auto-approve tfplan'
            }
        }
         stage('Terraform validate') {
            steps {
                sh 'terraform validate'
            }
        }
 }
}

