## Automated CodeDeploy with Node.js Application on EC2

Below is a step-by-step documentation on how we automate the deployment of a Node.js application using AWS CodeDeploy, including setting up EC2 instances, preparing scripts, and configuring AWS CodeDeploy.

## Prerequisites
- AWS `EC2 instance` set up and running
- AWS `IAM Roles` with appropriate permissions
- AWS `CodeDeploy` set up for EC2 instances
- `Node.js` and `npm` installed

### Step-by-Step Guide
#### 1. **Setup EC2 Instance and IAM Roles**
   
#### 2. **Create an IAM Role for EC2 with permissions:**
- `AmazonS3ReadOnlyAccess`
- `AWSCodeDeployRole`
Attach the IAM Role to your EC2 instance.

### 3. **Install CodeDeploy Agent on your EC2 instance (if not installed):**

```bash
sudo yum update
sudo yum install -y ruby
sudo yum install -y wget
wget https://github.com/aws/aws-codedeploy-agent/releases/download/latest/codedeploy-agent-1.0-1.0.x86_64.rpm
sudo rpm -i codedeploy-agent-1.0-1.0.x86_64.rpm
sudo service codedeploy-agent start

```

### 2. **Prepare Deployment Scripts**
- Create the appspec.yml file
- The appspec.yml file should be structured as follows:
```bash
version: 0.0
os: linux
files:
  - source: /<path-to-your-app-directory>/
    destination: /home/ubuntu/CMS-UEI/src
hooks:
  ApplicationStop:
    - location: scripts/application_stop.sh
      timeout: 300
      runas: ubuntu
  BeforeInstall:
    - location: scripts/before_install.sh
      timeout: 300
      runas: ubuntu
  ApplicationStart:
    - location: scripts/application_start.sh
      timeout: 300
      runas: ubuntu
```

- Create the application_start.sh file
```bash 

#!/bin/bash
sudo chmod -R 777 /home/ubuntu/
cd /home/ubuntu/CMS-UEI/src  

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion" 

npm install 

pm2 restart index || pm2 start index.js --name index
pm2 save
```
- Create the application_stop.sh file
```bash
#!/bin/bash
# Stopping existing node servers
# pkill node
```
- Create the before_install.sh file
```bash
#!/bin/bash
# Download node and npm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash
. ~/.nvm/nvm.sh
nvm install node

# Create working directory if it doesn't exist
DIR="/home/ubuntu/"
if [ ! -d "$DIR" ]; then
  echo "Creating ${DIR} directory"
  mkdir ${DIR}
fi
```
### 3. **Deploy Using AWS CodeDeploy**
- Create an Application in AWS CodeDeploy.
- Create a Deployment Group with:
- EC2 instances tagged using a unique tag (name, ec2-instance-name).
- Appropriate IAM roles for CodeDeploy.
### Step 4: **Create CodeDeploy Deployment Group**
Create Deployment Group:
- Inside your CodeDeploy application, create a Deployment Group.
- Define the `Deployment Group name` (e.g., MyApp-DeploymentGroup).
- Specify `EC2 Instances` with `EC2 Tag`:

Use the EC2 tag to define which EC2 instances should be part of the deployment group:
Tag Name: name
Tag Value: ec2-instance-name
This will target the EC2 instances that have this tag attached.
Select Deployment Configuration:

Choose In-Place as the deployment strategy. This will update the application on the existing EC2 instances without creating a new environment.
In-Place Deployment means the code is deployed directly to the EC2 instances that are already part of the deployment group.
Set IAM Role Permissions:

Ensure that the IAM Role for CodeDeploy has the following permissions:
AmazonS3ReadOnlyAccess: This grants read-only access to the S3 bucket where the source code is stored.
AWSCodeDeployRole: This role is required for CodeDeploy to interact with EC2 instances and other resources during the deployment process.
