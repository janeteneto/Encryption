# Harshicorp Vault

## How to safely secure secrets that are currently stored in a `tfstate` file

There is more than one way to use Harshicorp Vault to secure secrets, passwords, several platform credentials like AWS, K8 and Aure, and even ways to encrypt and safely secure files. Today I will explore ways to secure files, specifically a default file created when you initialise Terraform, called **`tfstate` file**. 

The first feature is **Transit Secrets Engine**:

- This Hashicorp feature allows you to encrypt data **in transit and at rest**.

![image](https://github.com/janeteneto/Harshicorp-Vault/assets/129942042/8da9f268-accd-4340-bcf4-59818f7a4fd9)

### Steps:
- I will base this tutorial using the CLI instead of the web UI

1. Install vault with the command `choco install vault` on a terminal as admin

2. Then command:
````
vault server -dev -dev-root-token-id root
````
- DO NOT RUN COMAND IN PRODUCTION!

3. Export an environment variable for the vault CLI to address the Vault server.
````
export VAULT_ADDR="http://127.0.0.1:8200"
````

4. Export an environment variable for the vault CLI to authenticate with the Vault server.
````
export VAULT_TOKEN=root
````

5. **Enable the transit secrets engine** by executing the following command:
````
vault secrets enable transit
````

6. Create an encryption key ring named `test` by executing the following command:
````
vault write -f transit/keys/test
````


