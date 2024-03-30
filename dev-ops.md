# AWS
- Register AWS free account.
- Create IAM user `name`
- Save credantials (access and secret)
- Attach `AdministratorAccess` policy to this user
- Install Terraform and aws cli
- Configure default aws cli profile:

```
    % aws configure
    AWS Access Key ID : <Access Key>
    AWS Secret Access Key : <Secret Key>
    Default region name []: us-east-1
    Default output format [None]:
```
- To install terraform, use binary - https://www.terraform.io/downloads.html
- Relocate file to
```bash
sudo mv terraform /usr/local/bin/
```
- Getting started guide - https://cloudkatha.com/getting-started-with-terraform-on-aws-in-right-way/
- Good VPC creating (AWS) tutorial: https://spacelift.io/blog/terraform-aws-vpc
- Creating EC2 instances via Security Groups - https://www.middlewareinventory.com/blog/terraform-aws-example-ec2/#Create_EC2_instance_with_Terraform_Terraform_EC2
