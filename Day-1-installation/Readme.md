# 📘 Terraform Guide

---

## 📌 What Is Terraform?

Terraform is an IaC tool, used primarily by DevOps teams to automate various infrastructure tasks. The provisioning of cloud resources, for instance, is one of the main use cases of Terraform. It’s a cloud-agnostic, open-source provisioning tool written in the Go language and created by HashiCorp.

Terraform allows you to describe your complete infrastructure in the form of code. Even if your servers come from different providers such as AWS or Azure, Terraform helps you build and manage these resources in parallel across providers. Think of Terraform as connective tissue and common language that you can utilize to manage your entire IT stack.

---

## 🚀 Benefits of Infrastructure-as-Code (IaC)

IaC replaces standard operating procedures and manual effort required for IT resource management with lines of code.

### Key Benefits:
- Speed and Simplicity
- Team Collaboration
- Error Reduction
- Disaster Recovery
- Enhanced Security

---

## 📁 Basic Terraform Folder Structure

```
projectname/
│
├── provider.tf
├── version.tf
├── backend.tf
├── main.tf
├── variables.tf
├── terraform.tfvars
├── outputs.tf
```

### Recommended Structure

```
provider.tf   → terraform block & provider
data.tf       → data sources
variables.tf  → variables
locals.tf     → local variables
output.tf     → outputs
```

---

## ⚙️ Important Terraform Commands

### Version
```
terraform –version
```

### Initialization
```
terraform init
terraform init -input=true
terraform init -lock=false
```

### Execution
```
terraform plan
terraform apply
terraform apply –auto-approve
terraform destroy –auto-approve
```

### Workspaces
```
terraform workspace new
terraform workspace select
terraform workspace list
terraform workspace show
terraform workspace delete
```

### Import
```
terraform import aws_instance.example i-abcd1234
```

---

## 📂 State File

### What is State?
Terraform must store state about your managed infrastructure and configuration.

### Remote State
Default:
```
terraform.tfstate
```

### State Lock
- Prevents corruption
- Enabled automatically

---

## ☁️ S3 Backend Configuration

```hcl
terraform {
  backend "s3" {
    encrypt = true    
    bucket = "sample"
    dynamodb_table = "terraform-state-lock-dynamo"
    key    = "terraform.tfstate"
    region = "us-east-1"
  }
}
```

---

## 🗄️ DynamoDB & S3 Setup

```hcl
resource "aws_s3_bucket" "example" {
  bucket = sample
}

resource "aws_dynamodb_table" "dynamodb-terraform-state-lock" {
  name = "terraform-state-lock-dynamo"
  hash_key = "LockID"
  read_capacity = 20
  write_capacity = 20

  attribute {
    name = "LockID"
    type = "S"
  }
}
```

---

## 📊 Data Source

```hcl
data "aws_vpc" "vpc" {
  id = vpc_id
}

data "aws_subnet" "subnet" {
  id = subnet_id
}
```

---

## 🧩 Terraform Import

```
terraform import aws_instance.myvm <Instance ID>
```

```hcl
resource "aws_instance" "myvm" {
  ami           = "unknown"
  instance_type = "unknown"
}
```

---

## 🔧 Meta-Arguments

- depends_on
- count
- for_each
- lifecycle
- provider
- provisioner
- connection
- variable
- output
- locals

---

## 🔗 depends_on

```hcl
resource "aws_instance" "dev" {
   ami = "ami-0440d3b780d96b29d"
   instance_type = "t2.micro"
   depends_on = [ aws_s3_bucket.example ]
}
```

---

## 🔢 count

```hcl
resource "aws_instance" "myec2" {
    ami = "ami-0230bd60aa48260c6"
    instance_type = "t2.micro"
    count = 2

    tags = {
      Name = "webec2-${count.index}"
    }
}
```

---

## 🔁 for_each

```hcl
resource "aws_instance" "sandbox" {
  ami           = var.ami
  instance_type = var.instance_type
  for_each      = var.sandboxes

  tags = {
    Name = each.value
  }
}
```

---

## 🌍 Multi Provider

```hcl
provider "aws" {
  region = "ap-south-1"
}

provider "aws" {
  region = "us-east-1"
  alias = "america"
}

resource "aws_s3_bucket" "test2" {
  bucket = "del-hyd-naresh-it-test2"
  provider = aws.america
}
```

---

## 🔄 Lifecycle

```hcl
lifecycle {
  create_before_destroy = true
}
```

---

## 📌 Locals

```hcl
locals {
  bucket-name = "${var.layer}-${var.env}-bucket-hydnaresh"
}
```

---

## ⚙️ Provisioners

### File
```hcl
provisioner "file" {
  source      = "conf/myapp.conf"
  destination = "/etc/myapp.conf"
}
```

### Local Exec
```hcl
provisioner "local-exec" {
  command = "echo ${self.private_ip} >> private_ips.txt"
}
```

### Remote Exec
```hcl
provisioner "remote-exec" {
  inline = [
    "touch file200",
    "echo hello from aws >> file200",
  ]
}
```

---

## 📦 Modules

```
modules/
  vpc/
    main.tf
    variables.tf
```

```hcl
module "my_vpc" {
  source     = "./modules/vpc"
  cidr_block = "10.0.0.0/16"
  name       = "my-vpc"
}
```

---

## 🔌 Connection Block

```hcl
connection {
  type        = "ssh"
  user        = "ubuntu"
  private_key = file("~/.ssh/id_rsa")
  host        = self.public_ip
}
```

---

## 📤 Output Values

```hcl
output "instance_public_ip" {
  value = aws_instance.test.public_ip
  sensitive = true
}
```

---

## ✅ Conditions

```hcl
variable "aws_region" {
  type = string
  default = "change me"

  validation {
    condition = var.aws_region == "us-west-2" || var.aws_region == "eu-west-1"
    error_message = "The variable 'aws_region' must be one of the following regions"
  }
}
```

---
