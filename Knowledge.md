# 💡 What is "resource lifecycle" in Terraform?

A resource lifecycle defines how Terraform manages resources throughout their life — from creation to updates and deletion.

Terraform offers lifecycle customization through a lifecycle block inside a resource.

⚙️ Example structure
```
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  lifecycle {
    create_before_destroy = true
    prevent_destroy       = true
    ignore_changes        = [tags, user_data]
  }
}
```
💡 Key lifecycle arguments  
🟢 1️⃣ create_before_destroy  
By default, when you make a change that requires replacement (e.g., changing an immutable property), Terraform destroys the old resource first, then creates a new one.  

✅ With create_before_destroy = true, Terraform creates the new resource first, then destroys the old one.  

💥 When to use?
To minimize downtime.

For resources where temporary duplication is okay (e.g., load balancers, VMs with ephemeral data).  

🔴 2️⃣ prevent_destroy  
If set to true, Terraform refuses to destroy this resource, even if a plan would normally remove it.  

💥 When to use?
Critical resources (e.g., production database, S3 bucket with important data).

Avoid accidental deletions.

Example error
```
Error: Instance cannot be destroyed

Resource aws_db_instance.production has lifecycle.prevent_destroy set, but the plan calls for this resource to be destroyed.
```

🟡 3️⃣ ignore_changes
Tell Terraform to ignore certain attribute changes made outside Terraform (e.g., manually in console).

Example
```
lifecycle {
  ignore_changes = [tags["Owner"], user_data]
}
```
💥 When to use?
When you allow certain manual edits (like changing tags).

When external tools or operators update a specific field.  

🔵 4️⃣ replace_triggered_by (Terraform 0.15+)
Force resource replacement when another resource or attribute changes.

Example
```
resource "aws_instance" "example" {
  ami           = "ami-123456"
  instance_type = "t3.micro"

  lifecycle {
    replace_triggered_by = [
      aws_ami.latest.id
    ]
  }
}
```
💥 When to use?
When dependent resources change (e.g., new AMI), and you want the instance to always recreate.

⚙️ Resource lifecycle phases (big picture)  
🟢 Create  
Terraform provisions resource as described in .tf.  

State file is updated with IDs and attributes.  

🟡 Update  
Changes detected via plan.  

Terraform updates resource in place if possible.  

If not possible (some changes require replacement), it triggers destroy/create logic.  

🔴 Destroy  
Terraform removes resource from infrastructure.  

State file updated to forget the resource.  

✅ Example practical scenario
```
resource "aws_s3_bucket" "logs" {
  bucket = "my-logs-bucket"

  lifecycle {
    prevent_destroy = true
  }
}
```
✅ What happens?
Bucket cannot be destroyed by mistake.

If you run terraform destroy, it will error on this bucket.

✅ ✅ Summary table
Argument	Purpose	Common use cases  
create_before_destroy	Avoid downtime	Load balancers, VMs needing zero downtime  
prevent_destroy	Prevent accidental deletion	Prod DBs, important buckets  
ignore_changes	Ignore manual or external changes	Tags, user data  
replace_triggered_by	Force recreate on dependency change	AMI or cert changes
