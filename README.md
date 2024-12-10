```markdown
# EC2 Instance Import and Provision Using Terraform

This repository provides a step-by-step guide to import an existing EC2 instance into Terraform and provision it into another AWS account.

---

## Prerequisites

- An EC2 instance already created manually in the **source AWS account**.
- AWS CLI configured on your system for both **source** and **destination** accounts.
- Terraform installed on your system.

---

## Steps to Import and Provision the EC2 Instance

### 1. Create an EC2 Instance for Configuration
- Launch a new EC2 instance to manage this process and configure AWS CLI with **source account credentials**.

### 2. Set Up the Local Development Environment
- Create a folder on your local machine and open it in VSCode.
- Inside the folder, create a file named `ec2_import.tf`.

### 3. Write Initial Terraform Configuration
Add the following to `ec2_import.tf`:
```hcl
provider "aws" {
  region = "us-east-1" # Update to your EC2 instance's region
}

resource "aws_instance" "imported_instance" {
  # The properties will be filled automatically after the import
}
```

### 4. Import the EC2 Instance
Run the following commands:
```bash
terraform init
terraform import aws_instance.imported_instance i-0c17868db5f07c2c0
```

### 5. Export EC2 Instance Configuration
Generate a configuration file using the following commands:
```bash
terraform plan -out=plan.out
terraform show -json plan.out > ec2_configuration.json
```

### 6. Update Terraform Configuration
Using the details from `ec2_configuration.json`, update `ec2_import.tf`:
```hcl
resource "aws_instance" "imported_instance" {
  ami           = "<AMI-ID>"          # Replace with the actual AMI ID
  instance_type = "<INSTANCE-TYPE>"   # Replace with the instance type (e.g., t2.micro)

  # Add other configurations like security group, tags, etc.
}
```

### 7. Configure Destination Account in Terraform
Update `ec2_import.tf` with the destination account details:
```hcl
# Source account provider
provider "aws" {
  alias  = "source"
  region = "ap-south-1"
}

# Destination account provider
provider "aws" {
  alias       = "destination"
  region      = "ap-south-1"
  access_key  = "" # Replace with your destination AWS Access Key
  secret_key  = "" # Replace with your destination AWS Secret Key
}

resource "aws_instance" "imported_instance" {
  provider      = aws.destination
  ami           = "ami-0614680123427b75e" # Replace with actual AMI ID
  instance_type = "t2.micro"              # Replace with actual instance type

  tags = {
    Name = "ImportedInstance"
  }
}
```

---

## Execution

1. Initialize Terraform:
   ```bash
   terraform init
   ```

2. Create a plan for the destination account:
   ```bash
   terraform plan -out=destination_plan.out
   ```

3. Apply the plan to provision the EC2 instance in the destination account:
   ```bash
   terraform apply destination_plan.out
   ```

---

## Verification
After running the above commands, log in to the **destination AWS account** and verify that the EC2 instance is created with the desired configurations.

---

---

## Resources
- [Terraform Documentation](https://www.terraform.io/docs)
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/index.html)

---

