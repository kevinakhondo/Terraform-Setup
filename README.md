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

#### Step 2: Creating IAM User

Iam User is the human identity. Still in _main.tf_, add the following line:

```
resource "aws_iam_user" "data_engineer" {
  name = "kevin.data.engineer"
}

```
These are IAM users that represent humans and are used for AWS Console / CLI access.

#### Step 3: Creating IAM Groups
These are created because:
- Easier permission management
- Industry best practice
- Users inherit group permissions

Still in _main.tf_, add the following:

```
resource "aws_iam_group" "data_engineers" {
  name = "data-engineers"
}

```

#### Step 4: Add A User to A group

The created user data_engineer with the name kevin.data.engineer is added to the group data_engineers.
Still on _main.tf_, add the following:

```
resource "aws_iam_group_membership" "data_engineer_membership" {
  name = "data-engineers-membership"

  users = [
    aws_iam_user.data_engineer.name
  ]

  group = aws_iam_group.data_engineers.name
}

```
#### Step 5: Attach Read Only Policy to Created Group

The reason for attaching read only is:
- Engineers can inspect infra
- Prevents accidental destruction
- Very common in production

In the _main.tf_, add the following

```
resource "aws_iam_group_policy_attachment" "readonly" {
  group      = aws_iam_group.data_engineers.name
  policy_arn = "arn:aws:iam::aws:policy/ReadOnlyAccess"
}

```

#### Step 6: Create IAM Role for EC2
Roles are asssumed by AWS.

In the _main.tf_, add the following

```
resource "aws_iam_role" "ec2_data_role" {
  name = "ec2-data-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
}

```

#### Step 7: Define Variables
In this, we define the S3 bucket. In your _variables.tf_ , add the following:

```
variable "raw_bucket_name" {
  description = "Raw S3 bucket name"
  type        = string
}

```

Also, in your _terraform.tfvars_, add the following:

```
raw_bucket_name = "data-platform-raw-kevins"

```
That defines the actual name of the bucket.

#### Step 8: Define Custom IAM Policy for S3 Access
We use least previlege where :
- only one bucket is accessed.
- Provide only required actions.
- There is no wildcard access.

In _main.tf_, add the following

```
resource "aws_iam_policy" "s3_raw_access" {
  name = "s3-raw-access"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"

      Action = [
        "s3:PutObject",
        "s3:GetObject",
        "s3:ListBucket"
      ]

      Resource = [
        "arn:aws:s3:::${var.raw_bucket_name}",
        "arn:aws:s3:::${var.raw_bucket_name}/*"
      ]
    }]
  })
}

```

#### Step 9: Attach Policy to the Role

Add the following

```
resource "aws_iam_role_policy_attachment" "ec2_s3_access" {
  role       = aws_iam_role.ec2_data_role.name
  policy_arn = aws_iam_policy.s3_raw_access.arn
}

```
The effect of that is:

```
EC2 → IAM Role → S3 Raw Bucket

```


