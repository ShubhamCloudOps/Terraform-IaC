# Terrafrom

---

## What Is Terraform?

Terraform is an IAC tool, used primarily by DevOps teams to automate various infrastructure tasks. The provisioning of cloud resources, for instance, is one of the main use cases of Terraform. It’s a cloud-agnostic, open-source provisioning tool written in the Go language and created by HashiCorp.

Terraform allows you to describe your complete infrastructure in the form of code. Even if your servers come from different providers such as AWS or Azure, Terraform helps you build and manage these resources in parallel across providers. Think of Terraform as connective tissue and common language that you can utilize to manage your entire IT stack.

### --- Benefits of Infrastructure-as-Code (IaC) ---

IaC replaces standard operating procedures and manual effort required for IT resource management with lines of code. Instead of manually configuring cloud nodes or physical hardware, IaC automates the process infrastructure management through source code.

Here are several of the major key benefits of using an IaC solution like Terraform:

Speed and Simplicity. IaC eliminates manual processes, thereby accelerating the delivery and management lifecycles. IaC makes it possible to spin up an entire infrastructure architecture by simply running a script.

Team Collaboration. Various team members can collaborate on IaC software in the same way they would with regular application code through tools like Github. Code can be easily linked to issue tracking systems for future use and reference.

Error Reduction. IaC minimizes the probability of errors or deviations when provisioning your infrastructure. The code completely standardizes your setup, allowing applications to run smoothly and error-free without the constant need for admin oversight.

Disaster Recovery. With IaC you can actually recover from disasters more rapidly. Because manually constructed infrastructure needs to be manually rebuilt. But with IaC, you can usually just re-run scripts and have the exact same software provisioned again.

Enhanced Security. IaC relies on automation that removes many security risks associated with human error. When an IaC-based solution is installed correctly, the overall security of your computing architecture and associated data improves massively.

------ ### Basic Terraform folder structure ### ------

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

Give Terraform files logical names
Terraform tutorials online often demonstrate a directory structure consisting of three files:

main.tf:    contains all providers, resources and data sources
variables.tf: contains all defined variables
output.tf:    contains all output resources

The issue     with this structure is that most logic is stored in the single main.tf file which therefore becomes pretty complex and long. Terraform, however, does not mandate this structure, it only requires a directory of Terraform files. Since the filenames do not matter to Terraform I propose to use a structure that enables users to quickly understand the code. Personally I prefer the following structure

provider.tf: contains the terraform block and provider block
data.tf: contains all data sources
variables.tf: contains all defined variables
locals.tf: contains all local variables
output.tf: contains all output resources

---------- ##### Importent Terraform Commands ### --------

### Version

```
terraform –version
```

Shows terraform version installed

### Initialize infrastructure

```
terraform init
terraform init -input=true
terraform init -lock=false
```

Initialize a working directory
Ask for input if necessary
Disable locking of state files during state-related operations

```
terraform plan
terraform apply
terraform apply –auto-approve
terraform destroy –auto-approve
```

Creates an execution plan (dry run)
Executes changes to the actual environment
Apply changes without being prompted to enter ”yes”
Destroy/cleanup without being prompted to enter ”yes”

### Terraform Workspaces

```
terraform workspace new
terraform workspace select
terraform workspace list
terraform workspace show
terraform workspace delete
```

Create a new workspace and select it
Select an existing workspace
List the existing workspaces
Show the name of the current workspace
Delete an empty workspace

### Terraform Import

```
terraform import aws_instance.example i-abcd1234(instance id)
```

import an AWS instance with ID i-abcd1234 into aws_instance resource named “foo”

---## Statefile ##--:

## What is state and why is it important in Terraform?

Terraform must store state about your managed infrastructure and configuration. This state is used by Terraform to map real world resources to your configuration, keep track of metadata, and to improve performance for large infrastructures. This state file is extremely important; it maps various resource metadata to actual resource IDs so that Terraform knows what it is managing. This file must be saved and distributed to anyone who might run Terraform.

### Remote State

By default, Terraform stores state locally in a file named terraform.tfstate. When working with Terraform in a team, use of a local file makes Terraform usage complicated because each user must make sure they always have the latest state data before running Terraform and make sure that nobody else runs Terraform at the same time.

With remote state, Terraform writes the state data to a remote data store, which can then be shared between all members of a team.

### State Lock

If supported by your backend, Terraform will lock your state for all operations that could write state. This prevents others from acquiring the lock and potentially corrupting your state.

State locking happens automatically on all operations that could write state. You won’t see any message that it is happening. If state locking fails, Terraform will not continue. You can disable state locking for most commands with the -lock flag but it is not recommended.

## Setting up our S3 Backend

Create a new file in your working directory labeled Backend.tf

Copy and paste this configuration:

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

## Creating our DynamoDB Table

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

## DATA source

What is Data Source?

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

Why Terraform Import?

Terraform is a relatively new technology and adopting it to manage an organisation’s cloud resources might take some time and effort.

### Import Command

```
terraform import aws_instance.myvm <Instance ID>
```

---

## Meta-arguments

depends_on
count
for_each
lifecycle
provider
provisioner
connection
variable
output
locals

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

## Multi provider

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
