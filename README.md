AWS Infrastructure with terrform code with Jenkins Pipeline

This project automates the deployment of AWS infrastructure such as VPC,PUBLIC AND PRIVATE SUBNET, RT,  EC2, ASG 
using Terraform and Jenkins.
Jenkins pulls Terraform code from GitHub, validates it, and deploys the infrastructure.
1. Prerequisites

Ensure you have the following installed on your EC2 instance:

  Git (git --version)-> steps to install git 
  sudo yum update -y
  sudo yum install git -y
  git --version

  Jenkins install
  sudo yum update –y
  sudo wget -O /etc/yum.repos.d/jenkins.repo \ https://pkg.jenkins.io/redhat-stable/jenkins.repo
  sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
  sudo yum upgrade
  sudo dnf install java-17-amazon-corretto -y
  sudo yum install jenkins -y
  sudo systemctl enable jenkins
  sudo systemctl start jenkins
  Connect to http://<your_server_public_DNS>:8080 from your browser. You will be able to access Jenkins through its management interface:
  sudo cat /var/lib/jenkins/secrets/initialAdminPassword

    
  Terraform install
  sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum install terraform -y
terraform version

Configure Jenkins to Use GitHub
 Generate SSH Key for Jenkins
 sudo su - jenkins
ssh-keygen -t rsa -b 4096 -C "jenkins@your-server IP"
Press Enter for all options (no passphrase needed).

Copy SSH Public Key to GitHub
cat ~/.ssh/id_rsa.pub

Test GitHub Connection
ssh -T git@github.com
If successful, it should return:
✅ Hi <your-username>! You've successfully authenticated, but GitHub does not provide shell access.

I get the below Issue: Permission denied (publickey)
Solution:

   Make sure SSH key is added to GitHub (create a new repo then inside repo settings Deploy keys)
    Restart SSH agent:

   eval "$(ssh-agent -s)"
    ssh-add ~/.ssh/id_rsa

Step 3: Clone the GitHub Repository

cd /home/ec2-user/
git clone git@github.com:YourUsername/YourRepo.git
cd YourRepo

✅ Issue: fatal: Could not read from remote repository.
Solution:

   Check SSH Key is correctly added to GitHub
    Ensure correct GitHub repo URL (git remote -v)

Step 4: Create Terraform Files

Inside /home/ec2-user/YourRepo, create:

touch main.tf variables.tf outputs.tf Jenkinsfile

Example: main.tf

provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-0abcdef1237890"
  instance_type = "t2.micro"
}

✅ Issue: Terraform provider not found
Solution: Run:

terraform init

Step 5: Add Terraform Code to GitHub

git add .
git commit -m "Added Terraform infrastructure"
git push origin main

✅ Issue: fatal: not a git repository
Solution: Run:

git init
git remote add origin git@github.com:YourUsername/YourRepo.git

3. Configure Jenkins Pipeline
Step 1: Open Jenkins Dashboard

    Go to http://your-server-ip:8080
    Create New Item → Pipeline
    In the pipeline config:
        Git URL: git@github.com:YourUsername/YourRepo.git
        Branch: master (or main)
        Credentials: Add your SSH key

✅ Issue: Jenkins GitHub Authentication Fails
Solution:

  Go to Jenkins → Manage Jenkins → Credentials
    Add SSH Key for GitHub

Step 2: Create Jenkinsfile

Edit Jenkinsfile:

pipeline {
    agent any

  stages {
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github-ssh-key', url: 'git@github.com:YourUsername/YourRepo.git'
            }
        }

  stage('Initialize Terraform') {
            steps {
                sh "terraform init"
            }
        }

   stage('Validate Terraform') {
            steps {
                sh "terraform validate"
            }
        }

   stage('Plan Terraform') {
            steps {
                sh "terraform plan -out=tfplan"
            }
        }

  stage('Apply Terraform') {
            steps {
                input message: "Proceed with Terraform apply?", ok: "Apply"
                sh "terraform apply -auto-approve tfplan"
            }
        }
    }
}

✅ Issue: Jenkins pipeline fails with terraform command not found
Solution: Install Terraform on Jenkins node:

sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum install terraform -y

4. Running the Pipeline

    Click "Build Now" in Jenkins
    Pipeline stages:
        Checkout code ✅
        Initialize Terraform (terraform init) ✅
        Validate Terraform (terraform validate) ✅
        Plan Terraform (terraform plan) ✅
        Apply Terraform (terraform apply) ✅

✅ Issue: Terraform apply fails due to incorrect AWS credentials
Solution: Configure AWS credentials in Jenkins:

aws configure

5. Destroy Infrastructure

To delete all resources:

terraform destroy -auto-approve

✅ Issue: State file missing (terraform state list shows empty results)
Solution: Check if Terraform state file exists (ls -lah terraform.tfstate)
6. Best Practices

✅ Store Terraform state in S3 with remote backend
✅ Use Git branches for different environments (dev, staging, prod)
✅ Automate deployment with Jenkins and Terraform
✅ Use Terraform modules for reusable code
