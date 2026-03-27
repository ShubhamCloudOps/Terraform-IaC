# Terraform Notes

---

## Terrafrom

---

## What Is Terraform?

Terraform is an IAC tool, used primarily by DevOps teams to automate various infrastructure tasks. The provisioning of cloud resources, for instance, is one of the main use cases of Terraform. It’s a cloud-agnostic, open-source provisioning tool written in the Go language and created by HashiCorp.

Terraform allows you to describe your complete infrastructure in the form of code. Even if your servers come from different providers such as AWS or Azure, Terraform helps you build and manage these resources in parallel across providers. Think of Terraform as connective tissue and common language that you can utilize to manage your entire IT stack.

---

## Benefits of Infrastructure-as-Code (IaC)

IaC replaces standard operating procedures and manual effort required for IT resource management with lines of code. Instead of manually configuring cloud nodes or physical hardware, IaC automates the process infrastructure management through source code.

### Key Benefits

* Speed and Simplicity. IaC eliminates manual processes, thereby accelerating the delivery and management lifecycles. IaC makes it possible to spin up an entire infrastructure architecture by simply running a script.
* Team Collaboration. Various team members can collaborate on IaC software in the same way they would with regular application code through tools like Github. Code can be easily linked to issue tracking systems for future use and reference.
* Error Reduction. IaC minimizes the probability of errors or deviations when provisioning your infrastructure.
* Disaster Recovery. With IaC you can recover from disasters rapidly by re-running scripts.
* Enhanced Security. Automation reduces human errors and improves security posture.

---

## Basic Terraform Folder Structure

```
projectname/
    |
    |-- provider.tf
    |-- version.tf
    |-- backend.tf
    |-- main.tf
    |-- variables.tf
    |-- terraform.tfvars
    |-- outputs.tf
```

### Recommended Structure

* provider.tf: contains the terraform block and provider block
* data.tf: contains all data sources
* variables.tf: contains all defined variables
* locals.tf: contains all local variables
* output.tf: contains all output resources

---

## Important Terraform Commands

### Version

```
terraform –version
```

### Initialize infrastructure

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

### Terraform Workspaces

```
terraform workspace new
terraform workspace select
terraform workspace list
terraform workspace show
terraform workspace delete
```

### Terraform Import

```
terraform import aws_instance.example i-abcd1234(instance id)
```

---

## Statefile

### What is state and why is it important in Terraform?

Terraform must store state about your managed infrastructure and configuration. This state is used by Terraform to map real world resources to your configuration, keep track of metadata, and to improve performance for large infrastructures.

### Remote State

By default, Terraform stores state locally in terraform.tfstate.

### State Lock

If supported by your backend, Terraform will lock your state.

---

## Setting up S3 Backend

```
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

## Creating DynamoDB Table

```
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

## Data Source

### What is Data Source?

Data source in terraform relates to resources but only it gives the information about an object rather than creating one.

### Example

```
data "aws_vpc" "vpc" {
  id = vpc_id
}

data "aws_subnet" "subnet" {
  id = subnet_id
}
```

---

## Terraform Import

### Import Command

```
terraform import aws_instance.myvm <Instance ID>
```

---

## Meta-Arguments

* depends_on
* count
* for_each
* lifecycle
* provider
* provisioner
* connection
* variable
* output
* locals

---

## depends_on

```
resource "aws_instance" "dev" {
   ami = "ami-0440d3b780d96b29d"
   instance_type = "t2.micro"
   depends_on = [ aws_s3_bucket.example]
}
```

---

## count

```
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

## for_each

```
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

## Multi Provider

```
provider "aws" {
  region = "ap-south-1"
}

provider "aws" {
  region = "us-east-1"
  alias = "america"
}
```

---

## Lifecycle

```
lifecycle {
  create_before_destroy = true
}
```

---

## Locals

```
locals {
  bucket-name = "${var.layer}-${var.env}-bucket-hydnaresh"
}
```

---

## Provisioners

### File Provisioner

```
provisioner "file" {
  source      = "conf/myapp.conf"
  destination = "/etc/myapp.conf"
}
```

### local-exec

```
provisioner "local-exec" {
  command = "echo ${self.private_ip} >> private_ips.txt"
}
```

### remote-exec

```
provisioner "remote-exec" {
  inline = [
    "touch file200",
    "echo hello from aws >> file200",
  ]
}
```

---

## Modules

### Example

```
module "my_vpc" {
  source     = "./modules/vpc"
  cidr_block = "10.0.0.0/16"
  name       = "my-vpc"
}
```

---

## Connection Block

```
connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = file("~/.ssh/id_rsa")
    host        = self.public_ip
}
```

---

## Output Values

```
output "instance_public_ip" {
    value = aws_instance.test.public_ip
    sensitive = true
}
```

---

## Conditions Meta Arguments

```
variable "aws_region" {
  description = "The region in which to create the infrastructure"
  type        = string
  nullable    = false
  default     = "change me"
  validation {
    condition = var.aws_region == "us-west-2" || var.aws_region == "eu-west-1"
    error_message = "The variable 'aws_region' must be one of the following regions: us-west-2, eu-west-1"
  }
}
```

---
