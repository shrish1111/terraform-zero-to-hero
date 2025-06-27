# Knowledge Topics:

In Terraform, you can use the output (attribute) of one resource as the input to another resource by directly referencing it using the resource_type.resource_name.attribute syntax.



🧩 Example: Using Output of One Resource as Input to Another
Let’s say:

You create a security group, and

Then you launch an EC2 instance using the ID of that security group.

✅ Full Working Example

📄 main.tf
provider "aws" {
  region = "us-east-1"
}



**Create a Security Group**
resource "aws_security_group" "web_sg" {
  name        = "web_sg"
  description = "Allow HTTP and SSH"
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}



**Launch EC2 using the SG's ID**
resource "aws_instance" "web_server" {
  ami                    = "ami-0c55b159cbfafe1f0" # Replace with valid AMI for your region
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.web_sg.id]

  tags = {
    Name = "WebServerWithSG"
  }
}



🔍 How It Works
**aws_security_group.web_sg.id is evaluated by Terraform at runtime.**

**It dynamically fetches the ID of the created security group and passes it into the aws_instance.**

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id  # Using VPC's ID as input
  cidr_block = "10.0.1.0/24"
}



🏗 Bonus: Use Resource Attribute as Module Input

module "my_module" {
  source  = "./modules/example"
  vpc_id  = aws_vpc.main.id



---------------------------------------------------------------------------------------------------------------


**If you want to use the length in a resource (e.g., for naming, count, tags, etc.), just reference it directly:**

**📄 Example: Using List Length in a Tag**

variable "my_list" {
  type    = list(string)
  default = ["apple", "banana", "cherry"]
}

resource "aws_s3_bucket" "example" {
  bucket = "my-bucket-${length(var.my_list)}"

  tags = {
    ItemCount = tostring(length(var.my_list))
  }
}
length(var.my_list) returns 3

Resulting bucket name will be: my-bucket-3

The tag ItemCount = "3" will be added



**✅ Option 2: Assign to a Local Variable (Cleaner)
If you need to use the length in multiple places, define it in a locals block:**

locals {
  list_size = length(var.my_list)
}

resource "aws_s3_bucket" "example" {
  bucket = "my-bucket-${local.list_size}"

  tags = {
    Size = tostring(local.list_size)
  }
}



**🚫 What You Don't Need to Do:
You don’t need to use an output block like this just to use it within the same config:**


output "list_length" {
  value = length(var.my_list)
}
That’s only useful for displaying or passing between modules — not for use within the same .tf code block.



**🧠 Bonus: Use in a count Meta-Argument**

resource "aws_s3_bucket" "buckets" {
  count  = length(var.my_list)
  bucket = "item-${count.index}-${var.my_list[count.index]}"
}
This would create 3 buckets:

item-0-apple
item-1-banana
item-2-cherry
