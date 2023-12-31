# Harshicorp Vault - Transit Secrets Engine

## How to safely encrypt a file before storing in S3

There is more than one way to use Harshicorp Vault to secure secrets, passwords, several platform credentials like AWS, K8 and Azure, and even ways to encrypt and safely secure files. Today I will explore ways to secure files using Vault's feature **Transit Secrets Engine** - this Hashicorp feature allows you to encrypt data/files **in transit and at rest**, and at **client-side**.

![image](https://github.com/janeteneto/Harshicorp-Vault/assets/129942042/8da9f268-accd-4340-bcf4-59818f7a4fd9)

### Create an S3 bucket and enable DSSE-KMS Encryption

**First, I will create a bucket on AWS with Dual-layer server-side encryption with AWS Key Management Service keys (DSSE-KMS). This will give the objects in the bucket two separate layers of encryption on server-side:**

1. Search S3 on AWS
2. Press `Create Bucket`
3. Select the following configurations:

![2023-07-04 (2)](https://github.com/janeteneto/Harshicorp-Vault/assets/129942042/e5674ff5-a9be-4de7-b627-a1d4d1afa54e)
![2023-07-04 (3)](https://github.com/janeteneto/Harshicorp-Vault/assets/129942042/7c2dd586-0ffa-4daa-9e61-152990d8189d)

4. If you don't have KMS keys already, press `Create KMS Keys`. If you do, jump to step 9:

![2023-07-04 (4)](https://github.com/janeteneto/Harshicorp-Vault/assets/129942042/794a24a7-67d4-4002-b96d-ac6331858e5c)

5. Let's create and configure the KMS keys with the following configurations:

![2023-07-04 (5)](https://github.com/janeteneto/Harshicorp-Vault/assets/129942042/f711022f-8c9a-4a60-a80b-39368f97b0c3)

- I chose the symmetric key type because no one needs to access this specific bucket on S3 except the owner itself, which means there is no need for a public key as this doesn't need to be shared with anyone.

6. Then press `Next` and add an alias named `test`.

7. On steps 3 and 4 of creating keys, you can specify relevant policies according to the company's guidelines to provide access or deny access to specific IAM users on your AWS root account. I will leave the policy to allow all, as I am the only person using the account.

![2023-07-04 (6)](https://github.com/janeteneto/Harshicorp-Vault/assets/129942042/f6a5f964-2bf1-498e-8a16-117d5c45f8c8)

8. Review and press `Finish`.

9. After creating the key, return to the S3 bucket configuration, select the key you just created, and press `Create bucket`.

- Only the root account can get access when you upload a file to this bucket, but this will go according to the policies you specify. The objects in the bucket can only be retrieved by authorised users, and they are encrypted, which means even authorised users will need the key to decrypt the object.

### Encrypt file locally (client-side) using Vault Transit Secrets Engine

#### Setup Vault
- I will base this tutorial using the CLI instead of the web UI

1. Install vault with the command `choco install vault` on a terminal as admin

2. Then command:
````
vault server -dev
````

3. Copy the IP address and paste it into the browser
4. Copy the root token from the terminal and paste it on the field - only one token is needed for accessing the same vault server
5. Open a new PowerShell terminal **as admin** and run the commands:
````
 $env:VAULT_ADDR="http://127.0.0.1:8200"
````
**Or**, if you are setting up Vault on a Linux environment, run the following commands to set the environment variable:
````
export VAULT_ADDR="http://127.0.0.1:8200"
````

- The vault is ready to be used!

#### Enable Transit Secrets Engine and encrypt a file

1. On the same PowerShell terminal, run the command
````
vault secrets enable transit
````

6. Create an encryption key ring named `my-key` by executing the following command:
````
vault write -f transit/keys/my-key
````
7. Run the following commands to carry out the encryption of the file:

````
# Read the contents of the file into a byte array
$fileContent = [System.IO.File]::ReadAllBytes('C:\Users\user\test.txt')

# Convert the byte array content to a Base64-encoded string
$base64Content = [System.Convert]::ToBase64String($fileContent)

# Encrypt the Base64-encoded content using the Transit Secrets Engine in Vault
$encryptedContent = vault write -field=ciphertext transit/encrypt/my-key plaintext=$base64Content

# Overwrite the original file with the encrypted content
$encryptedContent > 'C:\Users\user\test.txt'
````

8. Decrypt file:
````
# Read the encrypted content from the file
$encryptedContent = Get-Content -Raw -Path 'C:\Users\user\test.txt'

# Decrypt the content using the Transit Secrets Engine in Vault
$decryptedContent = vault write -field=plaintext transit/decrypt/my-key ciphertext=$encryptedContent

# Convert the decrypted Base64-encoded content back to a byte array
$decryptedBytes = [System.Convert]::FromBase64String($decryptedContent)

# Overwrite the original file with the decrypted content
[System.IO.File]::WriteAllBytes('C:\Users\user\test.txt', $decryptedBytes)
````
