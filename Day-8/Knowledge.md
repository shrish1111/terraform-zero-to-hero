# Migrating existing infra into Terraform state

When you already have infrastructure in place (e.g., VM, load balancer, database), Terraform does not know about it by default.

To "tell" Terraform about these existing resources, you use terraform import.

⚙️ How terraform import works
✅ Step-by-step
1️⃣ Write Terraform configuration

Write a .tf file describing what the resource should look like, but do not apply it yet.

Example
You already have an AWS EC2 instance with ID i-0abcd1234ef567890.
Write:

```
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0" # placeholder or actual
  instance_type = "t2.micro"
}
```
2️⃣ Run import command

```
terraform import aws_instance.example i-0abcd1234ef567890
```
aws_instance.example = resource address from .tf file.

i-0abcd1234ef567890 = actual resource ID in AWS.

3️⃣ Run terraform plan

After import, run:

```
terraform plan
```
Terraform compares what’s in your .tf code vs what it read from the real infra and your state file.

4️⃣ Adjust code to match reality

After import, your .tf file might not perfectly reflect all actual settings.

Terraform will show differences

You update .tf to match or intentionally change if needed

✅ Repeat for each resource
You can import any supported resource, one at a time.

💡 Where does it get stored?
After import, Terraform writes the resource info into your state file, so Terraform now tracks it.

🟢 Additional tip: Automating imports
There are community tools (like Terraformer) that can auto-generate .tf code and state imports for you, especially useful for big infrastructures.

# 💡 Part 2: Drift detection

✅ What is drift?
"Drift" means your real infra is different from your Terraform code/state, for example:

Someone manually changed a setting in the console

Tag added/removed outside Terraform

VM size modified

✅ How to detect drift
Simply run:
```
terraform plan
```
Terraform:

Refreshes state from the actual infra

Compares with your code

Shows "planned changes" if there is a drift

✅ What if you want to only check, not change?
```
terraform plan -detailed-exitcode
```
Exit codes:

0: No changes (no drift)  

2: Changes present (drift detected)  

1: Error  

💡 You can integrate this into CI pipelines to automatically fail if drift is detected.

🛡️ Best practice
Avoid manual infra changes after migrating to Terraform

Periodically run drift detection in CI (scheduled pipelines or cron)

#

🔎 What does “refresh” mean in terraform plan?
When you run:
```
terraform plan
```
#
Terraform:  

1️⃣ Reads your current state file (local or remote).  
2️⃣ Queries the real infrastructure (cloud APIs) to check actual resource states.  
3️⃣ Updates an in-memory copy of the state with these real values.  
4️⃣ Compares this refreshed in-memory state against your .tf configuration.  
5️⃣ Shows a proposed execution plan of what it would do to make real infra match your code.  

💡 Important: The state file on disk or in the backend is not modified during plan.
After plan, your state file is exactly the same as before.  

The "refreshed state" lives only in memory for that command.
