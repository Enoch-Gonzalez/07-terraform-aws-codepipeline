# Terraform VPC 3-Tier Architecture implementing CloudWatch + ALB + Autoscaling with Launch Templates using AWS CodePipeline

This Terraform project aims to create a VPC with a 3-tier architecture on AWS. The architecture consists of a public subnet containing a bastion EC2 instance, a private subnet where ec2 instances will be launch using launch templates with autoscaling, an Apache server installed in the ec2 instances launched. An index.html file along with a metadata  will be displayed for the EC2 instances. Terraform Backend with backend-config will be used.

![07-terraform-pipeline](https://github.com/Enoch-Gonzalez/images/blob/main/07-terraform-pipeline.png)

Two different environments dev and stag will be created. AWS CodePipeline paired to Github will be use to automate the communication process between the dev and stag environment.

![07-terraform-dev-env](https://github.com/Enoch-Gonzalez/images/blob/main/07-terraform-dev-env.png)

![07-terraform-stag-env](https://github.com/Enoch-Gonzalez/images/blob/main/07-terraform-stag-env.png)

## Prerequisites
	
- AWS Account: You need an active AWS account.
- AWS CLI: Install and configure AWS CLI on your local machine.
- SSH Key Pair: Generate an SSH key pair and place the private key (key.pem) in the private-key directory.
- Secure Parameters: Create environmental variables for the Access Key (AWS_ACCESS_KEY_ID) and the Secret Access key (AWS_SECRET_ACCESS_KEY) on the AWS Parameters Store IN AWS System Manager.
- Terraform: Install Terraform on your local machine.
- It will be required to have a registered Domain in AWS Route53 to implement this usecase.
- Github Account: You need an active Github Account.


## Usage

1. Clone this repository to your local machine.

2. Ensure Terraform is installed on your system.

3. Set up your AWS credentials using the AWS CLI:

    ```bash
    aws configure
    ```

4. Modify the c2-variable.tf file to adjust any desired settings such as region, instance type, and key pair.

5. Modify variables in terraform.tfvars and other .tfvars files if needed.

## Domain Configuration

This project assumes the use of a registered domain in AWS Route53 for certain components. Before running Terraform, ensure to modify the following files located in the project directory to update the domain name:

1. `terraform-manifest/c6-02-datasource-route53-zone.tf`:
    - Locate and update the `domain_name` field in this file with your registered domain name.
    - Example: Change `<your-route53-domain>.com` to `*.yourdomain.com`.

2. `terraform-manifest/c11-acm-certificatemanager.tf`:
    - Locate and update the `domain_name` field in this file with your registered domain name.
    - Example: Change `*.<your-route53-domain>.com` to `*.yourdomain.com`.

3. `terraform-manifest/dev.tfvars`:
    - Update the `dns_name` field to reflect your chosen subdomain.
    - Example: Change `”devdemo5.<your-route53-domain>.com"` to `"devdemo5.your-subdomain.yourdomain.com"`.

4. `terraform-manifest/stag.tfvars`:
    - Update the `dns_name` field to reflect your chosen subdomain.
    - Example: Change `”devdemo5.<your-route53-domain>.com"` to `"devdemo5.your-subdomain.yourdomain.com"`.

    **Note**: Make sure to replace placeholders with your actual domain and subdomain names before executing Terraform commands.

## SNS Configuration

This project utilizes SNS. Before running Terraform, ensure to modify the following file located in the project directory to update your SNS email:

1. `terraform-manifest/c13-05-autoscaling-notifications.tf`:
    - Locate and update the `endpoint` field and pass the desired email that will be used to send the SNS.
    - Example: Change `<your-email>@gmail.com` to `your-email@gmail.com`.

## AWS Secure Parameters Configuration

As per a Prerequisite the Secure Parameters in Parameter Store must be configure.

1. Create MY_AWS_SECRET_ACCESS_KEY

	- Go to Services -> Systems Manager -> Application Management -> Parameter Store -> Create Parameter
	    + Name: /CodeBuild/MY_AWS_ACCESS_KEY_ID
	    + Descritpion: My AWS Access Key ID for Terraform CodePipeline Project
	    + Tier: Standard
	    + Type: Secure String
	    + Rest all defaults
	    + Value: `<YOUR-IAM-ACCES-KEY>`

2. Create MY_AWS_SECRET_ACCES_KEY
	
    - Go to Services -> Systems Manager -> Application Management -> Parameter Store -> Create Parameter
	    + Name: /CodeBuild/MY_AWS_SECRET_ACCESS_KEY
	    + Descritpion: My AWS Secret Access Key for Terraform CodePipeline Project
	    + Tier: Standard
	    + Type: Secure String
	    + Rest all defaults
	    + Value: `<YOUR-IAM-SECRET-ACCES-KEY>`

## Github Repository and Check-In file Configuration

1. Create a New Github Repository

    - Go to github.com and login with your credentials
    - URL: `https://github.com/Enoch-Gonzalez` (my git repo url)
	- Click on Repositories Tab
	- Click on New to create a new repository
	- Repository Name: terraform-iacdevops-with-aws-codepipeline
	- Description: Implement Terraform IAC DevOps for AWS Project with AWS CodePipeline
	- Repository Type: Private
	- Choose License: MIT License
	- Click on Create Repository
	- Click on Code and Copy Repo link
    - Clone Remote and Copy all related files
        + Change Directory

        ```bash
        cd demo-repos
        ```

        + Execute Git Clone

        ```bash    
        git clone https://github.com/Enoch-Gonzalez/07-terraform-aws-codepipeline.git
        ```

    - Copy all files from the repo
        + Source Folder Path: 07-terraform-aws-codepipeline/
        + Copy all files from Source Folder to Destination Folder
        + Destination Folder Path: demo-repos/terraform-iacdevops-with-aws-codepipeline
    
    - Verify Git Status

        ```bash
        git status
        ```

    - Git Commit
        
        ```bash
        git commit -am "First Commit"
        ```

    - Push files to Remote Repository
    
        ```bash
        git push
        ```

    - Verify same on Remote Repository

        ```bash
        https://github.com/`<YOUR-GITHUB-USER>`/terraform-iacdevops-with-aws-codepipeline.git
        ```

    - Verify if AWS Connector from AWS Developer Tools
	    + Go to below url and verify
	    + URL: `https://github.com/settings/installations`
    - Create Github Connection from AWS Developer Tools
	    + Go to Services -> CodePipeline -> Create Pipeline
	    + In Developer Tools -> Click on Settings -> Connections -> Create Connection
	    + Select Provider: Github
	    + Connection Name: terraform-iacdevops-aws-cp-con1
	    + Click on Connect to Github
	    + GitHub Apps: Click on Install new app
	    + It should redirect to github page Install AWS Connector for GitHub
	    + Only select repositories: terraform-iacdevops-with-aws-codepipeline
	    + Click on Install
	    + Click on Connect
	    + Verify Connection Status: It should be in Available state
	    + Go to below url and verify
	    + URL: https://github.com/settings/installations
	    + You should see Install AWS Connector for GitHub app installed

## Getting Started

1. Create AWS CodePipeline

Go to Services -> CodePipeline -> Create Pipeline

- Pipeline settings     
    + Pipeline Name: tf-iacdevops-aws-cp1
    + Service role: New Service Role
    + rest all defaults
    + Artifact store: Default Location
    + Encryption Key: Default AWS Managed Key
    + Click Next

- Source Stage
    + Source Provider: Github (Version 2)
    + Connection: terraform-iacdevops-aws-cp-con1
    + Repository name: terraform-iacdevops-with-aws-codepipeline
    + Branch name: main
    + Change detection options: leave to defaults as checked
    + Output artifact format: leave to defaults as CodePipeline default

- Add Build Stage
    + Build Provider: AWS CodeBuild
    + Region: N.Virginia
    + Project Name: Click on Create Project
    + Project Name: codebuild-tf-iacdevops-aws-cp1
    + Description: CodeBuild Project for Dev Stage of IAC DevOps Terraform Demo
    + Environment image: Managed Image
    + Operating System: Amazon Linux 2
    + Runtimes: Standard
    + Image: latest available today (aws/codebuild/amazonlinux2-x86_64-standard:3.0)
    + Environment Type: Linux
    + Service Role: New (leave to defaults including Role Name)
    + Build specifications: use a buildspec file
    + Buildspec name - optional: buildspec-dev.yml (Ensure that this file is present in root folder of your github repository)
    + Rest all leave to defaults
    + Click on Continue to CodePipeline
    + Project Name: This value should be auto-populated with codebuild-tf-iacdevops-aws-cp1
    + Build Type: Single Build
    + Click Next

- Add Deploy Stage
    + Click on Skip Deploy Stage

- Review Stage
    + Click on Create Pipeline


## Verify the Pipeline created

1. Verify Source Stage: Should pass

2. Verify Build Stage: should fail with error

3. Verify Build Stage logs by clicking on details in pipeline screen


## Verify Resources and Test

1. Confirm SNS Subscription in your email

2. Verify EC2 Instances

3. Verify Launch Templates (High Level)

4. Verify Autoscaling Group (High Level)

5. Verify Load Balancer

6. Verify Load Balancer Target Group - Health Checks
	
7. Access and Test

- Go to:

    ```
    http://devdemo1.<your-route53-domain>.com
    ```

    ```
    http://devdemo1.<your-route53-domain>.com/app1/index.html
    ```

    ```
    http://devdemo1.<your-route53-domain>.com/app1/metadata.html
    ```

- Upload index.html and test

    +  Go to AWS Services -> S3 -> Buckets
        * Click on the bucket that was created
        * Click “Upload”
        * Click on “Add files”
        * Path/to/the/the/directory/static-website/index.html

- Test

    + Go to your browser
        * Endpoint Format test
	    
        ```
        http://example-bucket.s3-website.Region.amazonaws.com/
        ```

        * Replace Values (Bucket Name, Region)
	    
        ```
        http://mybucket-1047.s3-website.us-east-1.amazonaws.com/
        ```

- Add Approval Stage before deploying to staging environment

    + Go to Services -> AWS CodePipeline -> tf-iacdevops-aws-cp1 -> Edit

- Add Stage

    + Name: Email-Approval

- Add Action Group

    + Action Name: Email-Approval
	+ Action Provider: Manual Approval
	+ SNS Topic: Select SNS Topic from drop down
	+ Comments: Approve to deploy to staging environment


- Add Staging Environment Deploy Stage

    + Go to Services -> AWS CodePipeline -> tf-iacdevops-aws-cp1 -> Edit

- Add Stage

    + Name: Stage-Deploy

- Add Action Group

    + Action Name: Stage-Deploy
    + Region: US East (N.Virginia)
    + Action Provider: AWS CodeBuild
    + Input Artifacts: Source Artifact
    + Project Name: Click on Create Project
    + Project Name: stage-deploy-tf-iacdevops-aws-cp1
    + Description: CodeBuild Project for Staging Environment of IAC DevOps Terraform Demo
    + Environment image: Managed Image
    + Operating System: Amazon Linux 2
    + Runtimes: Standard
    + Image: latest available today (aws/codebuild/amazonlinux2-x86_64-standard:3.0)
    + Environment Type: Linux
    + Service Role: New (leave to defaults including Role Name)
    + Build specifications: use a buildspec file
    + Buildspec name - optional: buildspec-stag.yml (Ensure that this file is present in root folder of your github repository)
    + Rest all leave to defaults
    + Click on Continue to CodePipeline
    + Project Name: This value should be auto-populated with stage-deploy-tf-iacdevops-aws-cp1
    + Build Type: Single Build
    + Click on Done
    + Click on Save

- Update the IAM Role

Update the IAM Role created as part of this stage-deploy-tf-iacdevops-aws-cp1 CodeBuild project by adding the policy systems-manger-get-parameter-access1

- Run the Pipeline
    
    + Go to Services -> AWS CodePipeline -> tf-iacdevops-aws-cp1
	+ Click on Release Change
	+ Verify Source Stage
	+ Verify Build Stage (Dev Environment - Dev Depploy phase)
	+ Verify Manual Approval Stage - Approve the change
	+ Verify Stage Deploy Stage
	+ Verify build logs

## Verify Staging Environment and Test

1. Confirm SNS Subscription in your email

2. Verify EC2 Instances

3. Verify Launch Templates (High Level)

4. Verify Autoscaling Group (High Level)

5. Verify Load Balancer

6. Verify Load Balancer Target Group - Health Checks

7. Access and Test

- Go to: 

    ```
    http://stagedemo1.<your-route53-domain>.com
    ```

    ```
    http://stagedemo1.<your-route53-domain>.com/app1/index.html
    ```

    ```
    http://stagedemo1.<your-route53-domain>.com/app1/metadata.html
    ```

### Cleanup

1. Update buildspec-dev.yml
    
    - Before
        TF_COMMAND: "apply"
        #TF_COMMAND: "destroy"
    - After
        #TF_COMMAND: "apply"
        TF_COMMAND: "destroy"  

2. Update build spec-stag.yml

    - Before
        TF_COMMAND: "apply"
        #TF_COMMAND: "destroy"
    - After
        #TF_COMMAND: "apply"
        TF_COMMAND: "destroy"    

3. Commit changes via Git Repo

    - Verify Changes
    
    ```bash    
    git status
    ```

    - Commit Changes to Local Repository
    
    ```bash
    git add .
    ```
    
    ```bash
    git commit -am "Destroy Resources"
    ```

    - Push changes to Remote Repository
    
    ```bash
    git push
    ```

    **Note**: Remember to destroy resources once you're done to avoid incurring charges.
    
## License

This project is licensed under MIT License.