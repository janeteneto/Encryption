# Encrypting `tfstate` with SOPS Step-by-step
- We will go through the steps of encrypting a `tfstate` file and storing it in **S3** for additional protection.

## Setting up AWS S3 and CLI
1. Make sure to have an AWS account and an IAM user with the required permissions (S3 full access, KMS full access, etc).
2. Log into your IAM user

3. Go to S3 Dashboard on AWS and create a bucket with DSSE-KMS. Find how [here](https://github.com/janeteneto/Encryption/blob/main/Hashicorp%20Vault.md)

4. Open a bash terminal on VSCode and create a new directory to start this project. I named it `SOPS`

5. Make sure to have AWS CLI downloaded or download it with this [step-by-step guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

6. Configure your AWS credentials on the terminal

**- Now we are ready to use SOPS and Terraform**

## Setting up SOPS

1. Download SOPS with the command `choco install sops`
2. Create a new file on the current directory called `sops.yaml`, where it should look like this:
````
creation_rules:
  - kms: arn:aws:kms:us-east-1:358271186147:key/0f7045e8-719b-4aa7-a4e4-fdee49e21321
````
- replace with your own KMS key's arn

3. For the encryption of the file, I will create a script that will allow faster encryption and upload of the file to the bucket. The script file is called `script.sh` and looks like this:
````
#!/bin/bash
sops --encrypt --kms arn:aws:kms:us-east-1:358271186147:key/0f7045e8-719b-4aa7-a4e4-fdee49e21321 .terraform/terraform.tfstate > encrypted.tfstate
aws s3 cp encrypted.tfstate s3://janetetest/terraform.tfstate
````
- The first command uses our KMS key to encrypt the file
- The second command copies the encrypted file's content into the terraform.tfstate, located in our S3 bucket. **This way, whenever we run this script, the original content of the tfstate will be encrypted and be placed in a file called `encrypted.tfstate`, then with the copy command, it uploads the encrypted content and replaces the original tfstate file's content in the S3 bucket**

## Setting and initialising Terraform backend

1. On the same directory, create a file named `main.tf`. We will create this file to configure our infrastructure and enable us to use the s3 bucket as a backend. The file should look like this:
````
provider "aws" {
    region = "us-east-1"
}

terraform {
    backend "s3" {
        bucket = "janetetest"
        key = "terraform.tfstate"
        region = "us-east-1"
    }
}
````

2. On the terminal, run `chmod +x script.sh` to make the file executable
3. Then `terraform init`
4. Then `terraform apply && ./script.sh`

- The file is encrypted and uploaded to S3!

## Decrypt tfstate and re-initialise Terraform

1. Download the file from the S3 bucket

2. Delete the state file on the S3 bucket

- Every time we run `terraform init`, Terraform will go to the backend and fetch the tfstate file from there, which is the encrypted file we uploaded to S3. In order to initialise based on the latest state file, we need to **delete the encrypted one on the s3 bucket after downloading it**. This way, Terraform will not use the file on the S3, and we can specify which state file we want to be used.

3. Make sure to be in the correct directory on the terminal, then run the command:
````
sops -d encrypted.tfstate > decrypted.tfstate
````

4. Run `terraform init` on your machine, if you haven't done so before

5. Step 4 created a brand new tfstate file, but we want to apply the previous one that has been decrypted. To do that, we need this command:
````
terraform apply -state=decrypted.tfstate && ./script.sh
````

- This way, Terraform will use the content in the decrypted state file.
