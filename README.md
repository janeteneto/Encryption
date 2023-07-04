# Harshicorp Vault - Transit Secrets Engine

## How to safely secure secrets that are currently stored in a `tfstate` file in S3

There is more than one way to use Harshicorp Vault to secure secrets, passwords, several platform credentials like AWS, K8 and Aure, and even ways to encrypt and safely secure files. Today I will explore ways to secure files, specifically a default file created when you initialise Terraform, called **`tfstate` file**. 

The first feature is **Transit Secrets Engine**- This Hashicorp feature allows you to encrypt data **in transit and at rest**:

![image](https://github.com/janeteneto/Harshicorp-Vault/assets/129942042/8da9f268-accd-4340-bcf4-59818f7a4fd9)

**First of all, I will create a bucket on AWS with Dual-layer server-side encryption with AWS Key Management Service keys (DSSE-KMS). This will give the objects in the bucket two separate layers of encryption:**

1. Search S3 on AWS
2. Press `Create Bucket`
3. Select the following configurations:

![2023-07-04 (2)](https://github.com/janeteneto/Harshicorp-Vault/assets/129942042/e5674ff5-a9be-4de7-b627-a1d4d1afa54e)
![2023-07-04 (3)](https://github.com/janeteneto/Harshicorp-Vault/assets/129942042/7c2dd586-0ffa-4daa-9e61-152990d8189d)

4. If you don't have KMS keys already, press `Create KMS Keys`. If you do, jump to step 9:

![2023-07-04 (4)](https://github.com/janeteneto/Harshicorp-Vault/assets/129942042/794a24a7-67d4-4002-b96d-ac6331858e5c)

5. Let's create and configure the KMS keys with the following configurations:

![2023-07-04 (5)](https://github.com/janeteneto/Harshicorp-Vault/assets/129942042/f711022f-8c9a-4a60-a80b-39368f97b0c3)

- I chose the symmetric key type because no one needs to access this specific bucket on S3 except the owner itself, which means that there is no need for a public key as this doesn't need to be shared with anyone.

6. Then press `Next` and add an alias named `test`.

7. On steps 3 and 4 of the creation of keys, you can specify relevant policies according to the company's guidelines to provide access or deny access to specific IAM users on your AWS root account. I will leave the policy to allow all, as I am the only person using the account.

![2023-07-04 (6)](https://github.com/janeteneto/Harshicorp-Vault/assets/129942042/f6a5f964-2bf1-498e-8a16-117d5c45f8c8)

8. Review and press `Finish`.

9. After the key is created, go back to the S3 bucket configuration, select the key you just created, and press `Create bucket`.

10. 
### Steps:
- I will base this tutorial using the CLI instead of the web UI

1. Install vault with the command `choco install vault` on a terminal as admin

2. Then command:
````
vault secrets enable transit
````

6. Create an encryption key ring named `test` by executing the following command:
````
vault write -f transit/keys/test
````


