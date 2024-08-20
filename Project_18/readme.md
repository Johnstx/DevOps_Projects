##  Automate Infrastructure With IaC using Terraform. Part 3 - Refactoring

In this project, more advanced concepts is introduced to enhance the AWS IAC developed in the previous projects.

The first concept to be introduced is [backend](https://www.terraform.io/docs/language/settings/backends/index.html)

Each terraform configurtion normally specifies a backend which is responsible for how Terraform stores its state, performs operations, and manages state locking. Terraform stores all the state of the infrastructure here in a ``json`` format.

Backend could either be ``local`` or ``remote`` backend. So far we have used the local backend, which requires no special configuration, and the here the states file is stored 'locally'. This option is not ideal when working in a collaborative scenario where multiple developers/members are in a team, bacause as can be predicted access to your 'local' device will be limited to you only and not to the team. Cloud or ``remote backend`` is  ideal when working in a team of other engineers/ developers. One of the cloud rescources provided by ``AWS``, a cloud service is [``S3 buckets, as a backend``](https://www.terraform.io/docs/language/settings/backends/s3.html).
No doubt S3 backends provides robust storage and east collaborations between teams. Another useful option that is supported by S3 bacekend is [State locking -](https://www.terraform.io/docs/language/state/locking.html).
State locking is used to loc the state for all operations that could write state, this prevents others from acquiring the lock and potentially corrupting your state. State locking feature for s# backedn is optional and requires another AWS services - [DynamoDB](https://aws.amazon.com/dynamodb/)

To change the Terraform Configuration to use S3 bucket as backend, we have to ``re-initialize`` Terraform.

 * Add S3 and DynamoDB resources blocks before deleting the local state file.
 * Update the terraform block to introduce the backend and locking