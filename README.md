# Harshicorp Vault

## How to safely secure secrets that are currently stored in a `tfstate` file

There is more than one way to use Harshicorp Vault to secure secrets, passwords, several platform credentials like AWS, K8 and Aure, and even ways to encrypt and safely secure files. Today I will explore ways to secure files, specifically a default file created when you initialise Terraform, called **`tfstate` file**. 

The first feature is **Transit Secrets Engine**:

- This Hashicorp feature allows you to encrypt and decrypt data
