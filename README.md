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
_______________________________________
|AWS Access Key ID:     |<from IAM>   |
|AWS Secret Access Key: |<from IAM>   | 
|Default region:        | us-east-1   |
|Default output:        | json        |
______________________________________
