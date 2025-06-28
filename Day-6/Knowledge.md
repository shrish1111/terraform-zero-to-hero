# 💡 What are Terraform workspaces?
#
By default, Terraform has a single workspace called default.

When you create additional workspaces, each one has its own separate state file, so you can use the same code for different environments (like dev, stage, prod).

**✅ Why use workspaces?**
#
Separate state for different environments

Avoid maintaining multiple copies of the same code

Easy to switch context

**⚙️ Common workspace commands**
🔹 List all workspaces
```
terraform workspace list
```
🔹 Create a new workspace
```
terraform workspace new dev
```
🔹 Select (switch to) a workspace
```
terraform workspace select dev
```
🔹 Show current workspace
```
terraform workspace show
```
🔹 Delete a workspace
```
terraform workspace delete dev
```
#
🛠️ How does it change state?
When you create a new workspace, Terraform creates a new state file automatically.

For example, if using an S3 backend:

```
key = "envs/terraform.tfstate"
```
Terraform actually changes the key internally:
#
envs/terraform.tfstate          ← for default workspace
envs/dev/terraform.tfstate      ← for workspace 'dev'
envs/prod/terraform.tfstate     ← for workspace 'prod'
#
💻 Example in code
```
resource "aws_s3_bucket" "example" {
  bucket = "my-bucket-${terraform.workspace}"
}
```
What happens?
In dev workspace → bucket name: my-bucket-dev

In prod workspace → bucket name: my-bucket-prod

🟢 Using terraform.workspace
Terraform provides a built-in variable:
```
terraform.workspace
```
You can use it for:

✅ Naming resources differently per workspace
✅ Choosing different configuration values

⚠️ When not to use workspaces
Workspaces are not meant to replace proper separate environments when you need completely different configurations, separate pipelines, or strict security separation.

In those cases, better to use separate directories or separate repos, each with its own backend.

#


