# State file management in Azure:
#

**✅ Create State File for Azure (with Azure Storage Account)**
Create Storage Account & Container
You need:

Resource Group

Storage Account

Blob Container

```
az group create --name tfstate-rg --location eastus

az storage account create \
  --name tfstatestorage123 \
  --resource-group tfstate-rg \
  --location eastus \
  --sku Standard_LRS

az storage container create \
  --name tfstate \
  --account-name tfstatestorage123
```

**Configure Backend in Terraform**
```
terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstatestorage123"
    container_name       = "tfstate"
    key                  = "terraform.tfstate"
  }
}
```

Run:

**terraform init**

🔒 State Locking
🔐 What Is It?
Terraform uses state locking to prevent:

Simultaneous operations (which can corrupt state)

Ensures that only one apply/plan happens at a time

**🔒 Locking Mechanisms**

S3	DynamoDB table (LockID)
AzureRM	Azure Blob lease
GCS	GCS object generation locking

----------------------------------------------------------------
#
#
**How locking works in Azure**

#

Terraform automatically:

Acquires a Blob lease on the file (your terraform.tfstate) before reading/updating it

Releases the lease after completion

Prevents others from acquiring it while in use

#

**How locking works in AWS**

DynamoDB is Amazon’s fully managed NoSQL database service. It’s:

Highly available

Scalable

Key-value and document based

In Terraform, it’s not used as a database for your infra — instead, it’s used to manage state locking.
#

**🔐 How DynamoDB Is Used for Locking**
#
🔁 Terraform + S3 + DynamoDB = Safe Remote State
When using the s3 backend with Terraform, you can configure DynamoDB for state locking.

**✅ How It Works (Step-by-Step)**
#
When you run terraform apply, Terraform tries to acquire a lock.

It writes a row in your DynamoDB table with:

A LockID, A timestamp and operation info like Apply, Plan, etc.

If someone else tries to run a command:

Terraform sees the lock already exists

It throws an error like:

```
Error acquiring the state lock
```

Once the operation is done:

The lock entry is deleted (unlocked)
