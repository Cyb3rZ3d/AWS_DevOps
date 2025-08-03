Ruben Valdez
July 17, 2025

The following is a training environment with learning AWS DevOps.

<br><br>

# ***Project 1:    Deploying a Static Website from GitHub to AWS EC2***

<br><br>

## Step 1: Launching Ubuntu EC2 Instance

- Name:  Project-1_WebServer
- Amazon Machine Image (AMI):   Ubuntu Server 24.04 LTS (Free Tier eligible) 64-Bit
- Instance Type:    t3.small
- Key Pair name:    Alamo_CyITS
- Network Settings:  
    - Security Group: Anywhere 70.120.67.69/32 (Used IP Chicken to acquire the network IP.  "/32" means only this exact IP can connect to SSH.)
        - Allow SSH
        - Allow HTTP

<br><br>

## Step 2: Connect to EC2 from Windows

1. Navigated to the directory where the PEM file was placed `AWS Keys` and change permissions to only be read by me.

        icacls "Alamo_CyITS.pem" /inheritance:r

        icacls "Alamo_CyITS.pem" /grant:r "$($env:USERNAME):(R)"

    ![alt text](image.png)


2. SSH into EC2 instance `Project-1_WebServer`

        ssh -i Alamo_CyITS.pem ubuntu@54.165.82.86

    ![alt text](image-7.png)    

<br><br>

## Step 3:  Start initial OS update and upgrade, and install Apache2 on EC2

1. Complete the following commands:

        sudo apt update && upgrade -y
        sudo apt install apache2 -y
        sudo systemctl start apache2
        sudo systemctl enable apache2
    
    ![alt text](image-2.png)

2. Upon completion, paste the external IP into a browser.  The result should be the Apache2 screen.

<br><br>

## Step 4: Create and Push Static Website to GitHub

*** On Windows PowerShell, make a new folder for your website ***

1. Create a folder for your website:

        mkdir C:\Dev\my-static-site
        cd C:\Dev\my-static-site


2. Create and Add a simple index.html

        notepad index.html

    - Apply the following in the file and save:

            <!-- index.html -->
            <!DOCTYPE html>
            <html>
            <head><title>Hello DevOps</title></head>
            <body>
            <h1>Hello from EC2 + Apache!</h1>
            </body>
            </html>


3. Initialize Git and push to GitHub

        git init
        git add .
        git commit -m "My static site"
        gh repo create my-static-site --public --source=. --remote=origin --push  
        
        //NOTE:  Requires GitHub CLI. If not, create a GitHub repo manually and push it.

    - Install GitHub CLI (gh)

        - Go to: https://cli.github.com/

        - Click “Download for Windows” and install it.

        - After install, restart PowerShell or open a new terminal.

        - Run:

                gh auth login

            - Choose GitHub.com

            - Choose HTTPS

            - Authenticate in browser when prompted

            - Using the same terminal

                    cd C:\Dev\my-static-site

                    gh repo create my-static-site --public --source=. --remote=origin --push

    ![alt text](image-8.png) ![alt text](image-9.png) 


4. Deploy the website to EC2 from Github

        cd /var/www/html
        sudo rm -rf *
        sudo apt install git -y
        sudo git clone https://github.com/<your-username>/my-static-site.git .


![alt text](image-11.png) ![alt text](image-10.png)


<br><br><br><br>


# ***Project 2: DevOps Tools & Automation***

- Containerize a Flask/Node.js app with Docker
- Write Terraform to deploy EC2 + security group + S3
- Create GitHub Actions to:
    - Run linter/test
    - Deploy app to EC2 if tests pass

<br>

Goal: Automate deployment of a containerized Flask/Node.js app on AWS EC2 using Terraform + GitHub Actions CI/CD pipeline.

Phases:
- Containerize App with Docker
- Provision AWS Resources with Terraform (EC2, Security Group, S3)
- Automate CI/CD with GitHub Actions


## Containerize App with Docker

![alt text](image-13.png)

### ***Part 1: Containerize a Flask or Node.js App with Docker***
1. Create Project Structure (Windows PowerShell)

        mkdir C:\Dev\flask-docker-app
        cd C:\Dev\flask-docker-app

    ![alt text](image-12.png)
    
    - Build a Simple Flask App (Python Example)

            notepad app.py

        ```
        from flask import Flask
        app = Flask(__name__)

        @app.route('/')
        def home():
            return "Hello from Dockerized Flask App!"

        if __name__ == "__main__":
            app.run(host="0.0.0.0", port=5000)
        ```

    - Create a requirements.txt

            notepad requirements.txt 

        ```
        flask = 2.3.3
        ```

    - Create a Dockerfile

            nano Dockerfile

        ```
        # Use official Python image
        FROM python:3.10-slim

        # Set working directory
        WORKDIR /app

        # Copy files
        COPY requirements.txt requirements.txt
        RUN pip install -r requirements.txt

        COPY . .

        # Expose port
        EXPOSE 5000

        # Run the application
        CMD ["python", "app.py"]
        ```

    - Build and Test Docker Container Locally

            docker build -t flask-docker-app .
            docker run -p 5000:5000 flask-docker-app

        - Visit the built site http://localhost:5000/

    ![alt text](image-15.png) ![alt text](image-16.png)



### ***Part 2: Terraform to Deploy EC2, Security Group, and S3***

***NOTE: Before starting Part 2, I needed to complete a few task such as:***

- Download and Install Terraform
    
    - After the install, unzipped the folder and and moved `Terraform.exe` to a new directory in C:\Terraform.

- Adding Terraform to PATH in System Environment Variables to have access throughout the system.
    - Start menu > System Environment Variables > Environment Variables:
        - In the System Variable window find and locate `Path` and select Edit then New
        - Add C:\Terraform then click OK on all windows till the System Environment Variables windows are closed.

- Confirming the Terraform Installation

        Terraform -version

- Installing AWS CLI using
    
    - https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

            msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi /qn

- Creating a AWS IAM User:

        https://933946046377.signin.aws.amazon.com/console
        terraform-user
        hOL5($v[!@

- Created Access Key and Secret Key, in the `AWS Console > IAM > Users > terraform-user > Security credentials > Create access Key`

        AKIA5S44FW6UZFQ4JYN2
        EgO9A1sHVMdmrBR6vngKNi4cGtxYrsllbqK6BVyy



1. Create Terraform Folder and Files

    - Go to the project folder and create a new directory named `terraform`

        cd C:\Dev\flask-docker-app
        mkdir terraform
        cd terraform

    - Create the following files

            main.tf (resources)
            variables.tf (input variables)
            outputs.tf (export info after deployment)

2. Before using Terraform, I needed to configure AWS CLI with my new user Access Key credentials:

    ![alt text](image-17.png)

3. Initialize Terraform

        terraform init

    ![alt text](image-18.png)

4.  Plan Deployment

        terraform plan -var "key_name=Alamo_CyITS"

    ![alt text](image-19.png) ![alt text](image-20.png)

5. Apply Deployment

        terraform apply -var "key_name=Alamo_CyITS" -auto-approve

    ![alt text](image-21.png) ![alt text](image-22.png)

6. Verify Deployment

    - In `AWS Console > S3` I was able to confirm the S3 bucket `flask-docker-bucket-3accc6ef` was created successfully

        ![alt text](image-23.png)


Provide detailed instructions for Part 3: GitHub Actions CI/CD


1.  Prepare EC2 for GitHub Deployment

    - Install Docker on EC2

            sudo apt update
            sudo apt install docker.io -y
            sudo systemctl start docker
            sudo systemctl enable docker

    - Allow to run Docker w/o Sudo

            sudo usermod -aG docker ubuntu

2.  Generate SSH Key for GitHub Actions 


type C:\Path\to\github-actions-key.pub | ssh -i "C:\Path\to\your-key.pem" ubuntu@<EC2_PUBLIC_IP> "cat >> ~/.ssh/authorized_keys"


type 'C:\Users\rubva\Documents\AWS Keys\Ubuntu_Keys\github-actions-key.pub' | ssh -i 'C:\Users\rubva\Documents\AWS Keys\Alamo_CyITS.pem' ubuntu@54.165.82.86 "cat >> ~/.ssh/authorized_keys"


3. Stored Environment Secrets in GitHub

    - In my GitHub repository, `my-static-site`, Settingg > Secrets and Variables > Actions > New Repository secret

    - 


4. Create GitHub Actions Workflow



