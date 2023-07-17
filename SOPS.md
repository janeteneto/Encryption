# Encrypting `tfstate` with SOPS Step-by-step
- We will go through the steps of encrypting a `tfstate` file and storing it in **S3** for additional protection.

## Setting up AWS S3
1. Make sure to have an AWS account and an IAM user with the required permissions (S3 full access, KMS full access, etc).
2. Log into your IAM user
3. Go to S3 Dashboard on AWS and create a bucket with DSSE-KMS. Find how [here](https://github.com/janeteneto/Encryption/blob/main/Hashicorp%20Vault.md)
4. Open a bash terminal on VSCode and create a new directory to start this project. I named it `SOPS`
5. Make sure to have AWS CLI downloaded or download it with this [step-by-step guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
