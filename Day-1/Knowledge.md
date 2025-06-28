# 💡 How does Terraform connect to cloud?
🟢 For AWS
✅ The most common methods
1️⃣ Environment variables

```
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="us-east-1"
```

Terraform automatically picks these up — most recommended for CI/CD (including GitLab).

2️⃣ Shared credentials file (like using aws configure)

When you run:

aws configure
It writes to:

~/.aws/credentials
Example:

```
[default]
aws_access_key_id = YOURKEY
aws_secret_access_key = YOURSECRET
```
Terraform can use this file if no explicit environment variables are set.

3️⃣ EC2 or ECS instance roles (for runners running in AWS)

If your runner is on an EC2 instance with an IAM role attached, Terraform can automatically use that role (metadata service).

🟠 For Azure
✅ The most common method
1️⃣ Using az login (local testing)

When you run:

az login
This saves your authentication token in:

~/.azure
Terraform (using the AzureRM provider) picks it up by default.

2️⃣ Service Principal (recommended for CI/CD)

In production or CI/CD, you almost always use a Service Principal.

You set these environment variables:
```
export ARM_CLIENT_ID="app-id"
export ARM_CLIENT_SECRET="app-secret"
export ARM_SUBSCRIPTION_ID="subscription-id"
export ARM_TENANT_ID="tenant-id"
```
Terraform then connects using these and does not rely on az login.

💥 Why not use az login or aws configure in CI?
In local development, az login or aws configure is fine.

But in CI/CD (like GitLab CI Docker runner):

Each job runs in a new, clean container, no local profiles or tokens persist.

az login requires interactive browser flow (bad for automation).

🟢 Recommended approach in GitLab CI with Docker runner
✅ AWS example
In .gitlab-ci.yml

```
variables:
  AWS_ACCESS_KEY_ID: "xxxxxx"
  AWS_SECRET_ACCESS_KEY: "xxxxxx"
  AWS_DEFAULT_REGION: "us-east-1"

before_script:
  - terraform init
  - terraform workspace select dev || terraform workspace new dev
```
#
```
✅ Azure example
In .gitlab-ci.yml

yaml
Copy
Edit
variables:
  ARM_CLIENT_ID: "xxxx"
  ARM_CLIENT_SECRET: "xxxx"
  ARM_SUBSCRIPTION_ID: "xxxx"
  ARM_TENANT_ID: "xxxx"

before_script:
  - terraform init
  - terraform workspace select dev || terraform workspace new dev
```

