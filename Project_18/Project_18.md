##  Automate Infrastructure With IaC using Terraform. Part 3 - Refactoring

In this project, more advanced concepts is introduced to enhance the AWS IAC developed in the previous projects.

The first concept to be introduced is [backend](https://www.terraform.io/docs/language/settings/backends/index.html)

Each terraform configurtion normally specifies a backend which is responsible for how Terraform stores its state, performs operations, and manages state locking. Terraform stores all the state of the infrastructure here in a ``json`` format.

Backend could either be ``local`` or ``remote`` backend. So far we have used the local backend, which requires no special configuration, and the here the states file is stored 'locally'. This option is not ideal when working in a collaborative scenario where multiple developers/members are in a team, bacause as can be predicted access to your 'local' device will be limited to you only and not to the team. Cloud or ``remote backend`` is  ideal when working in a team of other engineers/ developers. One of the cloud rescources provided by ``AWS``, a cloud service is [``S3 buckets, as a backend``](https://www.terraform.io/docs/language/settings/backends/s3.html).
No doubt S3 backends provides robust storage and east collaborations between teams. Another useful option that is supported by S3 bacekend is [State locking -](https://www.terraform.io/docs/language/state/locking.html).
State locking is used to loc the state for all operations that could write state, this prevents others from acquiring the lock and potentially corrupting your state. State locking feature for S3 backedn is optional and requires another AWS services - [DynamoDB](https://aws.amazon.com/dynamodb/)

To change the Terraform Configuration to use S3 bucket as backend, we have to `` re-initialize`` Terraform.

 * Add S3 bucket and DynamoDB resources blocks before deleting the local state file.
 * Update the terraform block to introduce the backend and locking
 * Re-initialize terraform
 * Delete the local ``tfstate`` file and check the one in S3 bucket.
 * Add ``outputs`` 
 * ``terraform apply``

 To know more on locking with DynamoDB, [check here](https://angelo-malatacca83.medium.com/aws-terraform-s3-and-dynamodb-backend-3b28431a76c1).

 So starting with switching to S3 bucket as backend,

 1. Add S3 bucket resource to the terraform block. NB-  Use  a unique name since S3 buckets have to be unique globally in AWS.

 ```
# Note: The bucket name may not work for you since buckets are unique globally in AWS, so you must give it a unique name.
resource "aws_s3_bucket" "terraform_state" {
  bucket = "dev-terraform-bucket"
  # Enable versioning so we can see the full revision history of our state files
  versioning {
    enabled = true
  }
  # Enable server-side encryption by default
  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }
}
``` 
 Terraform stores secret data (Passwords & secret keys) inside the state files, these are processed by resources when required. Therefore is vital to consider to always enable encryption. You can seee how we achived that with [server_side_encryption_configuration](https://docs.aws.amazon.com/AmazonS3/latest/userguide/serv-side-encryption.html).

 * Create DynamoDB table to handle locks and perfrom consistency checks. Locks are handled with a local file, `` terraform.tfstate.lock.info`` when local backend is in use. But in use of S3 as  a backend to store state file, DynamoDB(a cloud storage database) is used to handle the locks. so that anyone running Terraform against this infrastructure can use a central location to control a sittuation where terrafrom is rnning at the same time from multiplr different people.


Dynamo DB resources for locking and consistency checking:

```
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  attribute {
    name = "LockID"
    type = "S"
  }
}
```

Terraform expects the S3 bukcets and DynamoDB resources to be created fiest before the backend configuration is implemented. So after the above resources have been put in the block, run ``terrafrom apply`` to provision resources.


![S3 bucket and DynamoDB provisioning](<images/2 apply complete.jpg>)
*Showing the S3 bucket and DynamoDB provisioning.*

2. Configure s3 Backend.

```
terraform {
  backend "s3" {
    bucket         = "dev-terraform-bucket"
    key            = "global/s3/terraform.tfstate"
    region         = "eu-central-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}

```

Then run ``terraform init`` and procced to type ``yes`` if the ouput is desired.

![backend](<images/4 main backend config terraform init.jpg>)


3. Verify the changes.

* In the AWS UI, oberve the above resources are present.

![s3 bucket ](<images/6 s3 bucket content - tfstate.jpg>)
*S3 bucket with the state file object in it*





![DynamoDB table](<images/7 dyhnamodb table.jpg>)
*DynamoDB containing the 'locks' table*


More observation into the DynamoDB table to check changes BEFORE ``terraform apply`` and AFTER.
![DynamoDB tables](<images/7a dynamodb table.jpg>)
*Before ``terraform plan``*


![DynamoDB tables](<images/7b dynamodb1 after terraform plan.jpg>) ![alt text](<images/7c dynamodb 2 after terraform plan.jpg>)
*After ``terraform plan``*


4. Add Terraform Output
Add an ``output`` object so that the S3 bucket [Amazon Resource Names ARN](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html) and DynamoDB table name can be displayed.
In an ``output.tf`` file, paste below:

```
output "s3_bucket_arn" {
  value       = aws_s3_bucket.terraform_state.arn
  description = "The ARN of the S3 bucket"
}
output "dynamodb_table_name" {
  value       = aws_dynamodb_table.terraform_locks.name
  description = "The name of the DynamoDB table"
}
``` 
![outputs](<images/8 output.ft.jpg>)

5. Run ``terraform apply`` 
Terraform will automatically read the latest state from the S3 bucket to determine the current state of the infrastructure. Even if another engineer has applied changes, the state file will always be up to date.
Go to the S3 console, and observe the versions.

![versions](<images/8a versions.jpg>)
![versions content](<images/8b versions content.jpg>)

*With help of remote backend and locking configuration that we have just configured, collaboration is no longer a problem.*