# Terraform-Setup
Create your first terraform project

## Prerequisites

- Ensure that you have an AWS Account,  and VS Studio prior to this.
- On your terminal, check if terraform is installed by running _terraform version_
If not, follow this guide to install it https://developer.hashicorp.com/terraform/install

- Next, install AWS CLI.Terraform talks to AWS through the AWS CLI credentials.Check the version using _aws --version_
  . If it is missing, again install it using  _brew install awscli_ .
- Next, you need to create IAM user for Terraform. Do not use root credentials. Navigate to IAM create terraform user named _terraform-admin_.

<img width="1680" height="607" alt="image" src="https://github.com/user-attachments/assets/fe1bedf9-ec73-4299-9995-131ff2fd6994" />


Then, attach a policy AdministratorAccess. This is for learning purposes. once data, you need to generate an access and secret key. Save it.

-   Configure AWS Credentials locally on your machine.

On your terminal, run _aws configure_ and enter 

- AWS Access Key ID:     from IAM  
- AWS Secret Access Key: from IAM  
- Default region:         us-east-1   
- Default output:         json        

After correctly entering the above info, confirm if it's correct by running _aws sts get-caller-identity_ on your terminal. 
You should see your account ID and user name.

## Setting Up VS Code for Terraform
- Under VS Extension, install the HashiCorp Terraform.
- Next, create folder where everything will be stored.


_mkdir aws-data-engineering_
  
_cd aws-data-engineering_

_mkdir module-02-iam-terraform_

_cd module-02-iam-terraform_

- In the VS Studio, open the created folder and create the _main.tf_ file. Enter the code:

```
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}
```
<img width="1680" height="426" alt="image" src="https://github.com/user-attachments/assets/cc776eb1-5397-4d87-9120-467db10a7a34" />

This tells Terraform: Which providers to use and Which AWS region.

## Initialize Terraform
Back in the terminal, while in the folder, run

```
terraform init
```
This: Downloads AWS provider and Prepares the working directory

Further, on the vs studio, add the following folders

```
main.tf
variables.tf
terraform.tfvars
outputs.tf

```
### Hands-On

In this, we will build the following using terraform

```
Human access:
IAM User ──▶ IAM Group ──▶ ReadOnly Policy

Service access:
EC2 ──▶ IAM Role ──▶ Custom S3 Policy

```

#### Step 1: Setting Up terraform and provider

In your _main.tf_ on the VS studio, copy and paste the following

```
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

```

The above lines:
- Locks Terraform version
- Locks AWS provider version
- Makes your repo reproducible

The next part is creating provider configuration

```
provider "aws" {
  region = "us-east-1"
}

```
From the above line, Terraform creates everything in _us-east-1_.
