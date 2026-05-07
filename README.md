# 🏗️ Terraform — Complete Notes & Reference Guide

> A comprehensive reference covering Terraform concepts, commands, state management, modules, meta-arguments, provisioners, and more.

---

## 📚 Table of Contents

- [What Is Terraform?](#what-is-terraform)
- [Benefits of Infrastructure-as-Code (IaC)](#benefits-of-infrastructure-as-code-iac)
- [Terraform Folder Structure](#terraform-folder-structure)
- [Important Terraform Commands](#important-terraform-commands)
- [State File](#state-file)
  - [Remote State](#remote-state)
  - [State Lock](#state-lock)
  - [Setting up S3 Backend](#setting-up-s3-backend)
  - [Creating DynamoDB Table for State Lock](#creating-dynamodb-table-for-state-lock)
- [Data Sources](#data-sources)
  - [Example: VPC & Subnet via Data Source](#example-vpc--subnet-via-data-source)
  - [Example: Fetching AMI via Data Source](#example-fetching-ami-via-data-source)
- [Terraform Import](#terraform-import)
  - [Step-by-Step Import Guide](#step-by-step-import-guide)
- [Meta-Arguments](#meta-arguments)
  - [depends\_on](#depends_on)
  - [count](#count)
  - [for\_each](#for_each)
  - [Multi-Provider (provider alias)](#multi-provider-provider-alias)
  - [lifecycle](#lifecycle)
- [Locals](#locals)
- [Provisioners](#provisioners)
  - [File Provisioner](#file-provisioner)
  - [local-exec Provisioner](#local-exec-provisioner)
  - [remote-exec Provisioner](#remote-exec-provisioner)
- [Connection Block](#connection-block)
- [Output Values](#output-values)
- [Variable Validation (Conditions)](#variable-validation-conditions)
- [Modules](#modules)
  - [Benefits of Modules](#benefits-of-modules)
  - [Example 1: AWS VPC Module](#example-1-aws-vpc-module)
  - [Example 2: AWS EC2 Instance Module](#example-2-aws-ec2-instance-module)
  - [Module Source from GitHub](#module-source-from-github)

---

## What Is Terraform?

Terraform is an **IaC (Infrastructure as Code)** tool, used primarily by DevOps teams to automate various infrastructure tasks. The provisioning of cloud resources is one of the main use cases of Terraform.

- **Cloud-agnostic** — works across AWS, Azure, GCP, and more
- **Open-source** — created by HashiCorp, written in Go
- Allows you to describe your **complete infrastructure in the form of code**
- Even if your servers come from different providers such as AWS or Azure, Terraform helps you **build and manage these resources in parallel** across providers
- Think of Terraform as **connective tissue and common language** to manage your entire IT stack

---

## Benefits of Infrastructure-as-Code (IaC)

IaC replaces standard operating procedures and manual effort required for IT resource management with lines of code. Instead of manually configuring cloud nodes or physical hardware, IaC automates the process of infrastructure management through source code.

| Benefit | Description |
|---|---|
| **Speed and Simplicity** | Eliminates manual processes, accelerating delivery and management lifecycles. Spin up an entire infrastructure by simply running a script. |
| **Team Collaboration** | Team members can collaborate on IaC software the same way they would with regular code through tools like GitHub. Code can be linked to issue tracking systems. |
| **Error Reduction** | Minimizes the probability of errors or deviations when provisioning infrastructure. Completely standardizes your setup, allowing applications to run smoothly. |
| **Disaster Recovery** | Recover from disasters more rapidly. With IaC, you can re-run scripts to provision the exact same software again without manual rebuild. |
| **Enhanced Security** | Relies on automation that removes many security risks associated with human error. Improves the overall security of your computing architecture. |

---

## Terraform Folder Structure

### Basic Structure

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

### Recommended File Naming Convention

Terraform tutorials often demonstrate a three-file structure (`main.tf`, `variables.tf`, `output.tf`), but most logic ends up in `main.tf`, making it complex. Since filenames don't matter to Terraform, a more descriptive structure is recommended:

| File | Purpose |
|---|---|
| `provider.tf` | Contains the `terraform` block and `provider` block |
| `main.tf` | Contains all resources |
| `data.tf` | Contains all data sources |
| `variables.tf` | Contains all defined variables |
| `locals.tf` | Contains all local variables |
| `output.tf` | Contains all output resources |
| `backend.tf` | Contains backend (remote state) configuration |

---

## Important Terraform Commands

### Version

```shell
terraform -version         # Shows terraform version installed
```

### Initialize Infrastructure

```shell
terraform init                  # Initialize a working directory
terraform init -input=true      # Ask for input if necessary
terraform init -lock=false      # Disable locking of state files during state-related operations
```

### Plan, Apply & Destroy

```shell
terraform plan                    # Creates an execution plan (dry run)
terraform apply                   # Executes changes to the actual environment
terraform apply --auto-approve    # Apply changes without being prompted to enter "yes"
terraform destroy --auto-approve  # Destroy/cleanup without being prompted to enter "yes"
```

### Terraform Workspaces

```shell
terraform workspace new      # Create a new workspace and select it
terraform workspace select   # Select an existing workspace
terraform workspace list     # List the existing workspaces
terraform workspace show     # Show the name of the current workspace
terraform workspace delete   # Delete an empty workspace
```

### Terraform Import

```shell
# Import an AWS instance with ID i-abcd1234 into aws_instance resource named "example"
terraform import aws_instance.example i-abcd1234
```

---

## State File

> *"Terraform must store state about your managed infrastructure and configuration. This state is used by Terraform to map real world resources to your configuration, keep track of metadata, and to improve performance for large infrastructures. This state file is extremely important; it maps various resource metadata to actual resource IDs so that Terraform knows what it is managing. This file must be saved and distributed to anyone who might run Terraform."*

### Remote State

> *"By default, Terraform stores state locally in a file named `terraform.tfstate`. When working with Terraform in a team, use of a local file makes Terraform usage complicated because each user must make sure they always have the latest state data before running Terraform and make sure that nobody else runs Terraform at the same time."*

> *"With remote state, Terraform writes the state data to a remote data store, which can then be shared between all members of a team."*

### State Lock

> *"If supported by your backend, Terraform will lock your state for all operations that could write state. This prevents others from acquiring the lock and potentially corrupting your state."*

> *"State locking happens automatically on all operations that could write state. You won't see any message that it is happening. If state locking fails, Terraform will not continue. You can disable state locking for most commands with the `-lock` flag but it is not recommended."*

### Setting up S3 Backend

Create a new file in your working directory labeled `backend.tf` and add the following:

```hcl
terraform {
  backend "s3" {
    encrypt        = true    
    bucket         = "sample"
    dynamodb_table = "terraform-state-lock-dynamo"
    key            = "terraform.tfstate"
    region         = "us-east-1"
  }
}
```

> ⚠️ **Note:** Before configuring the backend, you must first create the S3 bucket and DynamoDB table resources that will be referenced in `backend.tf`.

### Creating DynamoDB Table for State Lock

Create a new file `dynamo.tf` (or add to `main.tf`):

```hcl
# S3 Bucket
resource "aws_s3_bucket" "example" {
  bucket = "sample"
}

# DynamoDB Table for State Locking
resource "aws_dynamodb_table" "dynamodb-terraform-state-lock" {
  name           = "terraform-state-lock-dynamo"
  hash_key       = "LockID"
  read_capacity  = 20
  write_capacity = 20

  attribute {
    name = "LockID"
    type = "S"
  }
}
```

---

## Data Sources

### What is a Data Source?

Data source in Terraform relates to resources but **only gives information about an object** rather than creating one. It provides dynamic information about entities defined **outside of Terraform**.

- Allows fetching data about infrastructure components' configuration
- Fetches data from cloud provider APIs using Terraform scripts
- When you refer to a resource using a data source, it **won't create the resource** — it gets information about it so you can use it in further configuration

### Example: VPC & Subnet via Data Source

**Step 1: `provider.tf`**

```hcl
provider "aws" {
  region     = "us-east-1"
  access_key = "your_access_key"
  secret_key = "your_secret_key" # Keys no need to configure here — they will be called from .aws folder locally
}
```

**Step 2: `demo_datasource.tf`**

```hcl
data "aws_vpc" "vpc" {
  id = vpc_id
}

data "aws_subnet" "subnet" {
  id = subnet_id
}

# Creating security group by calling existing VPC using data source block
resource "aws_security_group" "sg" {
  name   = "sg"
  vpc_id = data.aws_vpc.vpc.id

  ingress = [
    {
      cidr_blocks     = ["0.0.0.0/0"]
      description     = ""
      from_port       = 22
      protocol        = "tcp"
      security_groups = []
      self            = false
      to_port         = 22
    }
  ]

  egress = [
    {
      cidr_blocks     = ["0.0.0.0/0"]
      description     = ""
      from_port       = 0
      protocol        = "-1"
      security_groups = []
      self            = false
      to_port         = 0
    }
  ]
}

resource "aws_instance" "dev" {
  ami             = data.aws_ami.amzlinux.id
  instance_type   = "t2.micro"
  subnet_id       = data.aws_subnet.dev.id
  security_groups = [data.aws_security_group.dev.id]

  tags = {
    Name = "DataSource-Instance"
  }
}
```

> In the above code, we are using a VPC and a subnet already created on AWS. The `data` block fetches information about them. The security group uses the `vpc_id` fetched via data block, and the EC2 instance uses the `subnet_id` fetched via data block.

### Example: Fetching AMI via Data Source

```hcl
data "aws_ami" "amzlinux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-gp2"]
  }

  filter {
    name   = "root-device-type"
    values = ["ebs"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  filter {
    name   = "architecture"
    values = ["x86_64"]
  }
}
```

---

## Terraform Import

### Why Terraform Import?

Terraform is a relatively new technology and teams often start using cloud infrastructure directly via their respective web consoles before adopting IaC.

- **Getting pre-existing cloud resources under Terraform management** is facilitated by `terraform import`
- `import` is a Terraform CLI command which reads real-world infrastructure and updates the state so that future updates can be applied via IaC
- The import functionality helps **update the state locally** — it does **not** create the corresponding configuration automatically

### Step-by-Step Import Guide

#### 1. Prepare the EC2 Instance

For this example, the target EC2 instance has:
- **Name:** MyVM
- **Instance ID:** `i-0b9be609418aa0609`
- **Type:** `t2.micro`
- **VPC ID:** `vpc-1827ff72`

#### 2. Create `main.tf` and Set Provider Configuration

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}
```

Initialize the directory:

```shell
terraform init
```

Expected output:

```
Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/aws versions matching "~> 3.0"...
- Installing hashicorp/aws v3.51.0...
- Installed hashicorp/aws v3.51.0 (signed by HashiCorp)

Terraform has been successfully initialized!
```

#### 3. Write Config for Resource To Be Imported

Append the `main.tf` file with a minimal EC2 config (ami and instance_type are required arguments):

```hcl
resource "aws_instance" "myvm" {
  ami           = "unknown"  # update from state file reference
  instance_type = "unknown"  # update from state file reference
}
```

#### 4. Run the Import Command

```shell
terraform import aws_instance.myvm <Instance_ID>
```

Example:

```shell
terraform import aws_instance.myvm i-0b9be609418aa0609
```

Successful output:

```
aws_instance.myvm: Importing from ID "i-0b9be609418aa0609"...
aws_instance.myvm: Import prepared!
  Prepared aws_instance for import
aws_instance.myvm: Refreshing state... [id=i-0b9be609418aa0609]

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.
```

#### 5. Observe State Files and Plan Output

After import, the directory will contain a `terraform.tfstate` file. Run `terraform plan` to compare the state and the configuration. The plan may indicate replacements — the goal is to close the gap by updating the config.

#### 6. Improve Config to Avoid Replacement

Find attributes that cause replacement (highlighted with `~` in the plan). Update the config to match what's in the state file. Example — change `ami` from `"unknown"` to the actual AMI ID:

```hcl
resource "aws_instance" "myvm" {
  ami           = "ami-00f22f6155d6d92c5"
  instance_type = "t2.micro"

  tags = {
    "Name" = "MyVM"
  }
}
```

#### 7. Verify No Changes

```shell
terraform plan
```

Successful output:

```
aws_instance.myvm: Refreshing state... [id=i-0b9be609418aa0609]

No changes. Your infrastructure matches the configuration.
```

Congratulations — the cloud resource is now fully managed by Terraform!

---

## Meta-Arguments

Meta-arguments in Terraform are special arguments that can be used with **resource blocks and modules** to control their behavior or influence the infrastructure provisioning process.

| Meta-Argument | Description |
|---|---|
| `depends_on` | Specifies dependencies between resources — ensures one is created before another |
| `count` | Controls resource instantiation by setting the number of instances created |
| `for_each` | Creates multiple instances of a resource based on a map or set of strings |
| `lifecycle` | Defines lifecycle rules for managing resource updates, replacements, and deletions |
| `provider` | Specifies the provider configuration for a resource |
| `provisioner` | Specifies actions taken on a resource after creation (scripts, commands) |
| `connection` | Defines connection details to a resource for remote execution or file transfers |
| `variable` | Declares input variables provided during Terraform execution |
| `output` | Declares output values displayed after Terraform execution |
| `locals` | Defines local values used within configuration files |

---

### depends_on

Terraform internally knows the sequence in which dependent resources need to be created. However, in some scenarios **dependencies cannot be automatically inferred**. In these cases, use `depends_on` to explicitly define the dependency.

> - Must be a list of references to other resources in the same calling module
> - Available in resources and modules (Terraform version 0.13+)

#### Example 1 — EC2 depends on S3 Bucket

```hcl
provider "aws" {}

resource "aws_s3_bucket" "example" {
  bucket = "qwertyuiopasdfg"
}

resource "aws_instance" "dev" {
  ami           = "ami-0440d3b780d96b29d"
  instance_type = "t2.micro"

  # EC2 will only be created after S3 bucket is created successfully
  depends_on = [aws_s3_bucket.example]
}
```

#### Example 2 — EC2 depends on IAM Role

```hcl
### Create IAM policy
resource "aws_iam_policy" "example_policy" {
  name        = "example_policy"
  description = "Permissions for EC2"
  policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Action   = "ec2:*"
        Effect   = "Allow"
        Resource = "*"
      }
    ]
  })
}

### Create IAM role
resource "aws_iam_role" "example_role" {
  name = "example_role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Sid    = "examplerole"
        Principal = {
          Service = "ec2.amazonaws.com"
        }
      }
    ]
  })
}

### Attach IAM policy to IAM role
resource "aws_iam_policy_attachment" "policy_attach" {
  name       = "example_policy_attachment"
  roles      = [aws_iam_role.example_role.name]
  policy_arn = aws_iam_policy.example_policy.arn
}

### Create instance profile using role
resource "aws_iam_instance_profile" "example_profile" {
  name = "example_profile"
  role = aws_iam_role.example_role.name
}

### Create EC2 instance and attach IAM role
resource "aws_instance" "example_instance" {
  instance_type        = var.ec2_instance_type
  ami                  = var.image_id
  iam_instance_profile = aws_iam_instance_profile.example_profile.name

  # EC2 will only be created after the IAM role is created, then the role is attached
  depends_on = [aws_iam_role.example_role]
}
```

---

### count

By default a resource block creates a **single infrastructure object**. Use `count` to create multiple resources with the same configuration without duplicating the block.

- `count` requires a whole number
- Use `count.index` to uniquely identify each instance (ranges from `0` to `count-1`)
- Available in resources and modules (Terraform version 0.13+)
- **Cannot be used together with `for_each`**

#### Example 1 — Simple count

```hcl
resource "aws_instance" "myec2" {
  ami           = "ami-0230bd60aa48260c6"
  instance_type = "t2.micro"
  count         = 2

  tags = {
    Name = "webec2-${count.index}"
  }
}
```

#### Example 2 — count with a list variable

```hcl
# variables.tf
variable "ami" {
  type    = string
  default = "ami-0440d3b780d96b29d"
}

variable "instance_type" {
  type    = string
  default = "t2.micro"
}

variable "sandboxes" {
  type    = list(string)
  default = ["sandbox_server_two", "sandbox_server_three"]
}

# main.tf
resource "aws_instance" "sandbox" {
  ami           = var.ami
  instance_type = var.instance_type
  count         = length(var.sandboxes)

  tags = {
    Name = var.sandboxes[count.index]
  }
}
```

---

### for_each

`for_each` is a more flexible way to create multiple resource instances compared to `count`. It accepts a **map or set of strings** and creates one instance per member.

| Object | Description |
|---|---|
| `each.key` | The map key or set member corresponding to each instance |
| `each.value` | The map value corresponding to each instance |

> Available in resources (Terraform v0.12.6+) and modules (Terraform v0.13+)

#### Example — for_each with a set variable

```hcl
# variables.tf
variable "ami" {
  type    = string
  default = "ami-0078ef784b6fa1ba4"
}

variable "instance_type" {
  type    = string
  default = "t2.micro"
}

variable "sandboxes" {
  type    = set(string)
  default = ["sandbox_one", "sandbox_two", "sandbox_three"]
}

# main.tf
resource "aws_instance" "sandbox" {
  ami           = var.ami
  instance_type = var.instance_type
  for_each      = var.sandboxes

  tags = {
    Name = each.value # For a set, each.value and each.key are the same
  }
}
```

---

### Multi-Provider (provider alias)

The `provider` meta-argument specifies which provider to use for a resource. This is useful when using **multiple providers**, usually for multi-region resources.

- Differentiate providers using the `alias` field
- Reference the alias in resources as `provider = <provider>.<alias>`

#### Example — Multi-Region S3 Buckets

```hcl
# Provider-1 for ap-south-1 (Default Provider)
provider "aws" {
  region = "ap-south-1"
}

# Provider-2 for us-east-1 with alias
provider "aws" {
  region = "us-east-1"
  alias  = "america"
}

resource "aws_s3_bucket" "test" {
  bucket = "del-hyd-naresh-it"
}

resource "aws_s3_bucket" "test2" {
  bucket   = "del-hyd-naresh-it-test2"
  provider = aws.america
}
```

---

### lifecycle

The `lifecycle` meta-argument is a nested configuration block within a resource block. It specifies how Terraform should handle the **creation, modification, and destruction of resources**.

#### Available lifecycle attributes

| Attribute | Description |
|---|---|
| `create_before_destroy` | Creates the new object first, then destroys the old one — reduces downtime |
| `prevent_destroy` | Prevents Terraform from accidentally removing critical resources |
| `ignore_changes` | Ignores specified attribute changes that occur outside of Terraform |

#### Full Example

```hcl
resource "aws_instance" "test" {
  ami               = "ami-0440d3b780d96b29d"
  instance_type     = "t2.micro"
  availability_zone = "us-east-1b"

  tags = {
    Name = "test"
  }

  lifecycle {
    create_before_destroy = true  # Creates new object first, then destroys the old one
  }

  # lifecycle {
  #   prevent_destroy = true  # Terraform will error if it attempts to destroy this resource
  # }

  # lifecycle {
  #   ignore_changes = [tags,]  # Terraform will never update the object but can create or destroy it
  # }
}
```

#### `create_before_destroy`

```hcl
lifecycle {
  create_before_destroy = true
}
```

#### `prevent_destroy`

```hcl
lifecycle {
  prevent_destroy = true
}
```

If Terraform tries to destroy a resource with `prevent_destroy = true`, it raises this error:

```
Error: Instance cannot be destroyed
Resource [resource_name] has lifecycle.prevent_destroy set, but the plan calls
for this resource to be destroyed. To avoid this error and continue with the plan,
either disable lifecycle.prevent_destroy or reduce the scope of the plan using
the -target flag.
```

#### `ignore_changes`

Ignore a specific tag:

```hcl
lifecycle {
  ignore_changes = [
    tags["department"]
  ]
}
```

Ignore all attribute changes (Terraform can still create or destroy the object):

```hcl
lifecycle {
  ignore_changes = [all]
}
```

> The `ignore_changes` option is useful when attributes are updated outside Terraform (e.g., via an Azure Policy that automatically applies tags).

---

## Locals

A local value assigns a name to an expression so you can use the name **multiple times within a module**. They help avoid repeating the same values or expressions, but if overused they can make a configuration hard to read.

> Local values are not set by user input or values in terraform files — they are set **locally** to the configuration.

#### Example — S3 Bucket with Local Value

```hcl
locals {
  bucket-name = "${var.layer}-${var.env}-bucket-hydnaresh"
}

resource "aws_s3_bucket" "demo" {
  bucket = local.bucket-name

  tags = {
    Name        = local.bucket-name
    Environment = var.env
  }
}
```

---

## Provisioners

Terraform includes the concept of **provisioners** as a measure of pragmatism, knowing that there will always be certain behaviors that can't be directly represented in Terraform's declarative model.

Provisioners can be used to model specific actions on the **local machine** or on a **remote machine** in order to prepare servers or other infrastructure objects for service.

---

### File Provisioner

The file provisioner is used to **copy files or directories** from the machine executing `terraform apply` to the newly created resource. It can connect to the resource using either `ssh` or `winrm` connections.

```hcl
resource "aws_instance" "web" {
  # ...

  # Copies the myapp.conf file to /etc/myapp.conf
  provisioner "file" {
    source      = "conf/myapp.conf"
    destination = "/etc/myapp.conf"
  }
}
```

---

### local-exec Provisioner

The `local-exec` provisioner invokes a local executable **after a resource is created**. This invokes a process on the **machine running Terraform**, not on the resource.

- Used when you want to perform tasks on your local machine where Terraform is installed
- Never used to perform tasks on a remote machine

```hcl
resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    command = "echo ${self.private_ip} >> private_ips.txt"
  }
}
```

---

### remote-exec Provisioner

The `remote-exec` provisioner invokes a script **on a remote resource** after it is created. This can be used to run a configuration management tool, bootstrap into a cluster, etc.

- Always works on the **remote machine**
- Requires a `connection` block
- Supports both `ssh` and `winrm`

```hcl
resource "aws_instance" "web" {
  # ...

  # Establishes connection to be used by all generic remote provisioners (i.e. file/remote-exec)
  connection {
    type        = "ssh"
    user        = "ubuntu"  # Replace with the appropriate username for your EC2 instance
    # private_key = file("C:/Users/veerababu/.ssh/id_rsa")
    private_key = file("~/.ssh/id_rsa")  # private key path
    host        = self.public_ip
  }

  provisioner "remote-exec" {
    inline = [
      "touch file200",
      "echo hello from aws >> file200",
    ]
  }
}
```

> It can be used inside the Terraform resource object (invoked once the resource is created) or inside a **null resource** — the null resource approach is preferred as it separates this non-Terraform behavior from the real Terraform behavior.

---

## Connection Block

Connection blocks describe **how to access the remote resource**. One use case for providing multiple connections is to have an initial provisioner connect as the root user to set up user accounts, then have subsequent provisioners connect as a user with more limited permissions.

- Connection blocks **don't take a block label**
- Can be nested within either a **resource** or a **provisioner**
- A connection block nested directly within a resource affects **all of that resource's provisioners**

#### Example

```hcl
connection {
  type        = "ssh"
  user        = "ubuntu"  # Replace with the appropriate username for your EC2 instance
  # private_key = file("C:/Users/veerababu/.ssh/id_rsa")
  private_key = file("~/.ssh/id_rsa")  # private key path
  host        = self.public_ip
}
```

---

## Output Values

Output values make **information about your infrastructure available on the command line** and can expose information for other Terraform configurations to use.

```hcl
output "instance_public_ip" {
  value     = aws_instance.test.public_ip
  sensitive = true
}

output "instance_id" {
  value = aws_instance.test.id
}

output "instance_public_dns" {
  value = aws_instance.test.public_dns
}

output "instance_arn" {
  value = aws_instance.test.arn
}
```

---

## Variable Validation (Conditions)

You can add a `validation` block inside a variable to enforce **conditions** on the input values. If the condition fails, Terraform returns the error message.

```hcl
variable "aws_region" {
  description = "The region in which to create the infrastructure"
  type        = string
  nullable    = false
  default     = "change me"  # Must be either us-west-2 or eu-west-1; any other region will produce an error

  validation {
    condition     = var.aws_region == "us-west-2" || var.aws_region == "eu-west-1"
    error_message = "The variable 'aws_region' must be one of the following regions: us-west-2, eu-west-1"
  }
}

provider "aws" {
  region = var.aws_region
}
```

---

## Modules

### What are Terraform Modules?

Terraform modules are **reusable and encapsulated collections of Terraform configurations**. They simplify managing resources, making your Terraform code more manageable and scalable. Modules make defining, configuring, and organizing resources modular and consistent while abstracting away their complexity.

### Benefits of Modules

| Benefit | Description |
|---|---|
| **Reusability** | Organize infrastructure resources and configurations into containers you can repurpose across projects and environments. Saves effort and reduces errors. |
| **Abstraction** | Simplify resource creation and configuration processes for Terraform configuration files, making them more concise and understandable. |
| **Encapsulation** | Isolate resources and their dependencies, making it straightforward to manage or modify individual pieces of infrastructure without impacting others. |
| **Versioning** | Modules can be versioned, making it easier to track changes and update dependencies in an orderly manner. |
| **Collaboration** | Allow teams and the wider community to work collaboratively by sharing modules via Terraform Registry or private module repositories. |

---

### Example 1: AWS VPC Module

Creating an AWS VPC is fundamental for many infrastructure deployments. Instead of defining the VPC configuration repeatedly, we create a Terraform module for it.

#### Module Directory Structure

```
modules/
  vpc/
    main.tf
    variables.tf
```

#### VPC Module Code (`modules/vpc/main.tf`)

```hcl
resource "aws_vpc" "example" {
  cidr_block = var.cidr_block
  tags       = { Name = var.name }
}
```

#### Input Variables (`modules/vpc/variables.tf`)

```hcl
variable "cidr_block" {
  description = "The CIDR block for the VPC."
}

variable "name" {
  description = "The name of the VPC."
}
```

#### Using the VPC Module (`main.tf`)

```hcl
module "my_vpc" {
  source     = "./modules/vpc"
  cidr_block = "10.0.0.0/16"
  name       = "my-vpc"
}
```

> In the main Terraform configuration, the `module` block is used to include the VPC module. We specify the module's source directory and provide values for the input variables. You can now easily create multiple VPCs with different configurations by reusing this module.

---

### Example 2: AWS EC2 Instance Module

#### Module Directory Structure

```
modules/
  ec2/
    main.tf
    variables.tf
```

#### EC2 Instance Module Code (`modules/ec2/main.tf`)

```hcl
resource "aws_instance" "example" {
  ami           = var.ami
  instance_type = var.instance_type
  subnet_id     = var.subnet_id
  key_name      = var.key_name

  tags = {
    Name = var.name
  }
}
```

#### Input Variables (`modules/ec2/variables.tf`)

```hcl
variable "ami" {
  description = "The AMI ID for the EC2 instance."
}

variable "instance_type" {
  description = "The instance type for the EC2 instance."
}

variable "subnet_id" {
  description = "The subnet ID for the EC2 instance."
}

variable "key_name" {
  description = "Key pair to associate with the EC2 instance."
}

variable "name" {
  description = "The name of the EC2 instance."
}
```

#### Using the EC2 Instance Module (`main.tf`)

```hcl
module "my_ec2" {
  source        = "./modules/ec2"
  ami           = "ami-12345678"
  instance_type = "t2.micro"
  subnet_id     = "subnet-01234567"
  key_name      = "my-key-pair"
  name          = "my-ec2-instance"
}
```

> With this module, you can easily create EC2 instances with different configurations across your infrastructure. These examples demonstrate how Terraform modules promote code reuse, abstraction, and encapsulation. By following similar patterns, you can create modules for databases, load balancers, networking resources, and more.

### Module Source from GitHub

```hcl
module "example" {
  source = "github.com/CloudTechDevOps/Terraform/root_modules"
  # root_modules is the source reference folder in the GitHub repo
}
```

---

> 📝 **Notes written by:** [Shubham Kharche](https://github.com/ShubhamCloudOps)
> 🔗 **Reference:** [HashiCorp Terraform Documentation](https://developer.hashicorp.com/terraform/docs)
