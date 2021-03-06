# [Terraform AWS Certonid Module](https://registry.terraform.io/modules/certonid/certonid/aws/latest)

## How to generate certonid function with symmetric encryption (not using AWS KMS keys)

```terraform
terraform {
  required_version = ">= 0.12"
}

provider "aws" {
  region = "eu-central-1"
}
provider "archive" {}

data "archive_file" "serverless_function" {
  type        = "zip"
  source_dir  = "./serverless/"
  output_path = "./build/serverless.zip"
}

module "terraform-aws-certonid-symmetric" {
  source = "certonid/certonid/aws"

  function_zip_file = data.archive_file.serverless_function.output_path
  symmetric_encryption_key = "<use 'certonid randstr' to generate one>"

  clients_names = [
    "users name 1",
    "users name 2"
  ]
}
```

#### Cli config

```yml
certificates:
  yourcoolname:
    public_key_path: ~/.ssh/id_ed25519.pub
    username: <your aws user name>
    runner: aws
    valid_until: 2h
    aws:
      profile: <your aws profile>
      region: eu-central-1
      function_name: CertonidCertificateGenerator
```

## How to create one certonid function with AWS KMS encryption

```terraform
terraform {
  required_version = ">= 0.12"
}

provider "aws" {
  region = "eu-central-1"
}
provider "archive" {}

data "archive_file" "serverless_function" {
  type        = "zip"
  source_dir  = "./serverless/"
  output_path = "./build/serverless.zip"
}

module "terraform-aws-certonid-eu-central-1" {
  source = "certonid/certonid/aws"

  function_zip_file = data.archive_file.serverless_function.output_path

  clients_names = [
    "iam_users_name"
  ]
}
```

#### Cli config

```yml
certificates:
  yourcoolname:
    public_key_path: ~/.ssh/id_ed25519.pub
    username: <your aws user name>
    runner: aws
    valid_until: 2h
    aws:
      profile: <your aws profile>
      region: eu-central-1
      function_name: CertonidCertificateGenerator
```

## How to create multi region certonid functions (with AWS KMS encryption, eu-central-1 and us-east-1 in example)

```terraform
terraform {
  required_version = ">= 0.12"
}

provider "aws" {
  region = "eu-central-1"
}

provider "aws" {
  alias  = "useast1"
  region = "us-east-1"
}

provider "archive" {}

data "archive_file" "serverless_function_eu-central-1" {
  type        = "zip"
  source_dir  = "./serverless-eu-central-1/"
  output_path = "./build/serverless-eu-central-1.zip"
}

data "archive_file" "serverless_function_us-east-1" {
  type        = "zip"
  source_dir  = "./serverless-us-east-1/"
  output_path = "./build/serverless-us-east-1.zip"
}

module "terraform-aws-certonid-eu-central-1" {
  source = "certonid/certonid/aws"

  providers = {
    aws = aws
  }

  function_zip_file = data.archive_file.serverless_function_eu-central-1.output_path
  function_iam_role_name = "certonid-lambda-role-eu-central-1" # you need provide uniq name for function IAM role
  function_iam_general_policy_name = "certonid-lambda-policy-eu-central-1" # you need provide uniq name for function general IAM policy
  function_iam_kms_policy_name = "certonid-lambda-kms-policy-eu-central-1" # you need provide uniq name for function KMS IAM policy
  clients_iam_policy_name = "certonid-clients-policy-eu-central-1" # you need provide uniq name for clients IAM policy

  clients_names = [
    "certonid-test-user"
  ]
}

module "terraform-aws-certonid-us-east-1" {
  source = "certonid/certonid/aws"

  providers = {
    aws = aws.useast1
  }

  function_zip_file = data.archive_file.serverless_function_us-east-1.output_path
  function_iam_role_name = "certonid-lambda-role-us-east-1" # you need provide uniq name for function IAM role
  function_iam_general_policy_name = "certonid-lambda-policy-eu-central-1" # you need provide uniq name for function general IAM policy
  function_iam_kms_policy_name = "certonid-lambda-kms-policy-eu-central-1" # you need provide uniq name for function KMS IAM policy
  clients_iam_policy_name = "certonid-clients-policy-us-east-1" # you need provide uniq name for clients IAM policy

  is_group_for_clients_exists = true # users managed in another function by 'clients_names' variable
}
```

#### Cli config

```yml
certificates:
  yourcoolname:
    public_key_path: ~/.ssh/id_ed25519.pub
    username: <your aws user name>
    runner: aws
    valid_until: 2h
    aws:
      profile: <your aws profile>
      region: eu-central-1
      function_name: CertonidCertificateGenerator
    failover:
    - region: us-east-1
```

## How to create multi region certonid functions with kmsauth (with AWS KMS encryption, eu-central-1 and us-east-1 in example)

```terraform
terraform {
  required_version = ">= 0.12"
}

provider "aws" {
  region = "eu-central-1"
}

provider "aws" {
  alias  = "useast1"
  region = "us-east-1"
}

provider "archive" {}

data "archive_file" "serverless_function_eu-central-1" {
  type        = "zip"
  source_dir  = "./serverless-eu-central-1/"
  output_path = "./build/serverless-eu-central-1.zip"
}

data "archive_file" "serverless_function_us-east-1" {
  type        = "zip"
  source_dir  = "./serverless-us-east-1/"
  output_path = "./build/serverless-us-east-1.zip"
}

module "terraform-aws-certonid-eu-central-1" {
  source = "certonid/certonid/aws"

  providers = {
    aws = aws
  }

  function_zip_file = data.archive_file.serverless_function_eu-central-1.output_path
  function_iam_role_name = "certonid-lambda-role-eu-central-1" # you need provide uniq name for function IAM role
  function_iam_general_policy_name = "certonid-lambda-policy-eu-central-1" # you need provide uniq name for function general IAM policy
  function_iam_kms_policy_name = "certonid-lambda-kms-policy-eu-central-1" # you need provide uniq name for function KMS IAM policy
  clients_iam_policy_name = "certonid-clients-policy-eu-central-1" # you need provide uniq name for clients IAM policy

  is_kmsauth_enabled = true # activate kmsauth
  function_iam_kmsauth_policy_name = "certonid-kmsauth-lambda-policy-eu-central-1" # you need provide uniq name for kmsauth IAM policy

  clients_names = [
    "certonid-test-user"
  ]
}

module "terraform-aws-certonid-us-east-1" {
  source = "certonid/certonid/aws"

  providers = {
    aws = aws.useast1
  }

  function_zip_file = data.archive_file.serverless_function_us-east-1.output_path
  function_iam_role_name = "certonid-lambda-role-us-east-1" # you need provide uniq name for function IAM role
  function_iam_general_policy_name = "certonid-lambda-policy-eu-central-1" # you need provide uniq name for function general IAM policy
  function_iam_kms_policy_name = "certonid-lambda-kms-policy-eu-central-1" # you need provide uniq name for function KMS IAM policy
  clients_iam_policy_name = "certonid-clients-policy-us-east-1" # you need provide uniq name for clients IAM policy

  is_kmsauth_enabled = true # activate kmsauth
  function_iam_kmsauth_policy_name = "certonid-kmsauth-lambda-policy-us-east-1" # you need provide uniq name for kmsauth IAM policy

  is_group_for_clients_exists = true # users managed in another function by 'clients_names' variable
  clients_iam_group_name = module.terraform-aws-certonid-eu-central-1.clients_iam_group_name
}
```

#### Cli config

```yml
certificates:
  yourcoolname:
    public_key_path: ~/.ssh/id_ed25519.pub
    username: <your aws user name>
    runner: aws
    valid_until: 2h
    aws:
      profile: <your aws profile>
      region: eu-central-1
      function_name: CertonidCertificateGenerator
      kmsauth:
        key_id: <put here module.terraform-aws-certonid-eu-central-1.kmsauth_kms_arn>
        service_id: certonid
        region: eu-central-1
        valid_until: 24h
    failover:
    - region: us-east-1
      kmsauth:
        key_id: <put here module.terraform-aws-certonid-us-east-1.kmsauth_kms_arn>
        service_id: certonid
        region: us-east-1
        valid_until: 24h
```
