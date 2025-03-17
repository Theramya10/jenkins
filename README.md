AWS Infrastructure Deployment using Terraform & Jenkins

This guide automates the deployment of AWS infrastructure (VPC, Public and Private Subnets, Route Tables, EC2, ASG) using Terraform and Jenkins.

Jenkins pulls the Terraform code from GitHub, validates it, and deploys the infrastructure.

1️⃣ Prerequisites

Ensure you have the following installed on your EC2 instance:

1.1 Install Git

sudo yum update -y
sudo yum install git -y

👉 Verify Git Installation:

git --version

1.3 Install Jenkins

https://www.jenkins.io/doc/tutorials/tutorial-for-installing-jenkins-on-AWS/

👉 Access Jenkins::

Open http://<EC2-Public-IP>:8080

Retrieve Admin Password:

sudo cat /var/lib/jenkins/secrets/initialAdminPassword

1.4 Install Terraform

sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum install terraform -y

👉 Verify Terraform Installation:

terraform version

2️⃣ Configure Jenkins to Use GitHub

2.1 Generate SSH Key for Jenkins

sudo su - jenkins
ssh-keygen -t rsa -b 4096 -C "jenkins@your-server"

Press Enter for all options (no passphrase needed).

2.2 Add SSH Key to GitHub

cat ~/.ssh/id_rsa.pub

Copy the key and add it to GitHub under Settings → SSH Keys.

2.3 Test GitHub Connection

ssh -T git@github.com

✅ If successful, it should return:

Hi <your-username>! You've successfully authenticated, but GitHub does not provide shell access.

Issue: Permission denied (publickey)
Solution:

eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa

3️⃣ Clone GitHub Repository

cd /home/ec2-user/
git clone git@github.com:YourUsername/YourRepo.git
cd YourRepo

✅ Issue: fatal: Could not read from remote repository.
Solution:

Ensure SSH Key is correctly added to GitHub.

Verify correct GitHub repo URL:

git remote -v

4️⃣ Create Terraform Files

Inside /home/ec2-user/YourRepo, create:

touch main.tf variables.tf outputs.tf Jenkinsfile

**Example **main.tf

provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-0abcdef1237890"
  instance_type = "t2.micro"
}

✅ Issue: Terraform provider not found
Solution:

terraform init

5️⃣ Push Terraform Code to GitHub

git add .
git commit -m "Added Terraform infrastructure"
git push origin main

✅ Issue: fatal: not a git repository
Solution:

git init
git remote add origin git@github.com:YourUsername/YourRepo.git

6️⃣ Configure Jenkins Pipeline

6.1 Create New Jenkins Pipeline

Go to http://your-server-ip:8080

New Item → Pipeline

Set Git URL: git@github.com:YourUsername/YourRepo.git

Branch: main

Credentials: Add your SSH key.

✅ Issue: Jenkins GitHub Authentication Fails
Solution:

Manage Jenkins → Credentials

Add SSH Key for GitHub

7️⃣ Create Jenkinsfile

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
Solution:

sudo yum install terraform -y

8️⃣ Running the Pipeline

Click "Build Now" in Jenkins.

✅ Pipeline stages:

Checkout Code ✅

Initialize Terraform (terraform init) ✅

Validate Terraform (terraform validate) ✅

Plan Terraform (terraform plan) ✅

Apply Terraform (terraform apply) ✅

✅ Issue: Terraform apply fails due to incorrect AWS credentials
Solution:

aws configure

9️⃣ Destroy Infrastructure

To delete all resources:

terraform destroy -auto-approve

✅ Issue: State file missing (terraform state list shows empty results)
Solution:

ls -lah terraform.tfstate

🔹 Best Practices

✅ Store Terraform state in S3 with remote backend.✅ Use Git branches for different environments (dev, staging, prod).✅ Automate deployment with Jenkins and Terraform.✅ Use Terraform modules for reusable code.

🚀 Conclusion

With this setup, your AWS infrastructure is fully automated using Jenkins and Terraform. 🎯

