💡 What is Vault?
HashiCorp Vault is a tool for securely storing and accessing secrets, such as:

API keys

Database passwords

Cloud provider credentials

Certificates
#

**⚙️ How to use Vault with Terraform**  
**✅ Overview steps**  
**1️⃣ Store secret in Vault**  
**2️⃣ Configure Terraform to authenticate to Vault**  
**3️⃣ Use vault provider to read the secret**  
**4️⃣ Pass that value into your other Terraform resources**  

🟢 Step 1: Store secret in Vault
On your Vault server (CLI or UI), run:
```
vault kv put secret/myapp/config db_password="supersecret123" api_key="abcd1234"
```

This creates a KV (Key-Value) secret at secret/myapp/config.
#
🔵 Step 2: Configure Terraform Vault provider

In main.tf
```
provider "vault" {
  address = "https://vault.mycompany.com"
  #token = "...."  # Optionally set here or via VAULT_TOKEN env variable
}
```
#
🔵 Step 3: Read secret from Vault
```
data "vault_kv_secret_v2" "myapp" {
  mount = "secret"
  name  = "myapp/config"
}
```

After this, you can access:


data.vault_kv_secret_v2.myapp.data["db_password"]
data.vault_kv_secret_v2.myapp.data["api_key"]
#
🟢 Step 4: Use these secrets in resources
```
resource "aws_db_instance" "example" {
  allocated_storage    = 20
  engine               = "mysql"
  instance_class       = "db.t2.micro"
  username             = "admin"
  password             = data.vault_kv_secret_v2.myapp.data["db_password"]
  skip_final_snapshot  = true
}
```
💥 How does Terraform authenticate to Vault?
✅ Usually via VAULT_TOKEN
```
export VAULT_ADDR="https://vault.mycompany.com"
export VAULT_TOKEN="s.xxxxxxxx"
```
Alternatively, if you're using GitLab CI or other CI/CD, store VAULT_TOKEN as a secret variable.

✅ Or via AppRole, AWS IAM, Azure MSI
Vault supports more advanced auth methods:

AppRole (for machines)

AWS IAM or STS

Azure MSI (managed identity)

Then you configure the provider:
```
provider "vault" {
  address = "https://vault.mycompany.com"
  auth_login {
    path = "auth/approle/login"
    parameters = {
      role_id   = var.vault_role_id
      secret_id = var.vault_secret_id
    }
  }
}
```
#
**⚠️ Best Practices**  
**✅ Do not hardcode Vault tokens in your Terraform code — use environment variables or CI/CD secrets.**  
**✅ Use dynamic secrets if possible (e.g., database creds that automatically rotate).**  
**✅ Limit access to Vault policies strictly (only allow reading exactly what you need).**  
**✅ Never store Terraform state file without encryption, because resolved secrets get written into the state.**  
  
💬 Summary Table
Step	What you do
1. Store	Put secret in Vault KV
2. Configure	Add Vault provider in Terraform
3. Fetch	Use data "vault_kv_secret_v2" block
4. Use	Reference secret data in resources

🌟 Example file snippet (summary)
```
provider "vault" {
  address = "https://vault.mycompany.com"
}

data "vault_kv_secret_v2" "example" {
  mount = "secret"
  name  = "myapp/config"
}

resource "some_resource" "example" {
  password = data.vault_kv_secret_v2.example.data["db_password"]
}
```
