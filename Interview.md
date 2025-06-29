🌱 Basic Terraform questions & answers  
#
**1️⃣ What is Terraform, and what problem does it solve?**

✅ Terraform is an Infrastructure as Code (IaC) tool that lets you define cloud and on-prem resources using code. It solves the problem of manual infrastructure provisioning, making deployments consistent, repeatable, and version-controlled.

**2️⃣ What is Infrastructure as Code (IaC)?**

✅ IaC means managing and provisioning infrastructure (VMs, networks, etc.) using declarative code rather than manual processes. You can track infra in source control, review changes, and automate deployments.

**3️⃣ Explain the difference between terraform init, plan, apply, and destroy.**

terraform init: Initializes the working directory, downloads provider plugins, and sets up the backend.

terraform plan: Creates an execution plan, showing what Terraform will change without actually changing anything.

terraform apply: Applies the changes required to reach the desired state as defined in your .tf files.

terraform destroy: Destroys the infrastructure defined in your Terraform files.

**4️⃣ What is a Terraform provider? Give examples.**  

✅ A provider is a plugin that lets Terraform interact with APIs of cloud or service platforms.
Examples:

AWS (hashicorp/aws)

Azure (hashicorp/azurerm)

Google Cloud (hashicorp/google)

VMware (hashicorp/vsphere)

**5️⃣ What is a state file? Why does Terraform need it?**  

✅ The state file (terraform.tfstate) tracks the current state of resources Terraform manages. It allows Terraform to know what’s already deployed and to calculate diffs when you make changes.

**6️⃣ Where is Terraform state stored by default?**

✅ By default, it’s stored locally as terraform.tfstate in your working directory.

**7️⃣ What are Terraform variables, and how do you define them?**

✅ Variables make Terraform configurations flexible.
Example:
```
variable "instance_type" {
  description = "EC2 instance type"
  default     = "t2.micro"
}
```

You reference it using var.instance_type.

**8️⃣ What is an output variable? When would you use it?**  

✅ Outputs let you display or pass data from a Terraform module.
Example:
```
output "instance_ip" {
  value = aws_instance.web.public_ip
}
```
Used to show values after apply, or pass between modules.

**9️⃣ What are terraform.tfvars files used for?**

✅ Files to assign values to variables automatically, so you don’t have to provide them via CLI every time.

**🔟 Explain the purpose of terraform fmt and terraform validate.**

terraform fmt: Automatically formats .tf files to canonical style.

terraform validate: Checks whether the configuration syntax is valid (without contacting cloud).

# 🌿 Intermediate Terraform questions & answers

**1️⃣ What are Terraform modules? How do they help?**  

✅ Modules are reusable blocks of Terraform code.
They help you avoid duplication and promote consistency across environments (e.g., network, vpc, eks modules).

**2️⃣ How do you reference an output from one module in another?**

✅ Using module.<module_name>.output_name.
Example:
```
module "vpc" {
  source = "./vpc"
}

resource "aws_instance" "web" {
  subnet_id = module.vpc.subnet_id
}
```

**3️⃣ What is a remote backend? Why would you use one?**

✅ A remote backend stores your Terraform state file remotely (e.g., S3, Azure Blob, Terraform Cloud).
Used for:

Collaboration (shared state)

State locking

Security

**4️⃣ How do you enable state locking in Terraform?**  

✅ Use a backend that supports locking, e.g., S3 with DynamoDB lock table, or Terraform Cloud.

**5️⃣ What is terraform import, and when would you use it?**  

✅ Imports an existing resource into Terraform state, so Terraform can manage it.
Used when you want to start managing already existing resources.

**6️⃣ What is drift detection? How do you perform it?**  

✅ Detecting when actual infrastructure differs from the Terraform state.
You run:
```
terraform plan
```
Terraform refreshes state and shows differences.

**7️⃣ Explain the purpose of lifecycle meta-arguments.**  

✅ Control resource behavior (e.g., prevent destroy, create before destroy, ignore changes).

**8️⃣ What happens when you run terraform plan -refresh=false?**  


✅ Terraform skips refreshing infra state; it compares only code to the last known state. Drift will not be detected.

**9️⃣ How does terraform taint work?**  

✅ Marks a resource for destruction and recreation during the next apply. Used when resource needs to be force-replaced.

**🔟 Explain count vs for_each.**  

Feature	count	for_each
Type	Integer index	Map or set of strings
Identifiers	count.index	each.key, each.value
Use case	Simple replication	Resource per unique key/value

# 🌳 Advanced Terraform questions & answers  

**1️⃣ Explain how ignore_changes works.**  

✅ Tells Terraform to ignore certain attributes if they change outside Terraform (e.g., tags)., If some attribute is set to ignore and you update its value in your .tf file then terraform will apply this change when you run terraform apply command.

But if someone go to console and changes this attribute manually and then if you run plan?  

Terraform will ignore this drift and not try to revert it.  

plan will show "no changes". 


**2️⃣ How does create_before_destroy affect behavior?**  

✅ Creates replacement resource first, then destroys old resource. Avoids downtime but may temporarily use extra resources.

**3️⃣ What is replace_triggered_by?**  


✅ Forces a resource to be replaced if another resource or value changes. Useful for tightly coupled dependencies.

**4️⃣ How do you manage secrets securely?**  

✅ Use external secret managers (e.g., Vault, AWS SSM). Avoid hardcoding sensitive data in .tf or state files.

**5️⃣ How can you integrate Vault to fetch secrets?**  

✅ Use vault provider and vault_kv_secret_v2 data block. Export VAULT_TOKEN securely.
Example:
```
data "vault_kv_secret_v2" "db_creds" {
  name  = "database/creds"
  mount = "secret"
}
```

**6️⃣ Explain Terraform workspaces.**

✅ Allow multiple state files under one configuration. Commonly used for managing multiple environments (e.g., dev, prod).

**7️⃣ How do you structure code for multi-environment deployments?**  

✅ Use separate workspaces, directories, or modules. Store environment-specific variables in separate files (dev.tfvars, prod.tfvars).

**8️⃣ What is terraform state command for?**  

✅ Manipulate state file directly (move, remove, list resources).
Example:
```
terraform state list
terraform state rm aws_instance.old
terraform state mv aws_instance.old aws_instance.new
```

**9️⃣ How to handle corrupt/inconsistent state?**  

✅ Options:

Restore from backup

Use terraform state commands to repair

In worst case: manually edit state file (dangerous!)

**🔟 How do you use dynamic blocks?**  

✅ Used to generate nested blocks dynamically, especially when attributes are conditionally repeated.

Example:
```
dynamic "ingress" {
  for_each = var.ingress_rules
  content {
    from_port = ingress.value.from
    to_port   = ingress.value.to
    protocol  = "tcp"
    cidr_blocks = ingress.value.cidr_blocks
  }
}
```

# 🌟 Bonus scenario questions & answers  

**1️⃣ You have manual infra. How to start managing with Terraform?**  

✅ Write resources in .tf files.
Use terraform import to import each resource into state.
Run plan to align code with reality.

**2️⃣ Rollback failed apply?**  

✅ If changes applied partially:

Review plan

Revert .tf files to previous state

Re-run apply

Use version control (git) to roll back code

**3️⃣ Unauthorized changes to prod — what to do?**  

✅ Run terraform plan to detect drift.
Revert unauthorized changes by re-applying correct .tf code.
Consider restricting manual console changes and enabling audit logging.

**4️⃣ Minimize downtime when updating prod resources?**  

✅ Use create_before_destroy.
Use rolling updates (if supported).
Use blue-green or canary deployments.

**5️⃣ Multi-region code reuse?**  

✅ Use modules for common infra logic.
Pass region and other inputs as variables.
Optionally separate environment-specific configurations into different directories.


##
##
Are terraform.tfvars and variables the same?

✅ What are variables?
In Terraform, you define variables using a variable block, for example:

```
variable "instance_type" {
  description = "EC2 instance type"
  default     = "t2.micro"
}
```

This declares a variable.

You can optionally set a default value.

✅ What is terraform.tfvars?
terraform.tfvars (or *.tfvars files) are variable value files.  

They provide values to your defined variables.  

You do not define variables here — only assign them.  

Example terraform.tfvars
```
instance_type = "t3.medium"
region        = "us-east-1"
```
✅ How does Terraform consume these?
Terraform automatically loads:

A file named terraform.tfvars

Any files ending in .auto.tfvars

Other .tfvars files (with custom names) must be loaded manually using:

```
terraform apply -var-file="myvars.tfvars"
```

💡 Priority order of variable values
When Terraform evaluates variable values, it uses the following priority order (from highest to lowest):

1️⃣ Command-line flags

```
terraform apply -var="instance_type=t3.large"
```
2️⃣ Environment variables

```
export TF_VAR_instance_type="t3.large"
```
3️⃣ terraform.tfvars file (automatically loaded)  

4️⃣ Any *.auto.tfvars files (automatically loaded)  

5️⃣ Default value in variable block  

✅ Example flow
```
# variables.tf
variable "instance_type" {
  default = "t2.micro"
}
```
```
# terraform.tfvars
instance_type = "t3.medium"

```
If you run as is
Terraform picks t3.medium from terraform.tfvars.

If you run with CLI override
```
terraform apply -var="instance_type=t3.large"
```
Terraform uses t3.large, ignoring tfvars.


#
#

💡 count vs for_each in Terraform  
✅ count  
Works with integers.  

Good for creating N identical resources.  

Indexing via count.index.  

🔹 Example using count
```
resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  tags = {
    Name = "web-${count.index}"
  }
}
```
Result
Creates 3 instances.

Names: web-0, web-1, web-2.
#
✅ for_each
Works with maps or sets of strings (can also work with lists converted to sets).  

Good for creating different resources based on keys/values.  

Indexing via each.key and each.value.  

🔹 Example using for_each with a map
```
variable "instances" {
  type = map(string)
  default = {
    "app1" = "t2.micro"
    "app2" = "t2.small"
    "app3" = "t3.micro"
  }
}

resource "aws_instance" "web" {
  for_each      = var.instances
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = each.value
  tags = {
    Name = each.key
  }
}
```

Result
Creates 3 instances, each with a different name and type:  

app1 → t2.micro  

app2 → t2.small  

app3 → t3.micro  

✅ When to use which?
Use case	Prefer
Simple, identical N copies	count
Unique names or configs	for_each

#
#
💡 What is a dynamic block?  
✅ In Terraform, a dynamic block is used when you need to create repeated nested blocks inside a resource dynamically, based on a variable or list.  

Instead of manually repeating the block multiple times, dynamic lets you generate them programmatically.

⚙️ Common example: security group ingress rules
🟢 Without dynamic block
```
resource "aws_security_group" "example" {
  name = "example-sg"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```
🔵 With dynamic block
```
variable "ingress_rules" {
  type = list(object({
    from_port   = number
    to_port     = number
    protocol    = string
    cidr_blocks = list(string)
  }))
  default = [
    {
      from_port   = 22
      to_port     = 22
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    },
    {
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  ]
}

resource "aws_security_group" "example" {
  name = "example-sg"

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}
```

✅ How it works  
Keyword	Meaning
dynamic	Start a dynamic block
for_each	Loop over list or map
content {}	Defines what goes inside each generated block

🔥 Additional example: Azure tags
```
variable "tags" {
  type = map(string)
  default = {
    "Environment" = "dev"
    "Owner"      = "teamA"
  }
}

resource "azurerm_resource_group" "example" {
  name     = "example-rg"
  location = "East US"

  tags = var.tags
}
```
You don't need a dynamic block for flat tags in Azure because tags already accept a map — but if you had nested blocks (like NSG security rules), you'd use dynamic.  
