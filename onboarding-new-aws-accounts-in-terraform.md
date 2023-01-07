<!-- omit in toc -->
# ![Annalect](https://static-annalect.s3.amazonaws.com/annalect_logo.png) Onboard new AWS Account in Terraform
<!-- omit in toc -->

Table of Contents

- [Deployment overview](#deployment-overview)
- [Prerequisites for onboarding an account](#prerequisites-for-onboarding-an-account)
- [Deployment Steps - follow the order](#deployment-steps---follow-the-order)
  - [1. Create a repo or project for root terraform modules in GitLab](#1-create-a-repo-or-project-for-root-terraform-modules-in-gitlab)
  - [2. Update 'pylect-infra-terraform' module to incorporate new account](#2-update-pylect-infra-terraform-module-to-incorporate-new-account)
  - [3. Accommodate new account into terraform 'label' module](#3-accommodate-new-account-into-terraform-label-module)
  - [4. Bootstrap an AWS account with Terraform state backend](#4-bootstrap-an-aws-account-with-terraform-state-backend)
  - [5. Set up an AWS account with the reasonably secure configuration baseline](#5-set-up-an-aws-account-with-the-reasonably-secure-configuration-baseline)
  - [6. Build AWS VPC & VPC components in each allowed region](#6-build-aws-vpc--vpc-components-in-each-allowed-region)
  - [7. Deploy custom and standardized Security Groups and Ingress/Egress rules](#7-deploy-custom-and-standardized-security-groups-and-ingressegress-rules)
  - [8. Deploy the WAF (Web Application Firewall) module](#8-deploy-the-waf-web-application-firewall-module)
  - [9. Create managed IAM roles and policies for Administrative job functions](#9-create-managed-iam-roles-and-policies-for-administrative-job-functions)
  - [10. Deploy custom managed granular IAM policies](#10-deploy-custom-managed-granular-iam-policies)
  - [11. Deploy Fine Grained IAM Roles](#11-deploy-fine-grained-iam-roles)
  - [12. Add global roles using Okta SAML 2.0 Federation](#12-add-global-roles-using-okta-saml-20-federation)
  - [13. Create and upload SSH keys for EC2 and EMR resources](#13-create-and-upload-ssh-keys-for-ec2-and-emr-resources)
  - [14. Create Infra management S3 Buckets in each allowed region](#14-create-infra-management-s3-buckets-in-each-allowed-region)
  - [15. Deploy Management SNS topics in each allowed region](#15-deploy-management-sns-topics-in-each-allowed-region)
    - [15.1 Create `RDS` SNS topic for rds notifications](#141-create-rds-sns-topic-for-rds-notifications)
    - [15.2 Create a `RedShift` SNS topic for redshift events](#142-create-a-redshift-sns-topic-for-redshift-events)
    - [15.3 Create the `SUPPORT` SNS topic to alert Annalect DevOps](#143-create-the-support-sns-topic-to-alert-annalect-devops)
  - [16. Deploy standard KMS CMKs in each allowed region](#16-deploy-standard-kms-cmks-in-each-allowed-region)
  - [17. Deploy Cloud Intelligence Dashboards on AWS QuickSight](#17-deploy-cloud-intelligence-dashboards-on-aws-quicksight)
  - [18. Build temporary EC2 instances in a VPC and validate AD, DNS and VPN connectivity](#18-build-temporary-ec2-instances-in-a-vpc-and-validate-ad-dns-and-vpn-connectivity)
  - [19. Deploy infrastructure related lambda functions](#19-deploy-infrastructure-related-lambda-functions)
  - [20. Deploy DLM (Data Lifecycle Manager) Module](#20-deploy-dlm-data-lifecycle-manager-module)
  - [21. Deploy required AWS System Manager Resources for Patching and Maintenance](#21-deploy-required-aws-system-manager-resources-for-patching-and-maintenance)
  - [22. Export the ACM certificate ARN to the Terraform Remote State (Optional)](#22-export-the-acm-certificate-arn-to-the-terraform-remote-state-optional)
  - [23. Deploy federated okta roles for data engineers [DATAMESH]](#23-deploy-federated-okta-roles-for-data-engineers-datamesh)
  - [24. Add job and service specific managed IAM policies in each region [DATAMESH]](#24-add-job-and-service-specific-managed-iam-policies-in-each-region-datamesh)
  - [25. Add fine-grained IAM roles for project specific jobs and services in each region [DATAMESH]](#25-add-fine-grained-iam-roles-for-project-specific-jobs-and-services-in-each-region-datamesh)
  - [26. Create fine-grained IAM policies for data engineering team [DATAMESH]](#26-create-fine-grained-iam-policies-for-data-engineering-team-datamesh)
  - [27. Deploy Athena for data engineering team [DATAMESH]](#27-deploy-athena-for-data-engineering-team-datamesh)
  - [28. Deploy AWS batch compute requirements [DATAMESH]](#28-deploy-aws-batch-compute-requirements-datamesh)
  - [29. Deploy AWS batch job definitions [DATAMESH]](#29-deploy-aws-batch-job-definitions-datamesh)
  - [30. Deploy AWS batch job queues [DATAMESH]](#30-deploy-aws-batch-job-queues-datamesh)
  - [31. Set up the hosting service for Apache Airflow [DATAMESH]](#31-set-up-the-hosting-service-for-apache-airflow-datamesh)
  - [32. Deploy the managed emr solution [DATAMESH]](#32-deploy-the-managed-emr-solution-datamesh)
  - [33. Deploy project specific s3 buckets [DATAMESH]](#33-deploy-project-specific-s3-buckets-datamesh)
  - [34. Deploy team specific s3 buckets [DATAMESH]](#34-deploy-team-specific-s3-buckets-datamesh)
  - [35. Cleanup `accountonboardingadmin` role](#35-cleanup-accountonboardingadmin-role)
- [Document Version History](#document-version-history)

---

# Deployment overview

Annalect has adopted Terraform for automation to rapidly build complex, multitiered cloud environments, and enable Infrastructure as Code (IaC). This onboarding deployment document make it feasiblie to deploy and customize AWS account using Terraform modules.

# Prerequisites for onboarding an account

| :zap:        Ignore at your own risk!   |
|-----------------------------------------|

Ensure you meet the following requirements prior to deploying the resources.

- Have [Terraform Prerequisites](https://bitbucket.org/annalect/terraform_aws_modules/src/dc3013313cab/docs/get-started-with-terraform.md?at=master) installed and configued on your computer.
- Take a look at respective [AWS Account Details](https://bitbucket.org/annalect/terraform_aws_modules/src/dc3013313cab/docs/annalect-aws-accounts.md?at=master)
- Note the ARN of the `accountonboardingadmin` role to assume for initial deployment.
- Make sure you have access to new project (repo) in GitLab to create PRs.
- Add account-creation-checklist in respsective account specific folder on [sharepoint](https://oneomnicom.sharepoint.com/:f:/r/sites/OMG_Annalect-DevOps/Shared%20Documents/AWS/OMC-ACCOUNT-ONBOARDING?csf=1&web=1&e=AHmJnI) and keep the status updated as you deploy the module. Here's the [template](https://oneomnicom.sharepoint.com/:x:/r/sites/OMG_Annalect-DevOps/Shared%20Documents/AWS/OMC-ACCOUNT-ONBOARDING/TEMPLATES/account-onboarding-template.xlsx?d=w7834919f8e044cb39ab0f9288e8464f6&csf=1&web=1&e=E7WyCd).
  
---

| :warning: WARNING          |
|:---------------------------|
| 1. Follow documentation steps and deploy each module in order.|
| 2. Always pin module source to latest TAG version.|
| 3. Make sure you're creating resources under right terraform layer.|
| 4. Make sure to use appropriate provider alias to provision resources in required region.|
| 5. Do not add resources without provider, as it will create resouces in trusted account (TIO.|
| 6. Ensure to add or replace the local variable values as advised. Update/replace the local variable value with actual value as necessary.|
| 7. Have a feature branch for each module deployment, create PR for each module|
| 8. Always run `make test` until all tests pass before creating PR/MR.|
| 9. Install and activate the pre-commit hooks by executing `make ensure_pre_commit` command from the root of the repo.|
| 10. Share the account-creation-checklist with [Mocktar Tairou](mailto:mocktar.tairou@annalect.com) once all modules deployed.|


# Deployment Steps - follow the order

> **Note:** *Please help to improve the onboarding documentation. Email [Trenton Ornelas](mailto:trenton.ornelas@annalect.com) with your suggestions and recommendations.*


## 1. Create a repo or project for root terraform modules in GitLab

**NOTE:**

This step is usually done by Shibir, and NOT the person onboarding the account in Terraform.


0. These steps are to be followed by Shibir/Ayo, or anyone who has access to create repos.
1. Add a new project in the [GitLab](https://gitlab.annalect.com) under [terrform/aws](https://gitlab.annalect.com/terraform/aws) path.
2. Update configuration
3. Setup user permissions
4. Add branch protection
5. Create new branch 'master' and set it as default
6. Delete 'main' branch
7. Configure pipelines

> **Note:**
> Request internally to get the project created and configured in GitLab for the new account.


 <a href="#top">Back to top</a>
 
---
## 2. Update 'pylect-infra-terraform' module to incorporate new account

To update the mapping, you need to know the `account_alias` and `account_attr` for the new account. Take a look [here](https://bitbucket.org/annalect/terraform_aws_modules/src/master/docs/annalect-aws-accounts.md).

**Steps needed to update the pylect-infra-terraform (ptf) module**

1. Clone the `pylect-infra-terraform` repository found in [Bitbucket](https://bitbucket.org/annalect/pylect-infra-terraform).

```bash
$ git clone git@bitbucket.org:annalect/pylect-infra-terraform.git
```

2. Create a new feature branch from the `v.0.x` branch: 

```bash
$ git checkout v.0.x
$ git pull
$ git checkout -b v.0.x-new-account
```

3. Edit `pylect_infra_terraform/terraform.py` to add the new account information. The two maps to be changed are `account_attr_map` and `permitted_account_names`.

```bash
# replace vim with any text editor that you work with
$ vim pylect_infra_terraform/terraform.py
```

**What to change inside of `pylect_infra_terraform/terraform.py`**

```python
account_attr_map = {
    'ann01-tioprod':'ann01tio',
    'annalect-infrastructure':'ann01tio',
    'ann02-nonprod-ads':'ann02ads',
    'ann03-nonprod-assets':'ann03assets',
    'ann04-sandbox':'ann04sandbox',
    'ann05-nonprod-aadl' : 'ann05aadl',
    'ann06-prod-healthcaredata' : 'ann06ahcd',
    'ann07-prod-commerce-graph' : 'ann07acg',
    'ann08-dev' : 'ann08dev',
    'ann09-prod-fbcapig' : 'ann09capi',
    'ann10-prod-private' : 'ann10pvt',
    'ann25-prod-fsd' : 'ann25fsd',
    'ann27-prod-amd' : 'ann27amd',
    'ann28-prod-invgrf' : 'ann28invgrf',
    'ann12-prod-elde2' : 'ann12elde2',
    'new-account-alias' : 'new-account-attr' # append to bottom of the list
}
```

```python
permitted_account_names = {
  "ann01-tioprod": '661095214357',
  "ann02-nonprod-ads": '732327056170',
  "ann03-nonprod-assets": '952850296487',
  "ann04-sandbox": '091021251638',
  "ann05-nonprod-aadl": '571875174059',
  "ann06-prod-healthcaredata": "431361936815",
  "ann07-prod-commerce-graph": "914006819694",
  "ann08-dev": "408764662748",
  "ann09-prod-fbcapig": "153175995455",
  "ann10-prod-private": "288489547540",
  "ann25-prod-fsd": "895757120061",
  "ann27-prod-amd": "175813369871",
  "ann28-prod-invgrf": "366146577560",
  "ann12-prod-elde2": "155409914190",
  'new-account-alias' : 'new-account-number' # append to bottom of the list
}
```

4. When ready to push, make sure to discard all build/* changes.

5. Add and commit the files to git. Push to remote.

```bash
$ pip3 install --upgrade bumpversion
$ git add pylect_infra_terraform/terraform.py
$ git commit -m "Add account new-account-alias"
$ git push --set-upstream origin v.0.x-new-account
```

6. Create a PR and merge the feature branch into v.0.x.

7. Checkout v.0.x, and run the release script.


```bash 
$ git checkout v.0.x
$ git pull
$ ./release.sh patch
```

8. Push the branch.

```bash
$ git push
```

9. Push the new tags to remote.

```bash
$ git push --tags
```

10. Confirm the version has been tagged and pushed.

```bash
$ git log -n 1
```

11. Test ptf locally with our added account. 

```bash
$ pip3 install . --upgrade

$ mkdir -p ~/new-account-alias/aue1/env-dev/services/ec2 && cd ~/new-account-alias/aue1/env-dev/services/ec2
```

12. Run `pylect-infra-terraform start` (for brevity, `pylect-infra-terraform` will be aliased to `ptf`) and confirm the backend was configured with the new account details.

```bash
$ ptf start

# inspect the generated files to ensure that the right account info was imported
```

**TROUBLESHOOTING:** If errors arise, you may have to manually uninstall pylect-infra-terraform first, before installing pylect-infra-terraform locally.

```bash
# potential workaround for incorrect ptf error
$ pip3 uninstall pylect-infra-terraform
Found existing installation: pylect-infra-terraform 0.9.48
Uninstalling pylect-infra-terraform-0.9.48:
  Would remove:
    /Users/user/Library/Python/3.8/bin/pylect-infra-terraform
    /Users/user/Library/Python/3.8/bin/pylect-infra-terraform-start-project
    /Users/user/Library/Python/3.8/lib/python/site-packages/pylect_infra_terraform-0.9.48.dist-info/*
    /Users/user/Library/Python/3.8/lib/python/site-packages/pylect_infra_terraform/*
Proceed (Y/n)? y
  Successfully uninstalled pylect-infra-terraform-0.9.48   

# migrate back to local directory of ptf
$ cd ~/${ptf-installation-location}/pylect-infra-terraform

$ pip3 install . --upgrade
```

13. Return to your test folder and rerun `ptf start`.

14. Merge feature branch to the version branch: 

```bash
$ git checkout v.0.x; git pull; git merge v.0.x-new-account
```

 <a href="#top">Back to top</a>
 
---

## 3. Accommodate new account into terraform 'label' module

1. Update standard abbreviations and attributions document [here](https://bitbucket.org/annalect/terraform_aws_modules/src/master/docs/attribute_mapping.md).
2. Clone the `terraform_aws_modules` repo from [Annalect GitLab](https://gitlab.annalect.com/devops/terraform_aws_modules) onto your local computer.
3. Create a new feature branch from the `master` branch of the `terraform_aws_modules` repository.
4. Add the new account mapping in `/modules/label/main.tf` to `account_attr_mapping` and commit changes to the feature branch.

```tf
account_attr_mapping = {
  ann01-tioprod             = "ann01tio"
  annalect-infrastructure   = "ann01tio"
  ann02-nonprod-ads         = "ann02ads"
  ann03-nonprod-assets      = "ann03assets"
  ann04-sandbox             = "ann04sandbox"
  ann05-nonprod-aadl        = "ann05aadl"
  ann06-prod-healthcaredata = "ann06ahcd"
  ann07-prod-commerce-graph = "ann07acg"
  ann08-dev                 = "ann08dev"
  ann09-prod-fbcapig        = "ann09capi"
  ann10-prod-private        = "ann10pvt"
  ann-emea-dev              = "emeadev"
  ann25-prod-fsd            = "ann25fsd"
  ann27-prod-amd            = "ann27amd"
}
```

5. Merge feature branch into the `master` branch, and then merge `master` into `v.0.x`. The `v.0.x` branch is specifically used for the `label` module only to standardized naming convention and follow tagging standards. 
Merging any branch into `v.0.x` may cause issues. Rebasing locally and getting someone with access to push to `v.0.x` should allow fix the merge issue if the two branches are out of sync.

6. Rebase `v.0.x` after you merge the feature branch into `master`, then get someone with access to force push to reset HEAD

```bash
$ git checkout v.0.x
$ git fetch origin
$ git rebase origin/master

# fix any merge conflicts

# user with access to v.0.x push will have to run the following command locally, after rebase
$ git push origin HEAD --force
```


6. Follow the GitLab deployment process for terraform.

 <a href="#top">Back to top</a>
 
---

## 4. Bootstrap an AWS account with Terraform state backend

Navigate to the [annalect accounts](https://bitbucket.org/annalect/terraform_aws_modules/src/master/docs/annalect-aws-accounts.md) before going further. Important account information can be found here.

Prerequisites:

1. Ensure default VPC's have been deleted for each active region.
2. Clone a 'terraform root' repository (project) of respective account to your local computer.
3. Create a new feature branch from the `master` branch of the respective account specific repository.
4. The `terraform-backend` modules needs to be deployed from local laptop
5. Ensure the following files have been copied from another active account:

    You can find the files [here](https://bitbucket.org/annalect/terraform_aws_modules/src/master/).
    ```
    .gitignore
    .gitlab-ci.yml
    .pre-commit-config.yaml
    detect-and-process-folder-changes.sh
    plan-changes.sh
    ```

    Run the following command to link the terraform modules to the new account:
    
    ```
    $ ln -s ../terraform_aws_modules/modules modules
    ```

6. DevOps administrator whoever has access to assume the accountonboardingadmin can deploy this module.
7. Follows the steps below to create Annalect managed IAM policies.
8. IMPORTANT: Track your progress of the modules in the [sharepoint directory](https://oneomnicom.sharepoint.com/sites/OMG_Annalect-DevOps/Shared%20Documents/Forms/AllItems.aspx?id=%2Fsites%2FOMG%5FAnnalect%2DDevOps%2FShared%20Documents%2FAWS%2FOMC%2DACCOUNT%2DONBOARDING%2FCHECKLIST&viewid=d815743f%2D6d5f%2D4ff5%2D84bb%2D982eca8442c8&OR=Teams%2DHL&CT=1664899779224&clickparams=eyJBcHBOYW1lIjoiVGVhbXMtRGVza3RvcCIsIkFwcFZlcnNpb24iOiIyOC8yMjA3MzEwMTAwNyIsIkhhc0ZlZGVyYXRlZFVzZXIiOmZhbHNlfQ). Clone the onboarding template and modify as necessary.


**Step 1:  Start off by creating a directory for Terraform `bootstrap` layer and sub-layers with the following command:**

```bash
$ mkdir -p bootstrap/terraform-backend
```

**Step 2: Run the following pylect command to let pylect creates initial terraform codes inside terraform-backend**

```bash
$ cd bootstrap/terraform-backend
$ ${pylect-infra-terraform update}
$ pylect-infra-terraform start
```

**Step 3: Update the `main.tf` - change the terraform backend block from 's3' to 'local'.**

```tf
terraform {
  backend "local" {}
}
```

**Step 4: Write Terraform configuration files**

Add `state_backend.tf` file with below code.


```hcl
module "terraform_state_backend" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/tfstate-backend?ref=tags/v3.2.110"
  providers = {
    aws     = aws.aue1
    aws.src = aws.aue1
    aws.dst = aws.aew1
  }
  tf_bucket_source_region = var.location_region["aue1"]
  tf_bucket_dest_region   = var.location_region["aew1"]
  dynamodb_table          = var.trusting_account["dynamodb_table"]
  account_name            = var.trusting_account["name"]
  create_alias            = true

  tags                    = local.tags_infra
}
```
Note:  Make sure to pin the module source to the latest version/tag of the module.

**Step 5: Update `outputs.tf`.**

Append following to outputs.tf.

```tf
output "terraform_state_backend_state" {
  description = "Terraform State Backend S3 Bucket"
  value       = {
    id = module.terraform_state_backend.s3_bucket_id
    arn = module.terraform_state_backend.s3_bucket_arn
  }
}

output "terraform_state_lock_dynamodb" {
  description = "Terraform State Backend DynamoDB"
  value       = {
    table = module.terraform_state_backend.dynamodb_table_name,
    arn = module.terraform_state_backend.dynamodb_table_arn
  }
}

output "account_alias" {
  description = "Trusting IAM Account Alias"
  value       = module.terraform_state_backend.account_alias
}

```

**Step 6: Replace role_arn in `backend.hcl`.**

```tf
role_arn       = "arn:aws:iam::856889162721:role/accountonboardingadmin"
```
Note: Replace the account id with the account id of an onboarding account.

**Step 7: Update `terraform.tfvars`.**

Replace the `assume_role_arn` and `tf_state_read_arn` with the same role used above in `terraform.tfvars` for `trusting_account `block.

**Note:** Do NOT modify `trusted_account`.

```tf
# AWS Trusting Account Parameters
trusting_account = {
  id                  = "895757120061"
  name                = "ann25-prod-fsd"
  assume_role_arn     = "arn:aws:iam::856889162721:role/accountonboardingadmin"
  assume_role_profile = "default"
  # ... omitted
  tf_state_read_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"
}
```


Update values for : `Owner`, `Agency`, `Client`, `costcenter`, `access-team`, and `access-project`. 

```tf
common_tags = {
  ##... omitted

  Owner               = "DevOps"                   # Owner/Requester of the resource
  Owner2              = ""                         # Owner/Requester of the resource
  Agency              = "ANN"                      # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                 # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-infra" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-infra" as default for infra projects.
  access-team         = "devops"                   # Specify 'access-team', else set as devops.
  access-project      = "infra"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 8: Temporarily comment out aliases**

Comment out the following lines on `main.tf`

```tf

locals {

  ##...

  account_alias = data.aws_iam_account_alias.current.account_alias == "annalect-infrastructure" ? "ann01-tioprod" : data.aws_iam_account_alias.current.account_alias

  ##...

}

##...

data "aws_iam_account_alias" "current" {
  provider = aws.aue1
}

##...

```

Comment out the block on `outputs.tf`

```tf
output "trusting_account_alias" {
  description = "Trusting IAM Account Alias"
  value       = data.aws_iam_account_alias.current.account_alias
}
```



**Step 9: Validate code and Deploy the module with local state**

```bash
$ tfswitch
$ terraform fmt   
$ terraform init
$ terraform plan
$ terraform apply
```

**Step 10: Restore commented out aliases**

Uncomment the lines commented out in Step 8, then reapply the new configuration

```bash
# uncomment the 3 items, then run:

$ terraform fmt
$ terraform plan
$ terraform apply
```

**Step 11: Migrate the state file from local to S3**

1. Update the terraform backend block back to s3

```tf
terraform {
  backend "s3" {}
}
```

2. Reinitialize terraform and migrate existing state to the s3 backend

```bash
$ make migrate # answer yes to prompt
```

3. Execute following commands to update the configuration.

```bash
$ make ensure_pre_commit
$ make prep
$ make test (re-run `make test` until all tests pass)
$ make plan
$ make apply
```

**Step 12: Restore the roles changed in `backend.hcl` and `terraform.tfvars` and push to the repo**

Replace the roles changes made to `backend.hcl` and `terraform.tfvars`. 

Do not rerun `make apply` with the restored roles, as the module will fail without the required roles.

Push the code to the repo and create a PR.

```bash
$ git commit -m "message [CI-SKIP]" 
```

> **NOTE:** We use the `[CI-SKIP]` flag in our commit messages to indicate that we ONLY want to push code to the repo, since the roles we reestablished will not run.

| :point_up:    | UPDATE: Notify [Ayomide](mailto:ayomide.oribamise@annalect.com) to add Trenton to list of merge approvers! |
|---------------|:------------------------|

<a href="#top">Back to top</a>

---

## 5. Set up an AWS account with the reasonably secure configuration baseline

Deploy a terraform module to set up your AWS account with the reasonably secure configuration baseline.
Most configurations are based on [CIS Amazon Web Services Foundations v1.4.0](https://www.cisecurity.org/benchmark/amazon_web_services/) and [AWS Foundational Security Best Practices v1.0.0](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-standards-fsbp.html).

Prerequisites (Done in previous step):
1. Clone a 'terraform root' repository (project) of respective account from [Annalect GitLab](https://gitlab.annalect.com/terraform/aws) to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks executing `make ensure_pre_commit` command from the root of the repo.
4. Follow the steps below!

**Step 1: Create a directory for the new configuration under the root of the account repo.**

```bash
$ mkdir -p security/secure-baseline
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```bash
$ cd security/secure-baseline
$ pip3 install git+ssh://git@bitbucket.org/annalect/pylect-infra-terraform@v.0.x --upgrade
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

> **Notes:**
>  1. Add regions under `target_regions` as necessary for the account.
>  2. Depending on the account requirements, set `true` or `false` for the last two variables.
>  3. Add additional variables as local block to `locals.tf`
>  4. Update/replace the local variable value with actual value if required. 
>  5. If you migrate a variable to local, make sure to update the variable where it is called in `main.tf`.

```tf
locals {
  target_regions                  = ["us-east-1"]
  create_loadbalancer_logs_bucket = true
  create_cloudfront_logs_bucket   = true
}
```

**Step 4: Write Terraform configuration files**


In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

**secure_baseline.tf**

```tf
module "secure_baseline" {
  source                          = "git@bitbucket.org:annalect/terraform_aws_modules//modules/secure-baseline?ref=tags/v3.2.110"
  audit_log_bucket_name           = join("-", [var.trusting_account["name"], "audit", "bucket"])
  use_external_audit_log_bucket   = false
  aws_account_id                  = var.trusting_account["id"]
  aws_account_name                = var.trusting_account["name"]
  source_aws_account_ids          = [var.trusting_account["id"]]
  region                          = var.location_region["aue1"]
  support_iam_role_principal_arns = ["arn:aws:iam::${local.aws_account_id}:root"]
  target_regions                  = local.target_regions
  tags                            = local.standard_tags_map.infra

  default_vpc_exists = { us-east-1 = false }

  guardduty_enabled                            = true
  guardduty_s3_protection                      = false
  securityhub_enabled                          = true
  securityhub_enable_cis_standard              = true
  securityhub_enable_aws_foundational_standard = true
  securityhub_enable_pci_dss_standard          = false

  create_administrators_group     = false
  audit_log_bucket_force_destroy  = false
  create_loadbalancer_logs_bucket = local.create_loadbalancer_logs_bucket
  create_cloudfront_logs_bucket   = local.create_cloudfront_logs_bucket

  audit_bucket_lifecycle_rule         = false
  s3accesslog_bucket_lifecycle_rule   = false
  inventory_s3_bucket_lifecycle_rule  = false
  analytics_s3_bucket_lifecycle_rule  = false
  create_apigw_cloudwatch_global_role = true

  # CloudTrail
  cloudtrail_enabled = true
  # event_selector = [{
  #   read_write_type           = "All"
  #   include_management_events = true
  #    data_resource = [{
  #      type   = "AWS::S3::Object"
  #      values = ["arn:aws:s3:::"]
  #     }]
  # }]

  # SNS Topics
  cis_alarm_sns_topic_subscribers = {
    sub1 = {
      protocol = "email"
      endpoint = "devops@annalect.com"
    }
  }

  config_changes_sns_topic_subscribers = {
    sub1 = {
      protocol = "email"
      endpoint = "devops@annalect.com"
    }
  }

  cloudtrail_sns_topic_subscribers = {}

  # Enable/Disable AWS S3 public access at the AWS account level
  aws_s3_account_block_public_acls       = true
  aws_s3_account_block_public_policy     = true
  aws_s3_account_ignore_public_acls      = true
  aws_s3_account_restrict_public_buckets = true


  # audit_log_bucket_force_destroy = true
  # Setting it to true means all audit logs are automatically deleted when you run `terraform destroy`.
  # Note that it might be inappropriate for highly secured environment.


  # This module only configure regions specified in target_regions argument though,
  # all providers still need to be passed to the module.
  providers = {
    aws                = aws
    aws.us-east-1      = aws.us-east-1
    aws.ap-northeast-1 = aws.ap-northeast-1
    aws.ap-northeast-2 = aws.ap-northeast-2
    aws.ap-northeast-3 = aws.ap-northeast-3
    aws.ap-south-1     = aws.ap-south-1
    aws.ap-southeast-1 = aws.ap-southeast-1
    aws.ap-southeast-2 = aws.ap-southeast-2
    aws.ca-central-1   = aws.ca-central-1
    aws.eu-central-1   = aws.eu-central-1
    aws.eu-north-1     = aws.eu-north-1
    aws.eu-west-1      = aws.eu-west-1
    aws.eu-west-2      = aws.eu-west-2
    aws.eu-west-3      = aws.eu-west-3
    aws.sa-east-1      = aws.sa-east-1
    aws.us-east-2      = aws.us-east-2
    aws.us-west-1      = aws.us-west-1
    aws.us-west-2      = aws.us-west-2
    aws.me-south-1     = aws.me-south-1
  }
}
```

**Step 5: Add additional providers to `providers.tf`.**

**NOTE:**

The provider alias for me-south-1 appears to have a typo-- this is intentional.

Setting the region to me-south-1 will result in a bug. There are some AWS services that we use that are not supported in the me-south-1 region.

```tf
# Additional providers required for this module
provider "aws" {
  region  = var.region
  profile = var.trusting_account["assume_role_profile"]
  assume_role {
    role_arn = var.trusting_account["assume_role_arn"]
  }
}

provider "aws" {
  profile = var.trusting_account["assume_role_profile"]
  assume_role {
    role_arn = var.trusting_account["assume_role_arn"]
  }
  region = "eu-west-2"
  alias  = "me-south-1"
}

provider "aws" {
  profile = var.trusting_account["assume_role_profile"]
  assume_role {
    role_arn = var.trusting_account["assume_role_arn"]
  }
  region = "ap-northeast-1"
  alias  = "ap-northeast-1"
}

provider "aws" {
  profile = var.trusting_account["assume_role_profile"]
  assume_role {
    role_arn = var.trusting_account["assume_role_arn"]
  }
  region = "ap-northeast-2"
  alias  = "ap-northeast-2"
}

provider "aws" {
  profile = var.trusting_account["assume_role_profile"]
  assume_role {
    role_arn = var.trusting_account["assume_role_arn"]
  }
  region = "ap-northeast-3"
  alias  = "ap-northeast-3"
}

provider "aws" {
  profile = var.trusting_account["assume_role_profile"]
  assume_role {
    role_arn = var.trusting_account["assume_role_arn"]
  }
  region = "ap-south-1"
  alias  = "ap-south-1"
}

provider "aws" {
  profile = var.trusting_account["assume_role_profile"]
  assume_role {
    role_arn = var.trusting_account["assume_role_arn"]
  }
  region = "ap-southeast-1"
  alias  = "ap-southeast-1"
}

provider "aws" {
  profile = var.trusting_account["assume_role_profile"]
  assume_role {
    role_arn = var.trusting_account["assume_role_arn"]
  }
  region = "ap-southeast-2"
  alias  = "ap-southeast-2"
}

provider "aws" {
  profile = var.trusting_account["assume_role_profile"]
  assume_role {
    role_arn = var.trusting_account["assume_role_arn"]
  }
  region = "ca-central-1"
  alias  = "ca-central-1"
}

provider "aws" {
  profile = var.trusting_account["assume_role_profile"]
  assume_role {
    role_arn = var.trusting_account["assume_role_arn"]
  }
  region = "eu-central-1"
  alias  = "eu-central-1"
}

provider "aws" {
  profile = var.trusting_account["assume_role_profile"]
  assume_role {
    role_arn = var.trusting_account["assume_role_arn"]
  }
  region = "eu-north-1"
  alias  = "eu-north-1"
}

provider "aws" {
  profile = var.trusting_account["assume_role_profile"]
  assume_role {
    role_arn = var.trusting_account["assume_role_arn"]
  }
  region = "eu-west-1"
  alias  = "eu-west-1"
}

provider "aws" {
  profile = var.trusting_account["assume_role_profile"]
  assume_role {
    role_arn = var.trusting_account["assume_role_arn"]
  }
  region = "eu-west-2"
  alias  = "eu-west-2"
}

provider "aws" {
  profile = var.trusting_account["assume_role_profile"]
  assume_role {
    role_arn = var.trusting_account["assume_role_arn"]
  }
  region = "eu-west-3"
  alias  = "eu-west-3"
}

provider "aws" {
  profile = var.trusting_account["assume_role_profile"]
  assume_role {
    role_arn = var.trusting_account["assume_role_arn"]
  }
  region = "sa-east-1"
  alias  = "sa-east-1"
}


provider "aws" {
  profile = var.trusting_account["assume_role_profile"]
  assume_role {
    role_arn = var.trusting_account["assume_role_arn"]
  }
  region = "us-east-1"
  alias  = "us-east-1"
}

provider "aws" {
  profile = var.trusting_account["assume_role_profile"]
  assume_role {
    role_arn = var.trusting_account["assume_role_arn"]
  }
  region = "us-east-2"
  alias  = "us-east-2"
}

provider "aws" {
  profile = var.trusting_account["assume_role_profile"]
  assume_role {
    role_arn = var.trusting_account["assume_role_arn"]
  }
  region = "us-west-1"
  alias  = "us-west-1"
}

provider "aws" {
  profile = var.trusting_account["assume_role_profile"]
  assume_role {
    role_arn = var.trusted_account["assume_role_arn"]
  }
  region = "us-west-2"
  alias  = "us-west-2"
}
```

**Step 6: Update `outputs.tf`.**

You will have to change the `values = {}` for each output. Do this by uncommenting the AZ's to match the regions that your account is active in.

If the account is active in `us-east-1` and `eu-west-1` only, then no other changes to the below code will be required.

```tf
# Add additional outputs below
output "bucket" {
  description = "Audit Buckets"
  value = {
    audit      = module.secure_baseline.audit_bucket.bucket
    cloudtrail = "${module.secure_baseline.audit_bucket.bucket}/cloudtrail"
    config     = "${module.secure_baseline.audit_bucket.bucket}/config"
    redshift   = "${module.secure_baseline.audit_bucket.bucket}/redshift/"
  }
}
output "s3accesslogs" {
  description = "The s3 bucket that logs s3 access in each region ."

  value = {
    "aue1" = try(module.secure_baseline.s3access_bucket.us-east-1.bucket, "")
    "aew1" = try(module.secure_baseline.s3access_bucket.eu-west-1.bucket, "")
    # "ams1" = module.secure_baseline.s3access_bucket.me-south-1.bucket
    # "ap-northeast-1" = module.secure_baseline.s3access_bucket.ap-northeast-1.bucket
    # "ap-northeast-2" = module.secure_baseline.s3access_bucket.ap-northeast-2.bucket
    # "ap-south-1"     = module.secure_baseline.s3access_bucket.ap-south-1.bucket
    # "ap-southeast-1" = module.secure_baseline.s3access_bucket.ap-southeast-1.bucket
    # "ap-southeast-2" = module.secure_baseline.s3access_bucket.ap-southeast-2.bucket
    # "ca-central-1"   = module.secure_baseline.s3access_bucket.ca-central-1.bucket
    # "eu-central-1"   = module.secure_baseline.s3access_bucket.eu-central-1.bucket
    # "eu-north-1"     = module.secure_baseline.s3access_bucket.eu-north-1.bucket
    # "eu-west-2"      = module.secure_baseline.s3access_bucket.eu-west-2.bucket
    # "eu-west-3"      = module.secure_baseline.s3access_bucket.eu-west-3.bucket
    # "sa-east-1"      = module.secure_baseline.s3access_bucket.sa-east-1.bucket
    # "us-east-2"      = module.secure_baseline.s3access_bucket.us-east-2.bucket
    # "us-west-1"      = module.secure_baseline.s3access_bucket.us-west-1.bucket
    # "us-west-2"      = module.secure_baseline.s3access_bucket.us-west-2.bucket
  }
}

output "s3analytics" {
  description = "The s3 analytics bucket."

  value = {
    "aue1" = try(module.secure_baseline.s3analytics_bucket.us-east-1.bucket, "")
    "aew1" = try(module.secure_baseline.s3analytics_bucket.eu-west-1.bucket, "")
    # "ams1" = module.secure_baseline.s3analytics_bucket.me-south-1.bucket
    # "ap-northeast-1" = module.secure_baseline.s3analytics_bucket.ap-northeast-1.bucket
    # "ap-northeast-2" = module.secure_baseline.s3analytics_bucket.ap-northeast-2.bucket
    # "ap-south-1"     = module.secure_baseline.s3analytics_bucket.ap-south-1.bucket
    # "ap-southeast-1" = module.secure_baseline.s3analytics_bucket.ap-southeast-1.bucket
    # "ap-southeast-2" = module.secure_baseline.s3analytics_bucket.ap-southeast-2.bucket
    # "ca-central-1"   = module.secure_baseline.s3analytics_bucket.ca-central-1.bucket
    # "eu-central-1"   = module.secure_baseline.s3analytics_bucket.eu-central-1.bucket
    # "eu-north-1"     = module.secure_baseline.s3analytics_bucket.eu-north-1.bucket
    # "eu-west-2"      = module.secure_baseline.s3analytics_bucket.eu-west-2.bucket
    # "eu-west-3"      = module.secure_baseline.s3analytics_bucket.eu-west-3.bucket
    # "sa-east-1"      = module.secure_baseline.s3analytics_bucket.sa-east-1.bucket
    # "us-east-2"      = module.secure_baseline.s3analytics_bucket.us-east-2.bucket
    # "us-west-1"      = module.secure_baseline.s3analytics_bucket.us-west-1.bucket
    # "us-west-2"      = module.secure_baseline.s3analytics_bucket.us-west-2.bucket
  }
}

output "s3inventory" {
  description = "The s3 inventory bucket."

  value = {
    "aue1" = try(module.secure_baseline.s3inventory_bucket.us-east-1.bucket, "")
    "aew1" = try(module.secure_baseline.s3inventory_bucket.eu-west-1.bucket, "")
    # "ams1" = module.secure_baseline.s3inventory_bucket.me-south-1.bucket
    # "ap-northeast-1" = module.secure_baseline.s3inventory_bucket.ap-northeast-1.bucket
    # "ap-northeast-2" = module.secure_baseline.s3inventory_bucket.ap-northeast-2.bucket
    # "ap-south-1"     = module.secure_baseline.s3inventory_bucket.ap-south-1.bucket
    # "ap-southeast-1" = module.secure_baseline.s3inventory_bucket.ap-southeast-1.bucket
    # "ap-southeast-2" = module.secure_baseline.s3inventory_bucket.ap-southeast-2.bucket
    # "ca-central-1"   = module.secure_baseline.s3inventory_bucket.ca-central-1.bucket
    # "eu-central-1"   = module.secure_baseline.s3inventory_bucket.eu-central-1.bucket
    # "eu-north-1"     = module.secure_baseline.s3inventory_bucket.eu-north-1.bucket
    # "eu-west-2"      = module.secure_baseline.s3inventory_bucket.eu-west-2.bucket
    # "eu-west-3"      = module.secure_baseline.s3inventory_bucket.eu-west-3.bucket
    # "sa-east-1"      = module.secure_baseline.s3inventory_bucket.sa-east-1.bucket
    # "us-east-2"      = module.secure_baseline.s3inventory_bucket.us-east-2.bucket
    # "us-west-1"      = module.secure_baseline.s3inventory_bucket.us-west-1.bucket
    # "us-west-2"      = module.secure_baseline.s3inventory_bucket.us-west-2.bucket
  }
}

output "loadbalancer_logs_bucket" {
  description = "The loadbalancer logs bucket."

  value = {
    "aue1" = try(module.secure_baseline.loadbalaner_logs_bucket.us-east-1.bucket, "")
    "aew1" = try(module.secure_baseline.loadbalaner_logs_bucket.eu-west-1.bucket, "")
    # "ams1" = module.secure_baseline.loadbalaner_logs_bucket.me-south-1.bucket
    # "ap-northeast-1" = module.secure_baseline.loadbalaner_logs_bucket.ap-northeast-1.bucket
    # "ap-northeast-2" = module.secure_baseline.loadbalaner_logs_bucket.ap-northeast-2.bucket
    # "ap-south-1"     = module.secure_baseline.loadbalaner_logs_bucket.ap-south-1.bucket
    # "ap-southeast-1" = module.secure_baseline.loadbalaner_logs_bucket.ap-southeast-1.bucket
    # "ap-southeast-2" = module.secure_baseline.loadbalaner_logs_bucket.ap-southeast-2.bucket
    # "ca-central-1"   = module.secure_baseline.loadbalaner_logs_bucket.ca-central-1.bucket
    # "eu-central-1"   = module.secure_baseline.loadbalaner_logs_bucket.eu-central-1.bucket
    # "eu-north-1"     = module.secure_baseline.loadbalaner_logs_bucket.eu-north-1.bucket
    # "eu-west-2"      = module.secure_baseline.loadbalaner_logs_bucket.eu-west-2.bucket
    # "eu-west-3"      = module.secure_baseline.loadbalaner_logs_bucket.eu-west-3.bucket
    # "sa-east-1"      = module.secure_baseline.loadbalaner_logs_bucket.sa-east-1.bucket
    # "us-east-2"      = module.secure_baseline.loadbalaner_logs_bucket.us-east-2.bucket
    # "us-west-1"      = module.secure_baseline.loadbalaner_logs_bucket.us-west-1.bucket
    # "us-west-2"      = module.secure_baseline.loadbalaner_logs_bucket.us-west-2.bucket
  }
}

output "cloudfront_logs_bucket" {
  description = "The cloudfront logs bucket."

  value = {
    "aue1" = local.create_cloudfront_logs_bucket ? module.secure_baseline.cloudfront_logs_bucket.us-east-1.bucket : ""
    "aew1" = local.create_cloudfront_logs_bucket ? module.secure_baseline.cloudfront_logs_bucket.us-east-1.bucket : ""
    # "ams1" = module.secure_baseline.cloudfront_logs_bucket.us-east-1.bucket
    # "ap-northeast-1" = module.secure_baseline.cloudfront_logs_bucket.us-east-1.bucket
    # "ap-northeast-2" = module.secure_baseline.cloudfront_logs_bucket.us-east-1.bucket
    # "ap-south-1"     = module.secure_baseline.cloudfront_logs_bucket.us-east-1.bucket
    # "ap-southeast-1" = module.secure_baseline.cloudfront_logs_bucket.us-east-1.bucket
    # "ap-southeast-2" = module.secure_baseline.cloudfront_logs_bucket.us-east-1.bucket
    # "ca-central-1"   = module.secure_baseline.cloudfront_logs_bucket.us-east-1.bucket
    # "eu-central-1"   = module.secure_baseline.cloudfront_logs_bucket.us-east-1.bucket
    # "eu-north-1"     = module.secure_baseline.cloudfront_logs_bucket.us-east-1.bucket
    # "eu-west-2"      = module.secure_baseline.cloudfront_logs_bucket.us-east-1.bucket
    # "eu-west-3"      = module.secure_baseline.cloudfront_logs_bucket.us-east-1.bucket
    # "sa-east-1"      = module.secure_baseline.cloudfront_logs_bucket.us-east-1.bucket
    # "us-east-2"      = module.secure_baseline.cloudfront_logs_bucket.us-east-1.bucket
    # "us-west-1"      = module.secure_baseline.cloudfront_logs_bucket.us-east-1.bucket
    # "us-west-2"      = module.secure_baseline.cloudfront_logs_bucket.us-east-1.bucket
  }
}
```

**Step 7: Update `terraform.tfvars`.**

Update values for : `Owner`, `Agency`, `Client`, `costcenter`, `access-team`, and `access-project`.

```tf
common_tags = {
  ##... omitted

  Owner               = "DevOps"                   # Owner/Requester of the resource
  Owner2              = ""                         # Owner/Requester of the resource
  Agency              = "ANN"                      # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                 # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-infra" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-infra" as default for infra projects.
  access-team         = "devops"                   # Specify 'access-team', else set as devops.
  access-project      = "infra"                   # Specify 'access-project' name, else set as "infra".
}
```

Modify `assume_role_arn` and `tf_state_read_arn` in the `trusting_account`, NOT `trusted_account`:

> **NOTE:** Remember the roles that you replace, as they will eventually be restored to their original values

```tf
trusting_account {
  ##...
  assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"
  ##...
  tf_state_read_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin" 
}
```

**Step 8: Modify the `backend.hcl` roles**

> **NOTE:** Remember the roles that you replace, as they will eventually be restored to their original values.

Modify the `role_arn` in `backend.hcl`, temporarily, to be:

```tf
# backend.hcl
##...
role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"
##...
```

**Step 9: Initialize Terraform - Execute terraform commands with 'make' to initialize the directory and to format, validate the terraform configuration.**

Execute below commands:

```bash
$ make ensure_pre_commit
$ make prep
$ make test # rerun until all tests pass
$ make plan
$ make apply 
```

> **NOTE:** If you get an s3 error: cloudfront bucket doesnt exist, run `make apply` again.


**Step 10: Push the code to the repo, and replace the roles temporarily changed during onboarding**

Replace the roles changes made to `backend.hcl` and `terraform.tfvars`.

> **NOTE:** Do NOT rerun `make apply`, the modules will not work with the restored roles YET.

Push the code to the account repo

```bash
$ cd ../..
$ git add ${all_files_here}
$ git commit -m "Add secure-baseline [CI-SKIP]"
$ git push --set-upstream origin ${branch_name_here}
```

Create a MR on gitlab and get it approved.

> **NOTE:**  We use the `[CI-SKIP]` flag in our commit messages to indicate that we ONLY want to push code to the repo, since the roles we reestablished will not run.



 <a href="#top">Back to top</a>
 
---

## 6. Build AWS VPC & VPC components in each allowed region


Prerequisites:

1.  We assume you already meet the previously listed prerequisites.
2.  Confirm with Shashi/Kiran the number of environments required.
    
    1. **Situation One:** AWS account only has one environment, DEV|QA|STG|PROD.
       Use the primary/dev subnet block in the `locals.tf` file.
    2. **Situation Two:** AWS account has multiple environments, ex: DEV && QA.
       Use the primary/dev subnet block as well as the other required environment in the `locals.tf` file.

**Step 1: Create a directory for the new configuration under the root of the account repo.**

To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1'.

```bash
$ mkdir -p aue1/networking/base
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` (shown aliased as `ptf`) commands to generate the initial terraform configuration files.**

```bash
$ cd aue1/networking/base
$ ${pylect-infra-terraform start}
$ ptf start
```

**Step 3: Declare all the local variables in the locals block.**

Append the following onto the bottom of the locals folder.

**locals.tf**

> **Notes:**
>  1. Get all the VPC and subnet CIDR's and ranges approved by senior engineers **BEFORE** running `make apply`.
>  2. IF your account has one environent (ex: qa), then you will need to manually set `environment` and `environment_abbr` in `variables.tf` to the right environment for the subnets to have the right env name.

```tf
# Additional locals
locals {
  vpc_name = "main"

  vpc_cidr               = "100.104.90.0/23"
  vpc_region             = "us-east-1"
  domain_name            = "ec2.internal"
  domain_name_servers    = ["100.104.90.2"]
  ntp_servers            = ["169.254.169.123"]
  private_vpce_addresses = []
  delete_default_vpc     = true

  # Temporary fix
  primary_environment         = var.environment
  public_subnet_suffix        = "${var.environment}-public"
  private_subnet_suffix       = "${var.environment}-private"
  public_subnet_tags          = var.environment
  private_subnet_tags         = var.environment
  public_route_table_tags     = var.environment
  private_route_table_tags    = var.environment
  public_acl_tags             = var.environment
  private_acl_tags            = var.environment



  availability_zones = ["${local.vpc_region}a", "${local.vpc_region}b", "${local.vpc_region}c"]

  # Primary or Dev Subnets
  private_subnets = ["100.104.90.0/25", "100.104.90.128/25", "100.104.91.0/25"]
  public_subnets  = ["100.104.91.128/26", "100.104.91.192/27", "100.104.91.224/27"]


  # private_subnets = ["100.104.88.0/25", "100.104.88.128/25"]
  # public_subnets  = ["100.104.89.0/26", "100.104.89.64/26"]

  # Management Subnets
  # private_subnets_mgmt = ["100.104.89.128/27", "100.104.89.160/27"]
  # public_subnets_mgmt  = ["100.104.89.192/27", "100.104.89.224/27"]

  # Production Subnets
  # private_subnets_prod = []
  # public_subnets_prod  = []

  # QA Subnets
  # private_subnets_qa = []
  # public_subnets_qa  = []

  # Staging Subnets
  # private_subnets_stg = []
  # public_subnets_stg  = []
}
```

**Step 4: In the Terraform directory, create Terraform configuration files. Paste the configuration below into respective file and save it.**

**vpc.tf**

NOTE: ALWAYS update the module source to the latest version.

```tf
module "vpc" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/networking/vpc?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  # Destroy Default VPC
  delete_default_vpc = local.delete_default_vpc

  # VPC 
  name = local.vpc_name

  # VPC - CIDR, AZs and Subnet mapping  

  cidr = local.vpc_cidr
  azs  = local.availability_zones

  # Primary or Dev Subnets
  private_subnets         = local.private_subnets
  public_subnets          = local.public_subnets
  map_public_ip_on_launch = false

  # Management Subnets
  # private_subnets_mgmt         = local.private_subnets_mgmt
  # public_subnets_mgmt          = local.public_subnets_mgmt
  # map_mgmt_public_ip_on_launch = false

  # Production Subnets
  # private_subnets_prod = local.private_subnets_prod
  # public_subnets_prod  = local.public_subnets_prod
  # map_prod_public_ip_on_launch = false

  # QA Subnets
  # private_subnets_qa = local.private_subnets_qa
  # public_subnets_qa  = local.public_subnets_qa
  # map_qa_public_ip_on_launch = false

  # Staging Subnets
  # private_subnets_stg = local.private_subnets_stg
  # public_subnets_stg  = local.public_subnets_stg
  # map_stg_public_ip_on_launch = false


  # Redshift Subnets
  # redshift_subnets = []

  # Default security group - ingress/egress rules cleared to deny all
  manage_default_security_group  = false
  default_security_group_ingress = [{}]
  default_security_group_egress  = [{}]

  # Base VPC configuration
  enable_dhcp_options              = true
  dhcp_options_domain_name         = local.domain_name
  dhcp_options_domain_name_servers = local.domain_name_servers
  dhcp_options_ntp_servers         = local.ntp_servers

  enable_dns_hostnames = true
  enable_dns_support   = true

  manage_default_network_acl = true
  default_network_acl_tags   = { Name = "${local.vpc_name}-default-nacl" }
  default_network_acl_name   = "${local.vpc_name}-default-nacl"

  enable_classiclink             = false
  enable_classiclink_dns_support = false

  enable_nat_gateway     = true
  single_nat_gateway     = false
  one_nat_gateway_per_az = true

  # Database/Redshift Subnet Groups
  create_database_subnet_group = true
  create_redshift_subnet_group = true

  # Redshift
  create_redshift_subnet_route_table = false
  redshift_dedicated_network_acl     = false

  # VPC endpoint for S3
  enable_s3_endpoint = true

  # VPC endpoint for DynamoDB
  enable_dynamodb_endpoint = true

  # VPC endpoint for SSM
  enable_ssm_endpoint              = true
  ssm_endpoint_private_dns_enabled = true
  ssm_endpoint_security_group_ids  = local.private_vpc_endpoint_security_group_ids

  # VPC endpoint for SSMMESSAGES
  enable_ssmmessages_endpoint              = false
  ssmmessages_endpoint_private_dns_enabled = false
  ssmmessages_endpoint_security_group_ids  = local.private_vpc_endpoint_security_group_ids

  # VPC Endpoint for EC2
  enable_ec2_endpoint              = true
  ec2_endpoint_private_dns_enabled = true
  ec2_endpoint_security_group_ids  = local.private_vpc_endpoint_security_group_ids

  # VPC Endpoint for EC2MESSAGES
  enable_ec2messages_endpoint              = false
  ec2messages_endpoint_private_dns_enabled = false
  ec2messages_endpoint_security_group_ids  = local.private_vpc_endpoint_security_group_ids

  # VPC Endpoint for ECR API
  enable_ecr_api_endpoint              = false
  ecr_api_endpoint_private_dns_enabled = false
  ecr_api_endpoint_security_group_ids  = local.private_vpc_endpoint_security_group_ids

  # VPC Endpoint for ECR DKR
  enable_ecr_dkr_endpoint              = false
  ecr_dkr_endpoint_private_dns_enabled = false
  ecr_dkr_endpoint_security_group_ids  = local.private_vpc_endpoint_security_group_ids

  # VPC endpoint for KMS
  enable_kms_endpoint              = true
  kms_endpoint_private_dns_enabled = true
  kms_endpoint_security_group_ids  = local.private_vpc_endpoint_security_group_ids

  # VPC endpoint for ECS
  enable_ecs_endpoint              = true
  ecs_endpoint_private_dns_enabled = true
  ecs_endpoint_security_group_ids  = local.private_vpc_endpoint_security_group_ids

  # VPC endpoint for ECS telemetry
  enable_ecs_telemetry_endpoint              = false
  ecs_telemetry_endpoint_private_dns_enabled = false
  ecs_telemetry_endpoint_security_group_ids  = local.private_vpc_endpoint_security_group_ids

  # VPC endpoint for SQS
  enable_sqs_endpoint              = true
  sqs_endpoint_private_dns_enabled = true
  sqs_endpoint_security_group_ids  = local.private_vpc_endpoint_security_group_ids

  # VPC endpoint for API gateway
  enable_apigw_endpoint              = true
  apigw_endpoint_security_group_ids  = local.private_vpc_endpoint_security_group_ids
  apigw_endpoint_private_dns_enabled = true

  # VPC endpoint for Lambda
  enable_lambda_endpoint              = true
  lambda_endpoint_private_dns_enabled = true
  lambda_endpoint_security_group_ids  = local.private_vpc_endpoint_security_group_ids
  lambda_endpoint_subnet_ids          = local.private_vpc_endpoint_subnet_ids

  # VPC endpoint for Glue
  enable_glue_endpoint              = true
  glue_endpoint_private_dns_enabled = true
  glue_endpoint_security_group_ids  = local.private_vpc_endpoint_security_group_ids
  glue_endpoint_subnet_ids          = local.private_vpc_endpoint_subnet_ids

  # VPC endpoint for Athena
  enable_athena_endpoint              = true
  athena_endpoint_private_dns_enabled = true
  athena_endpoint_security_group_ids  = local.private_vpc_endpoint_security_group_ids
  athena_endpoint_subnet_ids          = local.private_vpc_endpoint_subnet_ids

  # VPC ENDPOINT FOR PRIVATE-VPCE-SERVICES
  # enable_private_vpce_endpoint              = true
  # private_vpce_endpoint_security_group_ids  = local.private_vpc_endpoint_security_group_ids
  # private_vpce_endpoint_service_names       = local.private_vpce_addresses
  # private_vpce_endpoint_private_dns_enabled = true
  # private_vpce_endpoint_subnet_ids          = local.private_vpc_endpoint_subnet_ids
  #tags                = local.vpce_tags

  # VPC endpoint for SageMaker
  # enable_sagemaker_api_endpoint                   = true
  # enable_sagemaker_notebook_endpoint              = true
  # enable_sagemaker_runtime_endpoint               = true
  # sagemaker_notebook_endpoint_region              = var.location_region["aue1"]
  # sagemaker_runtime_endpoint_private_dns_enabled  = true
  # sagemaker_api_endpoint_private_dns_enabled      = true
  # sagemaker_notebook_endpoint_private_dns_enabled = true
  # sagemaker_api_endpoint_security_group_ids       = local.private_vpc_endpoint_security_group_ids
  # sagemaker_notebook_endpoint_security_group_ids  = local.private_vpc_endpoint_security_group_ids
  # sagemaker_runtime_endpoint_security_group_ids   = local.private_vpc_endpoint_security_group_ids
  # sagemaker_notebook_endpoint_subnet_ids          = module.vpc.private_subnets_prod
  # sagemaker_api_endpoint_subnet_ids               = module.vpc.private_subnets_prod
  # sagemaker_runtime_endpoint_subnet_ids           = module.vpc.private_subnets_prod

  # Dedicated NACLs
  public_dedicated_network_acl  = true
  private_dedicated_network_acl = true

  # private_subnet_qa_dedicated_network_acl = false
  # public_subnet_qa_dedicated_network_acl  = false

  # private_subnet_stg_dedicated_network_acl = false
  # public_subnet_stg_dedicated_network_acl  = false

  # public_subnet_prod_dedicated_network_acl  = false
  # private_subnet_prod_dedicated_network_acl = false

  # public_subnet_mgmt_dedicated_network_acl  = true
  # private_subnet_mgmt_dedicated_network_acl = false

  # Private Subnets - NACLs
  private_inbound_acl_rules = concat(
    local.network_acls["private_inbound"],
    # local.network_acls["default_inbound"],
  )
  private_outbound_acl_rules = concat(
    local.network_acls["private_outbound"],
    # local.network_acls["default_outbound"],
  )

  private_subnet_prod_inbound_acl_rules = concat(
    local.network_acls["private_inbound"],
    # local.network_acls["default_inbound"],
  )
  private_subnet_prod_outbound_acl_rules = concat(
    local.network_acls["private_outbound"],
    # local.network_acls["default_outbound"],
  )

  private_subnet_mgmt_inbound_acl_rules = concat(
    local.network_acls["private_inbound"],
    # local.network_acls["default_inbound"],
  )
  private_subnet_mgmt_outbound_acl_rules = concat(
    local.network_acls["private_outbound"],
    # local.network_acls["default_outbound"],
  )

  # Public Subnets - NACLs
  public_inbound_acl_rules = concat(
    local.network_acls["public_inbound"],
    # local.network_acls["default_inbound"],
  )
  public_outbound_acl_rules = concat(
    local.network_acls["public_outbound"],
    # local.network_acls["default_outbound"],
  )

  public_subnet_prod_inbound_acl_rules = concat(
    local.network_acls["public_inbound"],
    # local.network_acls["default_inbound"],
  )
  public_subnet_prod_outbound_acl_rules = concat(
    local.network_acls["public_outbound"],
    # local.network_acls["default_outbound"],
  )

  public_subnet_mgmt_inbound_acl_rules = concat(
    local.network_acls["public_inbound"],
    # local.network_acls["default_inbound"],
  )
  public_subnet_mgmt_outbound_acl_rules = concat(
    local.network_acls["public_outbound"],
    # local.network_acls["default_outbound"],
  )

  # VPC Flow Logs (Cloudwatch log group and IAM role will be created)
  enable_flow_log                      = true
  create_flow_log_cloudwatch_log_group = true
  create_flow_log_cloudwatch_iam_role  = false
  flow_log_traffic_type                = "ALL"
  flow_log_destination_type            = "cloud-watch-logs"
  # flow_log_destination_arn                        = "arn:aws:logs:us-east-1:${local.trusting_account_id}:log-group:default-vpc-flow-logs:*"
  flow_log_cloudwatch_iam_role_arn                = "arn:aws:iam::${local.trusting_account_id}:role/VPC_Flow_Logs"
  flow_log_cloudwatch_log_group_name_prefix       = join("-", [local.vpc_name, "vpc-flow-logs", ""])
  flow_log_cloudwatch_log_group_retention_in_days = 30

  # Tags
  tags                = local.tags
  private_subnet_tags = { "for-use-with-amazon-emr-managed-policies" = true, Environment = "dev", Placement = "private" }
  #private_subnet_prod_tags = {}
  #private_subnet_mgmt_tags = {}
  vpc_tags = merge(local.tags, { "Name" = local.vpc_name })
  vpc_endpoint_tags = merge({ Endpoint = "true" },
  { "Name" = join("-", [var.trusting_account["name"], "vpce"]) })
}
```
**vpc_sg.tf**

```tf
# Security group API Gateway
###########################
data "aws_security_group" "default" {
  provider = aws.aue1
  name     = "default"
  vpc_id   = module.vpc.vpc_id
}


# Private Link Security Groups
resource "aws_security_group" "privatelink" {
  provider    = aws.aue1
  name        = "privatelink-public-sec-grp"
  description = "Allows AWS PrivateLink Traffic over Public internet"

  vpc_id = module.vpc.vpc_id

  tags = {
    Name = "public-privatelink-sg"
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "internal_privatelink" {
  provider    = aws.aue1
  name        = "privatelink-private-sec-grp"
  description = "Allows AWS PrivateLink Traffic over internal and VPN network"

  vpc_id = module.vpc.vpc_id

  tags = {
    Name = "private-privatelink-sg"
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["100.64.0.0/10"]
  }

  ingress {
    from_port   = 444
    to_port     = 444
    protocol    = "tcp"
    cidr_blocks = ["100.64.0.0/10"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["10.5.224.0/20"]
  }

  ingress {
    from_port   = 444
    to_port     = 444
    protocol    = "tcp"
    cidr_blocks = ["10.5.224.0/20"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```
**nacl_rules.tf**

```tf
# Locals for NACLs
locals {
  network_acls = {
    default_inbound = [
      {
        rule_number = 900
        rule_action = "allow"
        protocol    = "all"
        cidr_block  = "0.0.0.0/0"
      }
    ]
    default_outbound = [
      {
        rule_number = 900
        rule_action = "allow"
        protocol    = "all"
        cidr_block  = "0.0.0.0/0"
      }
    ]
    public_inbound = [
      {
        rule_number = 100
        rule_action = "allow"
        from_port   = 80
        to_port     = 80
        protocol    = "tcp"
        cidr_block  = "0.0.0.0/0"
      },
      {
        rule_number = 110
        rule_action = "allow"
        from_port   = 443
        to_port     = 443
        protocol    = "tcp"
        cidr_block  = "0.0.0.0/0"
      },
      {
        rule_number = 120
        rule_action = "allow"
        protocol    = "all"
        cidr_block  = "100.0.0.0/8"
      },
      {
        rule_number = 130
        rule_action = "allow"
        protocol    = "all"
        cidr_block  = "10.0.0.0/8"
      },
      {
        rule_number     = 140
        rule_action     = "allow"
        from_port       = 80
        to_port         = 80
        protocol        = "tcp"
        ipv6_cidr_block = "::/0"
      },
      {
        rule_number     = 150
        rule_action     = "allow"
        from_port       = 443
        to_port         = 443
        protocol        = "tcp"
        ipv6_cidr_block = "::/0"
      },
      {
        rule_number = 160
        rule_action = "allow"
        cidr_block  = "0.0.0.0/0"
        protocol    = "icmp"
        from_port   = -1
        to_port     = -1
        icmp_type   = -1
        icmp_code   = -1
      },
      {
        rule_number = 170
        rule_action = "allow"
        from_port   = 1024
        to_port     = 65535
        protocol    = "tcp"
        cidr_block  = "0.0.0.0/0"
      },
    ]
    public_outbound = [
      {
        rule_number = 100
        rule_action = "allow"
        protocol    = "all"
        cidr_block  = "0.0.0.0/0"
      }
    ]
    private_inbound = [
      {
        rule_number = 100
        rule_action = "allow"
        from_port   = 80
        to_port     = 80
        protocol    = "tcp"
        cidr_block  = "0.0.0.0/0"
      },
      {
        rule_number = 110
        rule_action = "allow"
        from_port   = 443
        to_port     = 443
        protocol    = "tcp"
        cidr_block  = "0.0.0.0/0"
      },
      {
        rule_number = 120
        rule_action = "allow"
        protocol    = "all"
        cidr_block  = "100.0.0.0/8"
      },
      {
        rule_number = 130
        rule_action = "allow"
        protocol    = "all"
        cidr_block  = "10.0.0.0/8"
      },
      {
        rule_number     = 140
        rule_action     = "allow"
        from_port       = 80
        to_port         = 80
        protocol        = "tcp"
        ipv6_cidr_block = "::/0"
      },
      {
        rule_number     = 150
        rule_action     = "allow"
        from_port       = 443
        to_port         = 443
        protocol        = "tcp"
        ipv6_cidr_block = "::/0"
      },
      {
        rule_number = 160
        rule_action = "allow"
        cidr_block  = "0.0.0.0/0"
        protocol    = "icmp"
        from_port   = -1
        to_port     = -1
        icmp_type   = -1
        icmp_code   = -1
      },
      {
        rule_number = 170
        rule_action = "allow"
        from_port   = 1024
        to_port     = 65535
        protocol    = "tcp"
        cidr_block  = "0.0.0.0/0"
      },
    ]
    private_outbound = [
      {
        rule_number = 100
        rule_action = "allow"
        protocol    = "all"
        cidr_block  = "0.0.0.0/0"
      }
    ]
  }
}
```


**Step 5: Update `outputs.tf`.**

```tf
# Add additional outputs below
output "vpc_id" {
  description = "The ID of the VPC"
  value       = module.vpc.vpc_id
}

output "vpc_cidr" {
  description = "The CIDR block of the VPC"
  value       = module.vpc.vpc_cidr_block
}

output "vpc_name" {
  description = "The VPC Name"
  value       = "main"
}

output "vpcs" {
  description = "Specify if multiple vpcs"
  value = {
    main = {
      name = "main"
      id   = module.vpc.vpc_id
      cidr = module.vpc.vpc_cidr_block
    }
  }
}

# Subnets
output "private_az_subnet_ids" {
  value       = zipmap(["dev", "qa", "stg", "prod", "mgmt", "redshift"], [module.vpc.private_subnets, module.vpc.private_subnets_qa, module.vpc.private_subnets_stg, module.vpc.private_subnets_prod, module.vpc.private_subnets_mgmt, module.vpc.redshift_subnets])
  description = "private_az_subnet_ids"
}

output "public_az_subnet_ids" {
  value       = zipmap(["dev", "qa", "stg", "prod", "mgmt"], [module.vpc.public_subnets, module.vpc.public_subnets_qa, module.vpc.public_subnets_stg, module.vpc.public_subnets_prod, module.vpc.public_subnets_mgmt])
  description = "public_az_subnet_ids"
}


output "redshift_subnet_ids" {
  description = "List of IDs of redshift subnets"
  value       = module.vpc.redshift_subnets
}

output "redshift_az_subnet_ids" {
  value       = module.vpc.redshift_az_subnet_ids
  description = "redshift_az_subnet_ids"
}

output "dev_private_az_subnet_ids" {
  value       = module.vpc.dev_private_az_subnet_ids
  description = "dev_private_az_subnet_ids"
}

output "dev_public_az_subnet_ids" {
  value       = module.vpc.dev_public_az_subnet_ids
  description = "dev_public_az_subnet_ids"
}

output "qa_private_az_subnet_ids" {
  value       = module.vpc.qa_private_az_subnet_ids
  description = "qa_private_az_subnet_ids"
}

output "qa_public_az_subnet_ids" {
  value       = module.vpc.qa_public_az_subnet_ids
  description = "qa_public_az_subnet_ids"
}

output "stg_private_az_subnet_ids" {
  value       = module.vpc.stg_private_az_subnet_ids
  description = "stg_private_az_subnet_ids"
}

output "stg_public_az_subnet_ids" {
  value       = module.vpc.stg_public_az_subnet_ids
  description = "stg_public_az_subnet_ids"
}

output "prod_private_az_subnet_ids" {
  value       = module.vpc.prod_private_az_subnet_ids
  description = "prod_private_az_subnet_ids"
}

output "prod_public_az_subnet_ids" {
  value       = module.vpc.prod_public_az_subnet_ids
  description = "prod_public_az_subnet_ids"
}

output "mgmt_private_az_subnet_ids" {
  value       = module.vpc.mgmt_private_az_subnet_ids
  description = "mgmt_private_az_subnet_ids"
}

output "mgmt_public_az_subnet_ids" {
  value       = module.vpc.mgmt_public_az_subnet_ids
  description = "mgmt_public_az_subnet_ids"
}


# NAT gateways
output "nat_public_ips" {
  description = "List of public Elastic IPs created for AWS NAT Gateway"
  value       = module.vpc.nat_public_ips
}

output "natgw_ids" {
  description = "List of NAT Gateway IDs"
  value       = module.vpc.natgw_ids
}

output "natgw_eni_ids" {
  description = "The ENI ID of the network interface created by the NAT gateway."
  value       = module.vpc.natgw_eni_ids
}

# VPC endpoints
output "vpc_endpoint_ssm_id" {
  description = "The ID of VPC endpoint for SSM"
  value       = module.vpc.vpc_endpoint_ssm_id
}

output "vpc_endpoint_ssm_network_interface_ids" {
  description = "One or more network interfaces for the VPC Endpoint for SSM."
  value       = module.vpc.vpc_endpoint_ssm_network_interface_ids
}

output "vpc_endpoint_ssm_dns_entry" {
  description = "The DNS entries for the VPC Endpoint for SSM."
  value       = module.vpc.vpc_endpoint_ssm_dns_entry
}

# Route Table IDs
output "public_az_route_table_ids" {
  description = "List of IDs of public route tables"
  value       = zipmap(["dev", "qa", "stg", "prod", "mgmt"], [module.vpc.public_route_table_ids, module.vpc.public_qa_route_table_ids, module.vpc.public_stg_route_table_ids, module.vpc.public_prod_route_table_ids, module.vpc.public_mgmt_route_table_ids])
}

output "private_az_route_table_ids" {
  description = "List of IDs of private route tables"
  value       = zipmap(["dev", "qa", "stg", "prod", "mgmt"], [module.vpc.private_route_table_ids, module.vpc.private_qa_route_table_ids, module.vpc.private_stg_route_table_ids, module.vpc.private_prod_route_table_ids, module.vpc.private_mgmt_route_table_ids])
}

# NACLs
output "public_az_nacl_ids" {
  description = "ID of the public network ACL"
  value       = zipmap(["dev", "qa", "stg", "prod", "mgmt"], [module.vpc.public_network_acl_id, module.vpc.public_qa_network_acl_id, module.vpc.public_stg_network_acl_id, module.vpc.public_prod_network_acl_id, module.vpc.public_mgmt_network_acl_id])
}

output "private_az_nacl_ids" {
  description = "ID of the private network ACL"
  value       = zipmap(["dev", "qa", "stg", "prod", "mgmt"], [module.vpc.private_network_acl_id, module.vpc.private_qa_network_acl_id, module.vpc.private_stg_network_acl_id, module.vpc.private_prod_network_acl_id, module.vpc.private_mgmt_network_acl_id])
}

output "default_nacl_id" {
  description = "The ID of the default network ACL"
  value       = module.vpc.default_network_acl_id
}

output "redshift_nacl_id" {
  description = "ID of the redshift network ACL"
  value       = module.vpc.redshift_network_acl_id
}

output "private_az_subnet_cidr" {
  description = "List of cidr_blocks of private subnets"
  value       = zipmap(["dev", "qa", "stg", "prod", "mgmt", "redshift"], [module.vpc.private_subnets_cidr_blocks, module.vpc.private_subnets_qa_cidr_blocks, module.vpc.private_subnets_stg_cidr_blocks, module.vpc.private_subnets_prod_cidr_blocks, module.vpc.private_subnets_mgmt_cidr_blocks, module.vpc.redshift_subnets_cidr_blocks])
}

output "redshift_az_subnet_cidr" {
  description = "cidr_blocks of redshift subnets"
  value       = module.vpc.redshift_subnets_cidr_blocks
}


output "public_az_subnet_cidr" {
  description = "List of cidr_blocks of public subnets"
  value       = zipmap(["dev", "qa", "stg", "prod", "mgmt"], [module.vpc.public_subnets_cidr_blocks, module.vpc.public_subnets_qa_cidr_blocks, module.vpc.public_subnets_stg_cidr_blocks, module.vpc.public_subnets_prod_cidr_blocks, module.vpc.public_subnets_mgmt_cidr_blocks])
}

output "db_subnet_group" {
  description = "ID of database subnet group"
  value       = zipmap(["dev", "qa", "stg", "prod", "mgmt"], [module.vpc.database_subnet_group, module.vpc.database_subnet_group_qa, module.vpc.database_subnet_group_stg, module.vpc.database_subnet_group_prod, module.vpc.database_subnet_group_mgmt])
}

output "redshift_subnet_group" {
  description = "ID of redshift subnet group"
  value       = zipmap(["dev", "qa", "stg", "prod", "redshift"], [module.vpc.redshift_subnet_group_dev, module.vpc.redshift_subnet_group_qa, module.vpc.redshift_subnet_group_stg, module.vpc.redshift_subnet_group_prod, module.vpc.redshift_subnet_group])
}

output "prod_private_az_route_table_ids" {
  value       = module.vpc.prod_private_az_route_table_ids
  description = "prod_private_az_route_table_ids"
}
output "qa_private_az_route_table_ids" {
  value       = module.vpc.qa_private_az_route_table_ids
  description = "qa_private_az_route_table_ids"
}
output "stg_private_az_route_table_ids" {
  value       = module.vpc.stg_private_az_route_table_ids
  description = "stg_private_az_route_table_ids"
}
output "mgmt_private_az_route_table_ids" {
  value       = module.vpc.mgmt_private_az_route_table_ids
  description = "mgmt_private_az_route_table_ids"
}
output "dev_private_az_route_table_ids" {
  value       = module.vpc.dev_private_az_route_table_ids
  description = "dev_private_az_route_table_ids"
}
```

**Step 6: Update `terraform.tfvars`.**

Update `Owner`, `Agency`, `Client`, `Agency`, `costcenter`, `access-team`, and `access-project`. These are critical to reporting finances back to Annalect.

```tf
common_tags = {
  ##... omitted

  Owner               = "DevOps"                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-infra" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-infra" as default for infra projects.
  access-team         = "devops"                  # Specify 'access-team', else set as devops.
  access-project      = "infra"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Initialize Terraform - Execute terraform commands with 'make' to initialize the directory and to format, validate the terraform configuration.**

Note: Before executing terraform commands, make sure the following booleans are correct in `main.tf` to retrieve remote state data from Terraform backend.

```tf
  # Turn on and off to retrieve remote state from backend
  use_secure_baseline_remote_state = false
  use_vpc_remote_state             = false
  use_sg_remote_state              = false
  use_bootstrap_remote_state       = false
  use_iam_policy_remote_state      = false
```

**Step 8: Replace the roles temporarily to run the module:**

* Temporarily replace the `role_arn` in `backend.hcl` with `role/accountonboardingadmin`. 

* Temporarily replace the `assume_role_arn` and `tf_state_read_arn` in `trusting_account` with `role/accountonboardingadmin`, NOT `trusted_account`.

Remember the roles you replaced, as you will restore them right before pushing to the repo.

Execute below commands:

```bash
$ make ensure_pre_commit
$ make prep
$ make test # rerun until all tests pass
$ make plan
```

Get the final plan reviewed and approved by Shashi/Kiran, and then run `make apply`

```bash
$ make apply
```

**Step 9: Restore the existing roles, and then push the code to the repo:**

> **Notes:** 
> Restore the existing roles to `backend.hcl` and `terraform.tfvars`.
> Do not re-run the make commands. The pre-existing roles will not work at this point.


Push the feature branch to the repo, and create a MR to be merged into `master`.

 <a href="#top">Back to top</a>
 
---

## 7. Deploy custom and standardized Security Groups and Ingress/Egress rules

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit` command from the root of the repo.
4. Follows the steps below to create Annalect managed IAM policies.

**Step 1: Create a directory for the new configuration under the root of the account repo.**

To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1'.

```bash
$ mkdir -p aue1/networking/security_groups
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```bash
$ cd aue1/networking/security_groups
$ pip3 install git+ssh://git@bitbucket.org/annalect/pylect-infra-terraform@v.0.x --upgrade
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required. 

Change the `environments` variable to match the required environments for the account. "mgmt", however, should remain as it's required for management related security functions.

The openvpn and allow_from_global CIDR's should not change.

```tf
locals {
  environments                = ["dev", "mgmt"]
  redshift_cross_account_cidr = []
  rds_cross_account_cidr      = []
  openvpn_cidr_block          = ["10.5.224.128/25", "10.5.225.128/25"]
  allow_from_global_vpn_cidr  = ["10.5.224.22/32", "10.5.224.101/32"]
}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

**security_groups.tf**

-- NOTE: v3.2.113 was the LAST working version --

```tf
module "security_groups_main" {
  source = "git@bitbucket.org:annalect/terraform_aws_modules//modules/security-groups?ref=tags/v3.2.113"
  providers = {aws = aws.aue1}

  environments                = local.environments
  vpc                         = data.terraform_remote_state.vpc[0].outputs["vpcs"]["main"]
  redshift_cross_account_cidr = local.redshift_cross_account_cidr
  rds_cross_account_cidr      = local.rds_cross_account_cidr
  openvpn_cidr_block          = local.openvpn_cidr_block
  allow_from_global_vpn_cidr  = local.allow_from_global_vpn_cidr
  tags                        = local.tags_infra
}
```

**Step 5: Update `outputs.tf`.**

```tf
output "security_groups" {
  description = "VPC Security Groups"
  value = {
    main = tomap(module.security_groups_main)["security_groups"]
  }
}
```

**Step 6: Update `terraform.tfvars`.**

Update values for : `Owner`, `Agency`, `Client`, `costcenter`, `access-team`, and `access-project`.

```tf
common_tags = {
  ##... omitted

  Owner               = "DevOps"                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-infra" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-infra" as default for infra project.
  access-team         = "devops"                  # Specify 'access-team', else set as devops.
  access-project      = "infra"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Initialize Terraform - Execute terraform commands with 'make' to initialize the directory and to format, validate the terraform configuration.**

Note: Before executing terraform commands, make sure the following booleans are set to 'true' in main.tf to retrieve remote state data from Terraform backend.

```tf
# Make sure following boolean are turned on to activate ‘terraform_remote_state’ data source configuration.
use_vpc_remote_state             = true
```

Similarly to the previous steps, replace the `role_arn` in `backend.hcl` and the `assume_role_arn` in `trusting_account` under `terraform.tfvars` with `role/accountonboardingadmin`.

Execute below commands:

```bash
$ make ensure_pre_commit
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 8: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created.

 <a href="#top">Back to top</a>
 
---

## 8. Deploy the WAF (Web Application Firewall) module

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit` command from the root of the repo.
4. Follows the steps below to create the resources.

**Step 1: Create a directory for the new configuration under the root of the account repo.**

Repeat this entire section for each enabled region for the account.

To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1' at the root level of the directory.

```bash
$ mkdir -p security/wafv2
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```bash
$ cd security/wafv2
$ ${pylect-infra-terraform update}
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

> **Notes:**
>  1. Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required.
>  2. If no locals need to be added, skip this step.

```tf
locals {
  annalect_custom_whitelist_ip_list    = ["10.5.224.0/24", "10.5.225.0/24"]
  omc_network_custom_whitelist_ip_list = ["163.116.135.0/24", "103.219.78.0/24", "74.217.93.0/24", "163.116.128.0/17", "8.36.116.0/24", "31.186.239.0/24", "103.219.79.0/24", "8.39.144.0/24"]
  global_deny_ip_list                  = ["10.5.226.0/24", "10.5.227.0/24"]
}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

Add the required WAF modules into the directory.

**waf.tf**

```tf
module "cloudfront_waf" {
  source                   = "git@bitbucket.org:annalect/terraform_aws_modules//modules/waf?ref=tags/v3.2.110"
  providers                = { aws = aws.aue1 }
  scope                    = "CLOUDFRONT"
  global_deny_ip_list      = local.global_deny_ip_list
  custom_whitelist_ip_list = concat(local.annalect_custom_whitelist_ip_list, local.omc_network_custom_whitelist_ip_list)
  tags                     = local.tags_infra
}

module "alb_waf_us" {
  source = "git@bitbucket.org:annalect/terraform_aws_modules//modules/waf?ref=tags/v3.2.110"

  providers                = { aws = aws.aue1 }
  scope                    = "REGIONAL"
  global_deny_ip_list      = local.global_deny_ip_list
  custom_whitelist_ip_list = concat(local.annalect_custom_whitelist_ip_list, local.omc_network_custom_whitelist_ip_list)
  tags                     = local.tags_infra
}

module "alb_waf_eu" {
  source                   = "git@bitbucket.org:annalect/terraform_aws_modules//modules/waf?ref=tags/v3.2.110"
  providers                = { aws = aws.aew1 }
  scope                    = "REGIONAL"
  global_deny_ip_list      = local.global_deny_ip_list
  custom_whitelist_ip_list = concat(local.annalect_custom_whitelist_ip_list, local.omc_network_custom_whitelist_ip_list)
  tags                     = local.tags_infra
}
```

**main.tf**

Confirm all booleans for remote state match the following, in `main.tf`:

```tf
locals {
  ##...

  # Turn on and off to retrieve remote state from backend
  use_secure_baseline_remote_state = false
  use_vpc_remote_state             = false
  use_sg_remote_state              = false
  use_bootstrap_remote_state       = false
  use_iam_policy_remote_state      = false
}
```

**Step 5: Update Outputs**

Ensure all customer managed keys are available to other modules.

**outputs.tf**

```tf
# Add additional outputs below
output "waf_internal_web_acl" {
  description = "WAF Internal Web Acl"
  value = {
    "us-east-1" = {
      "dev"  = module.alb_waf_us.waf_internal_web_acl.arn
      "qa"   = module.alb_waf_us.waf_internal_web_acl.arn
      "stg"  = module.alb_waf_us.waf_internal_web_acl.arn
      "prod" = module.alb_waf_us.waf_internal_web_acl.arn
    },
    "eu-west-1" = {
      "dev"  = module.alb_waf_eu.waf_internal_web_acl.arn
      "qa"   = module.alb_waf_eu.waf_internal_web_acl.arn
      "stg"  = module.alb_waf_eu.waf_internal_web_acl.arn
      "prod" = module.alb_waf_eu.waf_internal_web_acl.arn
    },
    "cloudfront" = {
      "dev"  = module.cloudfront_waf.waf_internal_web_acl.arn
      "qa"   = module.cloudfront_waf.waf_internal_web_acl.arn
      "stg"  = module.cloudfront_waf.waf_internal_web_acl.arn
      "prod" = module.cloudfront_waf.waf_internal_web_acl.arn
    },
  }
}

output "waf_windows_web_acl" {
  description = "WAF windows Web Acl"
  value = {
    "us-east-1" = {
      "dev"  = module.alb_waf_us.waf_windows_web_acl.arn
      "qa"   = module.alb_waf_us.waf_windows_web_acl.arn
      "stg"  = module.alb_waf_us.waf_windows_web_acl.arn
      "prod" = module.alb_waf_us.waf_windows_web_acl.arn
    },
    "eu-west-1" = {
      "dev"  = module.alb_waf_eu.waf_windows_web_acl.arn
      "qa"   = module.alb_waf_eu.waf_windows_web_acl.arn
      "stg"  = module.alb_waf_eu.waf_windows_web_acl.arn
      "prod" = module.alb_waf_eu.waf_windows_web_acl.arn
    },
    "cloudfront" = {
      "dev"  = module.cloudfront_waf.waf_windows_web_acl.arn
      "qa"   = module.cloudfront_waf.waf_windows_web_acl.arn
      "stg"  = module.cloudfront_waf.waf_windows_web_acl.arn
      "prod" = module.cloudfront_waf.waf_windows_web_acl.arn
    },
  }
}

output "waf_linux_web_acl" {
  description = "WAF Linux Web Acl"
  value = {
    "us-east-1" = {
      "dev"  = module.alb_waf_us.waf_linux_web_acl.arn
      "qa"   = module.alb_waf_us.waf_linux_web_acl.arn
      "stg"  = module.alb_waf_us.waf_linux_web_acl.arn
      "prod" = module.alb_waf_us.waf_linux_web_acl.arn
    },
    "eu-west-1" = {
      "dev"  = module.alb_waf_eu.waf_linux_web_acl.arn
      "qa"   = module.alb_waf_eu.waf_linux_web_acl.arn
      "stg"  = module.alb_waf_eu.waf_linux_web_acl.arn
      "prod" = module.alb_waf_eu.waf_linux_web_acl.arn
    },
    "cloudfront" = {
      "dev"  = module.cloudfront_waf.waf_linux_web_acl.arn
      "qa"   = module.cloudfront_waf.waf_linux_web_acl.arn
      "stg"  = module.cloudfront_waf.waf_linux_web_acl.arn
      "prod" = module.cloudfront_waf.waf_linux_web_acl.arn
    }
  }
}
```

**Step 6: Update `terraform.tfvars`**

Update the required tags: 

* `Owner`
* `Agency`
* `Client`
* `costcenter`
* `access-team`
* `access-project`

```tf
common_tags = {
  ##... omitted

  Owner               = "DevOps"                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-infra" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-infra" as default for infra projects.
  access-team         = "devops"                  # Specify 'access-team', else set as devops.
  access-project      = "infra"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```tf
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```tf
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...

  - tf_state_read_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-terraform-shared-readonly"
  + tf_state_read_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```
> **NOTE:**  Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.

**Step 8: Apply the configuration, and push the feature branch**
Execute below commands:

```bash
$ make ensure_pre_commit
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 9: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch to master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.




 <a href="#top">Back to top</a>
 
---

## 9. Create managed IAM roles and policies for Administrative job functions
[Cloud Engineers & Architects]

This module also configure Okta as SAML Identity Provider

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit` command from the root of the repo.
4. Follows the steps below to add IAM Roles and Policies for Infra and Cloud Engineers.

**Step 1: Add `infra-admin-roles` folder as sub-layer under bootstrap.**

```bash
$ mkdir bootstrap/infra-admin-roles
```

**Step 2: Run the following pylect command to let pylect creates initial terraform codes inside infra-admin-roles folder**

```bash
$ cd bootstrap/infra-admin-roles
pip3 install git+ssh://git@bitbucket.org/annalect/pylect-infra-terraform@v.0.x --upgrade
$ pylect-infra-terraform start
```

**Step 3: Add following terraform files with code provided here.**

**infra_admin_access.tf**

```tf
module "infra_admin_access" {
  source = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/infra-admin-roles?ref=tags/v3.2.110"
  providers = {
    aws         = aws.aue1
    aws.trusted = aws.aue1
  }

  trusted_account_id            = var.trusted_account["id"]
  access_project                = local.tags_infra["access-project"]
  access_team                   = local.tags_infra["access-team"]
  nat_gw_eni_arn                = local.nat_gw_eni_arn
  vpc_ids                       = local.vpc_ids
  dev_subnet_ids                = local.dev_subnet_ids
  dev_security_group_ids        = local.dev_security_group_ids
  allowed_regions               = local.allowed_regions
  terraform_remote_state_bucket = "${local.aws_account_name}-terraform-state"
  terraform_dynamodb_table_name = "terraform-lock"

  create_template_based_custom_policy           = true
  prevent_data_exfiltration_policy              = true
  create_sandbox_boundary_policy                = false
  create_administrators_group                   = true
  administrators_group_users                    = local.administrators_group_users
  administrators_group_custom_group_policy_arns = ["arn:aws:iam::aws:policy/AdministratorAccess"]

  admin_boundary_deny_buckets     = local.admin_boundary_deny_buckets
  account_boundary_deny_buckets   = local.account_boundary_deny_buckets
  poweruser_boundary_deny_buckets = local.poweruser_boundary_deny_buckets
  devs_boundary_deny_buckets      = local.devs_boundary_deny_buckets
  sandbox_boundary_deny_buckets   = local.sandbox_boundary_deny_buckets

  account_boundary_deny_terraform_backend_key   = local.account_boundary_deny_terraform_backend_key
  poweruser_boundary_deny_terraform_backend_key = local.poweruser_boundary_deny_terraform_backend_key
  readonly_boundary_deny_terraform_backend_key  = local.readonly_boundary_deny_terraform_backend_key
  sandbox_boundary_deny_terraform_backend_key   = local.sandbox_boundary_deny_terraform_backend_key

  trusting_account_admins = local.trusting_account_admins
  sts_setsourceidentity   = local.sts_setsourceidentity

  service_control_policies = {
    admin-scp1 = {
      account_id     = local.aws_account_id
      nat_gw_eni_arn = local.nat_gw_eni_arn
    },
    admin-scp2 = {
      account_id          = local.aws_account_id
      dynamodb_table_name = var.trusting_account["dynamodb_table"]
    },
    scp1 = {
      restrict_buckets = local.account_boundary_deny_buckets
      dev_subnet_ids   = local.dev_subnet_ids
      vpc_ids          = [local.vpc_ids]
    },
    scp2 = {
      account_id = local.aws_account_id
    },
    scp3 = {
      boundary_policy = ["AccountBoundaryPolicy"]
    }
  }

  saml_provider_id = [module.okta_onewp.saml_provider_id]

  tags = local.tags_infra
}
```

**okta_idp.tf**

```tf
# Create SAML idP provider in AWS with AWS trust relation
module "okta_onewp" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/okta?ref=tags/v3.2.52"
  providers = { aws = aws.aue1 }

  description   = "Okta SAML Setup"
  provider_name = "okta_onewp"

  enable_okta_integration       = true
  role_permissions_boundary_arn = module.infra_admin_access.account_permissions_boundary_policy
  tags                          = merge(local.tags_infra, { "Name" = "Okta-Idp-cross-account-role" })
}
```

**Step 4: Add/append following contents to `locals.tf`.**

- Replace the variable values as appropriate to your account. Networking related parameter values can be updated once VPC is built.

```tf
locals {
  allowed_regions            = ["us-east-1", "eu-west-1"]
  administrators_group_users = []
  vpc_ids                    = try(tostring(data.terraform_remote_state.vpc[0].outputs.vpcs.main.id), "vpc-066666ee6666d6b6f")
  nat_gw_eni_arn             = try(data.terraform_remote_state.vpc[0].outputs.natgw_eni_ids, ["eni-066666d66666a6ef6"])
  dev_subnet_ids             = try(data.terraform_remote_state.vpc[0].outputs.private_az_subnet_ids["dev"], ["subnet-0be6ee666cc6666bb"])

  main_dev_app_sg    = try([data.terraform_remote_state.sg[0].outputs.security_groups.main.app["dev"]], ["sg-06666bc6c6a6666d6"])
  main_dev_lambda_sg = try([data.terraform_remote_state.sg[0].outputs.security_groups.main.lambda["dev"]], ["sg-06666bc6c6a6666d6"])
  main_airflow_sg    = try([data.terraform_remote_state.sg[0].outputs.security_groups.main.airflow["mixed"]], ["sg-06666bc6c6a6666d6"])
  main_dev_ecs_sg    = try([data.terraform_remote_state.sg[0].outputs.security_groups.main.ecs["dev"]], ["sg-06666bc6c6a6666d6"])

  dev_security_group_ids = compact(concat(local.main_dev_app_sg, local.main_dev_lambda_sg, local.main_airflow_sg, local.main_dev_ecs_sg))

  admin_boundary_deny_buckets = ["annalect-alienvault"]
  account_boundary_deny_buckets = [
    format("%v-%v", local.aws_account_name, "audit-bucket"),
    format("%v-%v", local.aws_account_name, "terraform-state-replica"),
    format("%v-%v", var.account_attr, "billing-cur"),
    format("%v-%v", var.account_attr, "infra-athena-query-results"),
    format("%v-%v", var.account_attr, "flowlogs-bucket"),
    format("%v-%v", var.account_attr, "infra-backups"),
    format("%v-%v", var.account_attr, "infra-logs"),
  ]
  poweruser_boundary_deny_buckets = [
    format("%v-%v", local.aws_account_name, "audit-bucket"),
    format("%v-%v", local.aws_account_name, "terraform-state"),
    format("%v-%v", local.aws_account_name, "terraform-state-replica"),
  ]
  devs_boundary_deny_buckets = [
    format("%v-%v", local.aws_account_name, "audit-bucket"),
    format("%v-%v", local.aws_account_name, "terraform-state"),
    format("%v-%v", local.aws_account_name, "terraform-state-replica"),
    format("%v-%v", var.account_attr, "billing-cur"),
  ]

  sandbox_boundary_deny_buckets = [
    format("%v-%v", local.aws_account_name, "audit-bucket"),
    format("%v-%v", local.aws_account_name, "terraform-state-replica"),
    format("%v-%v", var.account_attr, "billing-cur"),
  ]

  account_boundary_deny_terraform_backend_key   = ["bootstrap/*", "global/*", "security/*", "*/networking/*", "*/env-mgmt/*", "*/env-prod/*"]
  poweruser_boundary_deny_terraform_backend_key = ["bootstrap/*", "global/*", "security/*", "*/networking/*"]
  readonly_boundary_deny_terraform_backend_key  = ["bootstrap/*", "global/*", "security/*", "*/networking/*", "*/env-mgmt/*", "*/env-prod/*"]
  sandbox_boundary_deny_terraform_backend_key   = ["bootstrap/*", "global/*", "security/*", "*/networking/*", "*/env-mgmt/*"]

  trusting_account_admins = {
    access_team    = "devops"
    access_project = "infra"
    security       = []
    admins         = []
    networkadmins  = []
    powerusers     = []
    dbadmins       = []
    sysadmin       = []
    monitoring     = []
    backup         = []
    readonly       = []
  }

  sts_setsourceidentity = {
    security      = ["shashikant.kuwar@annalect.com", "gadupu.kumar@annalect.com"]
    admins        = ["shashikant.kuwar@annalect.com", "gadupu.kumar@annalect.com", "shibir.gargaravindra@annalect.com"]
    networkadmins = ["kodari.venkateswarlu@annalect.com"]
    powerusers    = ["anmichel.rodriguez@annalect.com", "kodari.venkateswarlu@annalect.com"]
    dbadmins      = ["anmichel.rodriguez@annalect.com", "shibir.gargaravindra@annalect.com", "kodari.venkateswarlu@annalect.com"]
    sysadmin = [
      "ayomide.oribamise@annalect.com",
      "divakar.matteddula@annalect.com",
      "mocktar.tairou@annalect.com",
      "trenton.ornelas@annalect.com",
      "vemula.kishore@annalect.com",
      "balaji.kalahasthi@annalect.com",
      "vamshikrishna.korivi@annalect.com",
      "manasa.koka@annalect.com"
    ]
    monitoring = [
      "ayomide.oribamise@annalect.com",
      "divakar.matteddula@annalect.com",
      "mocktar.tairou@annalect.com",
      "trenton.ornelas@annalect.com",
      "vemula.kishore@annalect.com",
      "balaji.kalahasthi@annalect.com",
      "vamshikrishna.korivi@annalect.com",
      "manasa.koka@annalect.com"
    ]
    backup = [
      "ayomide.oribamise@annalect.com",
      "divakar.matteddula@annalect.com",
      "mocktar.tairou@annalect.com",
      "trenton.ornelas@annalect.com",
      "vemula.kishore@annalect.com",
      "balaji.kalahasthi@annalect.com",
      "vamshikrishna.korivi@annalect.com",
      "manasa.koka@annalect.com"
    ]
    readonly = [
      "anmichel.rodriguez@annalect.com",
      "ayomide.oribamise@annalect.com",
      "divakar.matteddula@annalect.com",
      "gadupu.kumar@annalect.com",
      "mocktar.tairou@annalect.com",
      "shibir.gargaravindra@annalect.com",
      "trenton.ornelas@annalect.com",
      "vemula.kishore@annalect.com",
      "shashikant.kuwar@annalect.com",
      "balaji.kalahasthi@annalect.com",
      "vamshikrishna.korivi@annalect.com",
      "manasa.koka@annalect.com",
      "kodari.venkateswarlu@annalect.com"
    ]
    networkadmin_dev = [
      "anmichel.rodriguez@annalect.com",
      "ayomide.oribamise@annalect.com",
      "divakar.matteddula@annalect.com",
      "mocktar.tairou@annalect.com",
      "trenton.ornelas@annalect.com",
      "vemula.kishore@annalect.com",
      "balaji.kalahasthi@annalect.com",
      "vamshikrishna.korivi@annalect.com",
      "manasa.koka@annalect.com"
    ]
    dbadmin_dev = [
      "ayomide.oribamise@annalect.com",
      "divakar.matteddula@annalect.com",
      "mocktar.tairou@annalect.com",
      "trenton.ornelas@annalect.com",
      "vemula.kishore@annalect.com",
      "balaji.kalahasthi@annalect.com",
      "vamshikrishna.korivi@annalect.com",
      "manasa.koka@annalect.com"
    ]
  }
}
```

**Step 5: Add/append following contents to `outputs.tf`.**

```tf
output "saml_provider_name" {
  value       = module.okta_onewp.saml_provider_name
  description = "The name of the saml idp provider"
}

output "saml_provider_id" {
  value = [
    module.okta_onewp.saml_provider_id,
  ]
  description = "The id of the saml idp provider"
}

output "infra_admin_roles" {
  description = "Role ARNs for Infra Admins"
  value       = module.infra_admin_access.infra_admin_roles
}

output "boundary_policy_arn" {
  description = "Policy ARNs for Boundary Policies"
  value = {
    AccountBoundaryPolicy        = module.infra_admin_access.account_permissions_boundary_policy,
    DelegatedAdminBoundaryPolicy = module.infra_admin_access.admin_permissions_boundary_policy,
    ReadonlyRoleBoundaryPolicy   = module.infra_admin_access.readonly_role_permissions_boundary_policy,
    SandboxAdminBoundaryPolicy   = try(module.infra_admin_access.sandbox_permissions_boundary_policy, "")
  }
}

output "devops_administrator_group_users" {
  description = "Administrator Group Users"
  value       = module.infra_admin_access.admin_group_users
}

output "infra_admin_role_policies" {
  description = "Policy ARNs for Infra Admins Roles"
  value       = module.infra_admin_access.infra_admin_role_policies
}

output "service_control_policies" {
  description = "Annalect managed service control policy ARNs"
  value       = module.infra_admin_access.service_control_policies
}

output "region_boundary_policy" {
  description = "Annalect managed region boundary policy ARNs"
  value       = module.infra_admin_access.region_boundary_policy
}

output "other_policies" {
  description = "Other Annalect managed policies"
  value       = module.infra_admin_access.other_policies
}
```

**Step 6: Replace role_arn in `backend.hcl`.**

```tf
role_arn       = "arn:aws:iam::895757120061:role/accountonboardingadmin"
```
Note:
- Replace the account id with the account id of an onboarding account.
- Revert back the changes to role_arn once module is deployed.

**Step 7: Update `terraform.tfvars`.**

Replace the assume_role_arn  as above in `terraform.tfvars` for `trusting_account `block.

```tf
# AWS Trusting Account Parameters
trusting_account = {
  id                  = "895757120061"
  name                = "ann25-prod-fsd"
  assume_role_arn     = "arn:aws:iam::895757120061:role/accountonboardingadmin"
  assume_role_profile = "default"

  # ... omitted
}
```

Note:
- Do not replace the assume_role_arn for `trusted_account` map.
- Replace the account id with the account id of an onboarding account.
- You need to revert back the changes to assume_role_arn once module is deployed.

Update values for : `Owner`, `Agency`, `Client`, `costcenter`, `access-team`, and `access-project`.

```tf
common_tags = {
  ##... omitted

  Owner               = "DevOps"                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-infra" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-infra" as default for infra projects.
  access-team         = "devops"                  # Specify 'access-team', else set as devops.
  access-project      = "infra"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 8: Validate code and Deploy the module with local state**

```bash
$ make ensure_pre_commit
$ make prep
$ make test # rerun untill all tests pass
$ make plan
$ make apply
```
**Step 9: Change back role_arn and assume_role_arn in `backend.hcl` and `terraform.tfvars`, respectively.**

**backend.hcl**

```tf
role_arn       = "arn:aws:iam::895757120061:role/admin/ann23-prod-fsd-readonly"
```

**terraform.tfvars**

```tf
# AWS Trusting Account Parameters
trusting_account = {
  id                  = "895757120061"
  name                = "ann25-prod-fsd"
  assume_role_arn     = "arn:aws:iam::895757120061:role/admin/ann23-prod-fsd-readonly""
  assume_role_profile = "default"
  # ... omitted
}
```

**Step 10: Create the merge request to merge feature branch with master of account repo**

 <a href="#top">Back to top</a>
 
---

## 10. Deploy custom managed granular IAM policies

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit` command from the root of the repo.
4. Follows the steps below to create Annalect managed IAM policies.

**Step 1: Create a directory for the new configuration under the root of the account repo.**

To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1'.

```bash
$ mkdir -p global/iam-policies/annalect-managed
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```bash
$ cd global/iam-policies/annalect-managed
$ pip3 install git+ssh://git@bitbucket.org/annalect/pylect-infra-terraform@v.0.x --upgrade
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required.

```bash
# Additional locals
locals {}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

Create the configuration for Annalects ABAC strategy.

**attribute_based_access_policies.tf**

```tf
module "abac_policy" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-policy?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  description = "Annalect Attribute based access (ABAC) Policies"
  path        = "/admin/"

  # Labels
  name           = "abac"
  access_project = local.tags_infra["access-project"]
  access_team    = local.tags_infra["access-team"]
  attributes     = ["policy"]
  label_order    = ["account_attr", "access_project"]
  tags           = merge(local.tags_infra, { protected = "true" })

  create_abac_policy = true

  abac_policies = {
    abac-iam = {
      emr_editor = { access-project = [(local.access_project)], access-team = ["datateam"] }
    }
    abac1 = {
      billing_viewer           = []
      cloudshell_editor        = []
      aws_console_viewer_addon = { access-project = [(local.access_project)], access-team = ["datateam"] }
      awssupport_access        = { access-project = [(local.access_project)], access-team = ["datateam"] }
      cloudwatch_viewer        = { access-project = [(local.access_project)], access-team = ["datateam"] }
      cloudwatchlogs_viewer    = { access-project = [(local.access_project)], access-team = ["datateam"] }
    },
    abac2 = {
      allow_get_dev_ssm_parameters = []
      vpnclient_viewer             = []
      vpnclient_editor             = []
      codeartifact_viewer          = []
      codeartifact_editor          = []
      codebuild_report_viewer      = []
      ecs_viewer                   = { access-project = [(local.access_project)], access-team = ["datateam"] }
      codebuild_viewer             = []
      allow_ses_send_email         = { access-project = [(local.access_project)], access-team = ["datateam"] }
      sqs_viewer                   = { access-project = [(local.access_project)], access-team = ["datateam"] }
      eventbridge_viewer           = { access-project = [(local.access_project)], access-team = ["datateam"] }
      sns_viewer                   = { access-project = [(local.access_project)], access-team = ["datateam"] }
    },
    abac3 = {
      sagemaker_viewer = { access-project = [(local.access_project)], access-team = ["datateam"] }
      sagemaker_editor = { access-project = [(local.access_project)], access-team = ["datateam"] }
      awsbatch_viewer  = { access-project = [(local.access_project)], access-team = ["datateam", "pm"] }
      redshift_viewer  = { access-project = [(local.access_project)], access-team = ["datateam", "pm"] }
      lambda_viewer    = { access-project = [(local.access_project)], access-team = ["datateam"] }
      glue_viewer      = { access-project = [(local.access_project)], access-team = ["datateam", "pm"] }
      rds_viewer       = { access-project = [(local.access_project)], access-team = ["datateam", "pm"] }
      session_manager  = { access-project = [(local.access_project)], access-team = ["datateam"] }
    },
    abac4 = {
      batch_editor          = { access-project = [(local.access_project)], access-team = ["datateam"] }
      ecr_editor            = { access-project = [(local.access_project)], access-team = ["datateam"] }
      ecr_viewer            = { access-project = [(local.access_project)], access-team = ["datateam"] }
      parameterstore_editor = { access-project = [(local.access_project)], access-team = ["datateam"] }
    },
    abac5 = {
      secretsmanager_editor = []
      glue_editor           = { access-project = [(local.access_project)], access-team = ["datateam"] }
      airflow_viewer        = { access-project = [(local.access_project)], access-team = ["datateam", "pm"] }
      airflow_editor        = { access-project = [(local.access_project)], access-team = ["datateam"] }
      lakeformation_viewer  = { access-project = [(local.access_project)], access-team = ["pm", "datateam"] }
      lakeformation_editor  = { access-project = [(local.access_project)], access-team = ["datateam"] }
    },
    abac6 = {
      emr_viewer = { access-project = [(local.access_project)], access-team = ["datateam", "pm"] }
      emr_editor = { access-project = [(local.access_project)], access-team = ["datateam"] }
    },
    abac7 = {
      apigateway_viewer = { access-project = [(local.access_project)], access-team = ["datateam"] }
      dynamodb_viewer   = { access-project = [(local.access_project)], access-team = ["datateam", "pm"] }
      dynamodb_editor   = { access-project = [(local.access_project)], access-team = ["datateam"] }
    },
  }
}
```

**custom_managed_policies.tf**

```tf
module "custom_managed_policies" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-policy?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  enabled = true

  description = "Annalect custom managed IAM policies."

  # Labels
  name           = "managed-policy"
  access_project = local.tags_infra["access-project"]
  access_team    = local.tags_infra["access-team"]
  attributes     = ["managed", "policy"]
  label_order    = ["account_attr", "access_project"]
  tags           = merge(local.tags_infra, { protected = "true" })

  create_custom_managed_policy = true

  custom_managed_policies = {
    outpost24 = {
      account_id = local.aws_account_id
    },

    cloudhealth = {
      account_id = local.aws_account_id
    },

    s3listonly = {
      s3listonly_restricted_buckets = [
        format("%v-%v", local.aws_account_name, "audit-bucket"),
        format("%v-%v", local.aws_account_name, "terraform-state-replica"),
        format("%v-%v", var.account_attr, "billing-cur"),
        format("%v-%v", var.account_attr, "s3access-logs"),
        format("%v-%v", var.account_attr, "infra-athena-query-results"),
        format("%v-%v", var.account_attr, "flowlogs-bucket"),
        format("%v-%v", var.account_attr, "infra-backups"),
        format("%v-%v", var.account_attr, "infra-logs"),
      ]
    },

    awsview = {
      account_id = local.aws_account_id
    },

    ec2-automation = {
      ec2_automation_s3readonly    = ["arn:aws:s3:::Automationscripts"]
      ec2_automation_s3readwrite   = []
      ec2_automation_kmsid2decrypt = ["arn:aws:kms:*:${data.aws_caller_identity.trusted.account_id}:key/mrk-24eaf2d5b53a4387acc4b522463b9afc"]
      ec2_automation_secretsmanager_key = [
        "arn:aws:secretsmanager:*:${data.aws_caller_identity.trusted.account_id}:secret:ec2-automation-secrets-zTlRvm",
        "arn:aws:secretsmanager:*:${data.aws_caller_identity.trusted.account_id}:secret:datadog-keys-l0FkKf"
      ]
      ec2_automation_parameter_store_key = []
    },

    lakeformation-datalake-administrator = {
      lakeformation_workflow_role_arn = ["arn:aws:iam::${local.aws_account_id}:role/service-role/${var.account_attr}-${local.access_project}-lakeformation-workflow-role"]
      datalake_regions                = ["us-east-1"]
      athena_query_results_bucket     = []
    }

    lakeformation-data-engineer = {
      athena_workgroup_name           = [(local.access_project)]
      datalake_regions                = ["us-east-1"]
      athena_query_results_bucket     = []
      lakeformation_workflow_role_arn = ["arn:aws:iam::${local.aws_account_id}:role/service-role/${var.account_attr}-${local.access_project}-lakeformation-workflow-role"]
      glue_service_passrole_arns      = ["arn:aws:iam::${local.aws_account_id}:role/service-role/${var.account_attr}-${local.access_project}-glue-service-role"]
    }

    lakeformation-data-custodian = {
      account_id                  = local.aws_account_id
      athena_query_results_bucket = []
    }

    lakeformation-data-analyst = {
      account_id                  = local.aws_account_id
      athena_query_results_bucket = []
    }
  }
}
```

**scopedown_policies.tf**

```tf
module "scopedown_policy" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-policy?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  description = "Annalect scopedown policies for AWS services"
  path        = "/admin/"

  ## Labels
  name           = "scopedown"
  access_project = local.tags_infra["access-project"]
  access_team    = local.tags_infra["access-team"]
  attributes     = ["scopedown", "policy"]
  label_order    = ["account_attr", "access_project"]
  tags           = merge(local.tags_infra, { protected = "true" })

  create_scopedown_policy = true

  scopedown_policies = {
    lambda = {
      account_id                = local.aws_account_id
      lambda_security_group_ids = [data.terraform_remote_state.sg[0].outputs.security_groups.main.lambda["dev"]],
      lambda_subnet_ids         = data.terraform_remote_state.vpc[0].outputs.private_az_subnet_ids["dev"],
      vpc_ids                   = [data.terraform_remote_state.vpc[0].outputs.vpcs.main.id]
    },

    airflow = {
      account_id                 = local.aws_account_id
      airflow_security_group_ids = [data.terraform_remote_state.sg[0].outputs.security_groups.main.airflow["mixed"]],
      airflow_subnet_ids         = data.terraform_remote_state.vpc[0].outputs.private_az_subnet_ids["dev"],
      vpc_ids                    = [data.terraform_remote_state.vpc[0].outputs.vpcs.main.id],
      airflow_environment_roles  = ["${var.account_attr}-airflow/Admin", "${var.account_attr}-airflow/User"]
    },

    glue = {
      account_id = local.aws_account_id
    },

    awsbatch = {
      account_id              = local.aws_account_id
      batch_service_role_arns = []
      batch_job_role_arn      = []
    },

    emr = {
      account_id = local.aws_account_id
    },
  }
}
```

**infra_servicerole_policies.tf**

```tf
module "infra_service_role_policies" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-policy?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  enabled = true

  description = "Custom managed fine-grained IAM policy"
  path        = "/service-role/"

  # Labels
  name           = "service"
  access_project = local.tags_infra["access-project"]
  access_team    = local.tags_infra["access-team"]
  attributes     = ["servicerole", "policy"]
  label_order    = ["account_attr", "access_project"]
  tags           = merge(local.tags_infra, { protected = "true" })


  create_service_role_policy = true

  service_role_policies = {
    sagemaker = {
      account_id = local.aws_account_id
    },

    lambda = {
      account_id                = local.aws_account_id
      lambda_security_group_ids = [data.terraform_remote_state.sg[0].outputs.security_groups.main.lambda["dev"]],
      lambda_subnet_ids         = [data.terraform_remote_state.vpc[0].outputs.private_az_subnet_ids["dev"]],
      vpc_ids                   = [data.terraform_remote_state.vpc[0].outputs.vpcs.main.id]
    },

    s3batchops = {
      s3batchops_target_buckets   = []
      s3batchops_source_buckets   = []
      s3batchops_manifest_buckets = []
      s3batchops_report_buckets   = []
    },

    eventbridge = {
      account_id = local.aws_account_id
    },

    glue = {
      glue_s3_data_target     = []
      glue_s3_data_source     = []
      athena_datalake_regions = ["us-east-1"]
      athena_workgroup_name   = [local.access_project]
      athena_query_results_bucket = [
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-datateam",
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-athena"
      ]
    },

    emr = {
      emr_ec2_role_arn       = ["arn:aws:iam::${local.aws_account_id}:role/${local.account_attr}-${local.access_project}-${var.region_abbr}-emr-ec2-role"]
      emr_s3_data_target     = ["${local.account_attr}-${var.region_abbr}-${local.access_project}-datateam"]
      emr_s3_data_source     = []
      emr_security_group_ids = [data.terraform_remote_state.sg[0].outputs.security_groups.main.emr["master"], data.terraform_remote_state.sg[0].outputs.security_groups.main.emr["slave"], data.terraform_remote_state.sg[0].outputs.security_groups.main.emr["service"]],
      emr_subnet_ids         = data.terraform_remote_state.vpc[0].outputs.private_az_subnet_ids["dev"]
    },

    emrstudio = {
      emr_studio_workspace_storage = ["${var.account_attr}-*-emrstudio-workspace-storage"]
      emr_security_group_ids       = [data.terraform_remote_state.sg[0].outputs.security_groups.main.emr["studio"], data.terraform_remote_state.sg[0].outputs.security_groups.main.emr["workspace"]],
      emr_subnet_ids               = data.terraform_remote_state.vpc[0].outputs.private_az_subnet_ids["dev"]
    },

    mwaa = {
      mwaa_s3_data_target = []
      mwaa_s3_data_source = []
      mwaa_deny_buckets = [
        format("%v-%v", local.aws_account_name, "audit-bucket"),
        format("%v-%v", local.aws_account_name, "terraform-state"),
        format("%v-%v", local.aws_account_name, "terraform-state-replica"),
        format("%v-%v", var.account_attr, "billing-cur"),
      ]
      mwaa_kms_key_id            = []
      mwaa_batch_job_definitions = []
      mwaa_environment_name      = []
      athena_datalake_regions    = ["us-east-1"]
      athena_workgroup_name      = [local.access_project]
      athena_query_results_bucket = [
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-datateam",
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-athena"
      ]
    },

    bastion_dataconsumer = {
      bastion_consumer_source_buckets         = []
      bastion_consumer_glue_database_readonly = []
    },

    bastion_dataprovider = {
      bastion_provider_source_buckets         = []
      bastion_provider_glue_database_readonly = []
    },

    lakeformation = {
      lf_passrole_arns = []
    },

    codebuild = {
      codebuild_s3_data_source      = []
      codebuild_s3_data_target      = []
      codebuild_subnet_id           = []
      codebuild_parameter_store_key = []
      codebuild_secretsmanager_key  = []
      codebuild_kms_key_id          = []
      codebuild_passrole_arns       = []
    },

    batch = {
      account_id                     = local.aws_account_id
      batch_execution_role_arns      = []
      batch_s3_data_source           = []
      batch_s3_data_target           = []
      batch_parameter_store_key      = []
      batch_secretsmanager_key       = ["arn:aws:secretsmanager:${var.region}:${data.aws_caller_identity.trusted.account_id}:secret:bitbucket-deployment-key-*"]
      batch_kms_decrypt_key_id       = ["arn:aws:kms:${var.region}:${data.aws_caller_identity.trusted.account_id}:key/mrk-24eaf2d5b53a4387acc4b522463b9afc"]
      batch_passrole_arns            = []
      batch_ecr_pull_repository_arns = []
    },
  }
}
```


**Step 5: Update `outputs.tf`.**

```tf
# Add additional outputs below
output "infra_service_role_policies" {
  description = "Annalect managed service role policy ARNs"
  value       = module.infra_service_role_policies.oarn[*]
}

output "scopedown_policy_arn" {
  description = "Annalect managed service control policy ARNs"
  value       = module.scopedown_policy.oarn[*]
}

output "abac_policy_arn" {
  description = "Annalect managed attribute based access policy ARNs"
  value       = module.abac_policy.oarn[*]
}

output "managed_policy_arn" {
  description = "Annalect managed custom policy ARNs"
  value = [
    module.custom_managed_policies.oarn[*],
  ]
}
```

**Step 6: Update `terraform.tfvars`.**

Update values for : `Owner`, `Agency`, `Client`, `costcenter`, `access-team`, and `access-project`.

```tf
common_tags = {
  ##... omitted

  Owner               = "DevOps"                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-infra" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-infra" as default for infra projects.
  access-team         = "devops"                  # Specify 'access-team', else set as devops.
  access-project      = "infra"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Initialize Terraform - Execute terraform commands with 'make' to initialize the directory and to format, validate the terraform configuration.**

Note: Before executing terraform commands, make sure the following booleans are set to 'true' in main.tf to retrieve remote state data from Terraform backend.

```tf
# Make sure following boolean are turned on to activate ‘terraform_remote_state’ data source configuration.
use_vpc_remote_state             = true
use_sg_remote_state              = true
use_bootstrap_remote_state       = true
```

Similarly to the previous steps, replace the `role_arn` in `backend.hcl` and the `assume_role_arn` in `trusting_account` under `terraform.tfvars` with `role/accountonboardingadmin`.

Execute below commands:

```bash
$ make ensure_pre_commit
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 8: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created.

 <a href="#top">Back to top</a>
 
---

## 11. Deploy Fine Grained IAM Roles

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit` command from the root of the repo.
4. Follows the steps below to create Annalect managed IAM policies.
5. On AWS console, create a service quota request to raise the iam managed policies per role, if not raised already. The new value should be 20 (hard limit).

**Step 1: Create a directory for the new configuration under the root of the account repo.**

> **NOTE:**  The global folder will be under the root level, of the aws account.

```bash
$ mkdir -p global/iam-roles/annalect-managed
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```bash
$ cd global/iam-roles/annalect-managed
$ pip3 install git+ssh://git@bitbucket.org/annalect/pylect-infra-terraform@v.0.x --upgrade
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required.

```tf
locals {
  service_role_policies = {
    sagemaker = {
      account_id = local.aws_account_id
    },

    lambda = {
      account_id                = local.aws_account_id
      lambda_security_group_ids = [data.terraform_remote_state.sg[0].outputs.security_groups.main.lambda["dev"]],
      lambda_subnet_ids         = [data.terraform_remote_state.vpc[0].outputs.private_az_subnet_ids["dev"]],
      vpc_ids                   = [data.terraform_remote_state.vpc[0].outputs.vpcs.main.id]
    },

    s3batchops = {
      s3batchops_target_buckets   = []
      s3batchops_source_buckets   = []
      s3batchops_manifest_buckets = []
      s3batchops_report_buckets   = []
    },

    eventbridge = {
      account_id = local.aws_account_id
    },

    glue = {
      glue_s3_data_target = []
      glue_s3_data_source = []
    },

    emr = {
      emr_ec2_role_arn = []
    },

    mwaa = {
      mwaa_s3_data_target = []
      mwaa_s3_data_source = []
      mwaa_deny_buckets = [
        format("%v-%v", local.aws_account_name, "audit-bucket"),
        format("%v-%v", local.aws_account_name, "terraform-state"),
        format("%v-%v", local.aws_account_name, "terraform-state-replica"),
        format("%v-%v", var.account_attr, "billing-cur"),
      ]
      mwaa_kms_key_id            = []
      mwaa_batch_job_definitions = []
      mwaa_environment_name      = ["${var.account_attr}-airflow"]
    },

    bastion-dataconsumer = {
      bastion_consumer_source_buckets         = []
      bastion_consumer_glue_database_readonly = []
    },

    bastion-dataprovider = {
      bastion_provider_source_buckets         = []
      bastion_provider_glue_database_readonly = []
    },

    lakeformation = {
      lf_passrole_arns = []
    },

    codebuild = {
      codebuild_s3_data_source      = []
      codebuild_s3_data_target      = []
      codebuild_subnet_id           = []
      codebuild_parameter_store_key = []
      codebuild_secretsmanager_key  = []
      codebuild_kms_key_id          = []
      codebuild_passrole_arns       = []
    },

  }
}

locals {
  custom_role_policy_arns = {
    airflow_mwaa         = ["arn:aws:iam::${local.aws_account_id}:policy/service-role/${local.account_attr}-infra-mwaa-servicerole-policy"]
    bastion_dataconsumer = ["arn:aws:iam::${local.aws_account_id}:policy/service-role/${local.account_attr}-infra-bastion_dataconsumer-servicerole-policy"]
    bastion_dataprovider = ["arn:aws:iam::${local.aws_account_id}:policy/service-role/${local.account_attr}-infra-bastion_dataprovider-servicerole-policy"]
    awsbatch_automation  = []

    codebuild   = ["arn:aws:iam::${local.aws_account_id}:policy/service-role/${local.account_attr}-infra-codebuild-servicerole-policy"]
    emr_cluster = ["arn:aws:iam::${local.aws_account_id}:policy/service-role/${local.account_attr}-infra-emr-servicerole-policy"]
    emrstudio = [
      "arn:aws:iam::${local.aws_account_id}:policy/service-role/${local.account_attr}-infra-emrstudio-servicerole-policy"
    ]
    emr_ec2     = []
    eventbridge = ["arn:aws:iam::${local.aws_account_id}:policy/service-role/${local.account_attr}-infra-eventbridge-servicerole-policy"]
    glue        = ["arn:aws:iam::${local.aws_account_id}:policy/service-role/${local.account_attr}-infra-glue-servicerole-policy"]

    lakeformation_workflow = ["arn:aws:iam::${local.aws_account_id}:policy/service-role/${local.account_attr}-infra-lakeformation-servicerole-policy"]
    lambda                 = ["arn:aws:iam::${local.aws_account_id}:policy/service-role/${local.account_attr}-infra-lambda-servicerole-policy"]

    rds_monitoring = []
    s3batchops     = ["arn:aws:iam::${local.aws_account_id}:policy/service-role/${local.account_attr}-infra-s3batchops-servicerole-policy"]

    sagemaker = ["arn:aws:iam::${local.aws_account_id}:policy/service-role/${local.account_attr}-infra-sagemaker-servicerole-policy"]

    snowflake   = []
    outpost24   = ["arn:aws:iam::${local.aws_account_id}:policy/${local.account_attr}-infra-outpost24-managed-policy","arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-prevent-data-exfiltration-policy"]
    cloudhealth = ["arn:aws:iam::${local.aws_account_id}:policy/${local.account_attr}-infra-cloudhealth-managed-policy","arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-prevent-data-exfiltration-policy"]

    ciphertechs                  = ["arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-prevent-data-exfiltration-policy"]
    ssm_maintenance              = []
    ecs_task_execution_role_dev  = []
    ecs_task_execution_role_qa   = []
    ecs_task_execution_role_stg  = []
    ecs_task_execution_role_prod = []
    ecs_task_execution_role_mgmt = []

    lakeformation_datalake_administrator = [
      "arn:aws:iam::${local.aws_account_id}:policy/${local.account_attr}-infra-lakeformation-datalake-administrator-managed-policy",
      "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-service-control-policy-scp1",
      "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-service-control-policy-scp2",
      "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-service-control-policy-scp3",
      "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac1-policy",
      "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac2-policy",
      "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac3-policy",
      "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac4-policy",
      "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac5-policy",
      "arn:aws:iam::${local.aws_account_id}:policy/${local.account_attr}-infra-awsview-managed-policy"
    ]

    lakeformation_data_custodian = []
    lakeformation_data_analyst   = []
    lakeformation_data_engineer = [
      "arn:aws:iam::${local.aws_account_id}:policy/${local.account_attr}-infra-lakeformation-data-engineer-managed-policy",
      "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-aws-us-east-1-region-boundary-policy",
      "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-service-control-policy-scp1",
      "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-service-control-policy-scp2",
      "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-service-control-policy-scp3"
    ]
  }
}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

**infra_service_roles.tf**

```tf
module "infra_service_roles" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/infra-managed-roles-with-policies?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  description = "Annalect managed IAM roles for AWS services"

  # To create service roles, the boolean must be set to 'true'
  create_service_roles = true

  role_path = "/service-role/"

  # Labels
  name           = "service-role"
  access_project = local.tags_infra["access-project"]
  access_team    = local.tags_infra["access-team"]
  attributes     = ["service", "role"]
  label_order    = ["account_attr", "access_project"]
  tags           = merge(local.tags_infra, { protected = "true" })

  force_detach_policies = false

  custom_role_policy_arns = local.custom_role_policy_arns

  lakeformation_data_engineers         = ["shashikant.kuwar@annalect.com", "shibir.gargaravindra@annalect.com", "kodari.venkateswarlu@annalect.com"]
  lakeformation_datalake_administrator = ["shashikant.kuwar@annalect.com", "shibir.gargaravindra@annalect.com", "kodari.venkateswarlu@annalect.com"]
}
```

**service_linked_roles.tf**

```tf
# Creates AWS Service Linked Roles
resource "aws_iam_service_linked_role" "service_linked_roles" {
  provider = aws.aue1
  for_each = toset([
    "spotfleet.amazonaws.com",
    "ecs.amazonaws.com",
    "spot.amazonaws.com",
    "lakeformation.amazonaws.com"
  ])
  aws_service_name = each.key
}
```

**Step 5: Update `outputs.tf`.**

```tf
output "infra_service_roles" {
  description = "Service Role ARNs"
  value       = module.infra_service_roles.role_arns
}
```

**Step 6: Update `terraform.tfvars`.**

Update values for : `Owner`, `Agency`, `Client`, `costcenter`, `access-team`, and `access-project`.

```tf
common_tags = {
  ##... omitted

  Owner               = "DevOps"                     # Owner/Requester of the resource
  Owner2              = ""                           # Owner/Requester of the resource
  Agency              = "ANN"                        # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                   # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-infra" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-infra" as default for infra projects.
  access-team         = "devops"                     # Specify 'access-team', else set as devops.
  access-project      = "infra"                      # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Initialize Terraform - Execute terraform commands with 'make' to initialize the directory and to format, validate the terraform configuration.**

Note: Before executing terraform commands, make sure the following booleans are set to 'true' in main.tf to retrieve remote state data from Terraform backend.

```tf
# Make sure following boolean are turned on to activate ‘terraform_remote_state’ data source configuration.
use_vpc_remote_state             = true
use_sg_remote_state              = true
use_bootstrap_remote_state       = true
```

Similarly to the previous steps, replace the `role_arn` in `backend.hcl` and the `assume_role_arn` in `trusting_account` under `terraform.tfvars` with `role/accountonboardingadmin`.

Execute below commands:

```bash
$ make ensure_pre_commit
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 8: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created.


| :point_up:    | Notify [Mocktar Tairou](mailto:mocktar.tairou@annalect.com) once this module is deployed! |
|---------------|:------------------------|

 <a href="#top">Back to top</a>
 
---

## 12. Add global roles using Okta SAML 2.0 Federation

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit` command from the root of the repo.
4. Follows the steps below to create Annalect managed IAM policies.
5. You can create addditional okta roles required for project team.

**Step 1: Create a directory for the new configuration under the root of the account repo.**


```bash
$ mkdir -p global/iam-roles/okta
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```bash
$ cd global/iam-roles/okta
$ pip3 install git+ssh://git@bitbucket.org/annalect/pylect-infra-terraform@v.0.x --upgrade
$ pylect-infra-terraform start
```

**Step 3: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

**global.tf**

> **Notes:**
>  1. Note that there are THREE standard root modules to be added in `global.tf`


To add AWS account specific okta roles like readonly, billing, security-auditor etc.

```tf
module "readonly" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-assumable-roles-with-saml?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  create_trusted_role = true

  role_requires_setsourceidentity = true
  trusted_identity_account        = ["arn:aws:iam::${var.trusted_account["id"]}:root"]

  trusted_role_arns                     = ["arn:aws:iam::${var.trusted_account["id"]}:root"]
  max_session_duration                  = 43200
  trusted_role_permissions_boundary_arn = data.terraform_remote_state.bootstrap[0].outputs.boundary_policy_arn["AccountBoundaryPolicy"]
  standard_tags                         = merge(local.tags, { access-project = "awsreadonly", access-team = "devops" })
  trusted_role_name                     = "readonly"
  trusted_role_policy_arns = [
    "arn:aws:iam::aws:policy/ReadOnlyAccess",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-prevent-data-exfiltration-policy"
  ]
  saml_provider_id = data.terraform_remote_state.bootstrap[0].outputs.saml_provider_id

  role_sts_setsourceidentity = local.devops
}

module "billing" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-assumable-roles-with-saml?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  trusted_role_arns                     = ["arn:aws:iam::${var.trusted_account["id"]}:root"]
  max_session_duration                  = 43200
  trusted_role_permissions_boundary_arn = data.terraform_remote_state.bootstrap[0].outputs.boundary_policy_arn["AccountBoundaryPolicy"]
  standard_tags                         = merge(local.tags, { access-project = "billing", access-team = "billing", billing-admin = "true" })
  create_trusted_role                   = true
  trusted_role_policy_arns = [
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-prevent-data-exfiltration-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-service-control-policy-scp1",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-service-control-policy-scp2",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-service-control-policy-scp3",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac1-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac2-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac3-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac4-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac5-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/${local.account_attr}-infra-s3listonly-managed-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/${local.account_attr}-infra-awsview-managed-policy",
    "arn:aws:iam::aws:policy/job-function/ViewOnlyAccess"
  ]
  trusted_role_name = "billing"

  saml_provider_id = data.terraform_remote_state.bootstrap[0].outputs.saml_provider_id
}

module "security_auditor" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-assumable-roles-with-saml?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  create_trusted_role = true

  trusted_role_arns                     = ["arn:aws:iam::${var.trusted_account["id"]}:root"]
  max_session_duration                  = 43200
  trusted_role_permissions_boundary_arn = data.terraform_remote_state.bootstrap[0].outputs.boundary_policy_arn["AccountBoundaryPolicy"]
  standard_tags                         = merge(local.tags, { access-project = "security-auditor", access-team = "security-auditor" })
  trusted_role_name                     = "security-auditor"
  trusted_role_policy_arns              = ["arn:aws:iam::aws:policy/SecurityAudit", "arn:aws:iam::aws:policy/job-function/ViewOnlyAccess", "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-prevent-data-exfiltration-policy"]
  saml_provider_id                      = data.terraform_remote_state.bootstrap[0].outputs.saml_provider_id
}
```
**Step 4: Declare all the local variables in the locals block.**

**locals.tf**

> **Notes:**
>  1. For devops local block, add all memember of DevOps team to this list. This allows DevOps to assume 'readonly' role.


```
locals {
  devops = [
    "anmichel.rodriguez@annalect.com",
    "gadupu.kumar@annalect.com",
    "ayomide.oribamise@annalect.com",
    "divakar.matteddula@annalect.com",
    "gadupu.kumar@annalect.com",
    "mocktar.tairou@annalect.com",
    "trenton.ornelas@annalect.com",
    "vemula.kishore@annalect.com",
    "shashikant.kuwar@annalect.com",
    "balaji.kalahasthi@annalect.com",
    "vamshikrishna.korivi@annalect.com",
    "kodari.venkateswarlu@annalect.com",
    "shibir.gargaravindra@annalect.com",
    "vamshikrishna.korivi@annalect.com",
  ]
}
```

**Step 5: Update `outputs.tf`.**

```tf
output "global_okta_roles" {
  description = "Okta IAM Role ARNs"
  value = [
    module.readonly.trusted_iam_role_arn,
    module.billing.trusted_iam_role_arn,
    module.security_auditor.trusted_iam_role_arn,
  ]
}


# output "regional_okta_roles" {
#   description = "Okta IAM Role ARNs"
#   value = [
#     module.onepd_appteam.trusted_iam_role_arn
#     module.auds_datateam.trusted_iam_role_arn,
#     module.auds_project_managers.trusted_iam_role_arn,
#   ]
# }

```

**Step 6: Update `terraform.tfvars`.**

Update values for : `Owner`, `Agency`, `Client`, `costcenter`, `access-team`, and `access-project`.

```tf
common_tags = {
  ##... omitted

  Owner               = "DevOps"                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-infra" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-infra" as default for infra projects.
  access-team         = "devops"                  # Specify 'access-team', else set as devops.
  access-project      = "infra"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Initialize Terraform - Execute terraform commands with 'make' to initialize the directory and to format, validate the terraform configuration.**

Note: Before executing terraform commands, make sure the following booleans are set to 'true' in main.tf to retrieve remote state data from Terraform backend.

```tf
# Make sure following boolean are turned on to activate ‘terraform_remote_state’ data source configuration.
use_secure_baseline_remote_state = false
use_vpc_remote_state             = true
use_sg_remote_state              = true
use_bootstrap_remote_state       = true
use_iam_policy_remote_state      = false
```

Similarly to the previous steps, replace the `role_arn` in `backend.hcl` and the `assume_role_arn` in `trusting_account` under `terraform.tfvars`.


xecute below commands:

```bash
$ make ensure_pre_commit
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 8: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch to master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.

 <a href="#top">Back to top</a>

---

## 13 Create and upload ssh keys for EC2 and EMR resources 

Upload the public and private keys for the AWS EC2 and EMR resources for the appteam/datateams to use.

**Step 1: Generate the two keys via the AWS management console.**

Sign in via the AWS management console and assume the `role/accountonboardingadmin` for the account you want to onboard. 

On the left side of the EC2 page, under `Network & Security`, select the `Key Pairs` section to navigate to the account's stored keypairs.

Click `Create key pair` and create a RSA key in the `.pem` format. The naming convention should follow as such:

```
EC2 Key Format: "${var.account_attr}-${var.region_abbr}-infra-ec2-key"

// ex: "ann31prod-aue1-infra-ec2-key"

EMR Key Format: "${var.account_attr}-${var.region_abbr}-emr-cluster-key"

// ex: "ann32auds-aue1-emr-cluster-key"
```

Repeat this to create a keypair for EC2 and EMR.

**Step 2: Upload the two keys to the ann01tio infra bucket.**

Sign in to the ann01tio management console with the role `ann01-tioprod-sysadmin`.

Navigate to the [s3-bucket](https://s3.console.aws.amazon.com/s3/buckets/Technical_Infrastructure?region=us-east-1&prefix=Keys/AWS_EC2_Keys/&showversions=false) and upload the private EC2 and EMR keys for other DevOps members to use in an emergency.

```
NOTE: If you get a permission failed error, contact Shashi/Shibir to get access to the s3 path for infrastructure related keys.
```

---

## 14. Create Infra management S3 Buckets in each allowed region

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit` command from the root of the repo.


**Step 1: Create a directory for the new configuration under the root of the account repo.**

Repeat this entire section for each enabled region for the account.


To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1'.

```bash
$ mkdir -p aue1/s3buckets/projects/infra
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```bash
$ cd aue1/s3buckets/projects/infra
$ ${pylect-infra-terraform update}
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

```tf
locals {
  infra_buckets = [
    "${var.account_attr}-${var.region_abbr}-build-artifacts",
    "${var.account_attr}-${var.region_abbr}-infra-backups",
    "${var.account_attr}-${var.region_abbr}-infra-athena-query-results",
    "${var.account_attr}-${var.region_abbr}-infra-logs",
    "${var.account_attr}-${var.region_abbr}-flowlogs-bucket",
    "aws-logs-${local.aws_account_id}-${var.region}",
    "${var.account_attr}-${var.region_abbr}-emrstudio-workspace-storage",
  ]
}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

**buckets.tf**

Create the s3 buckets, pulling from the `locals.tf` for each bucket name.

```tf
module "infra_buckets" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/s3-bucket?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  for_each = toset(local.infra_buckets)

  bucket = each.key

  # S3 Server Logging, Inventory  and Analytics
  logging_bucket      = local.logging_bucket
  analytics_bucket    = local.analytics_bucket
  inventory_bucket    = local.inventory_bucket
  enable_s3_inventory = true
  enable_s3_analytics = true

  # Tags
  tags = merge(local.standard_tags_map["s3"], {
    Owner                 = "devops@annalect.com"
    Environment           = var.common_tags["Environment"]
    Client                = var.common_tags["Client"]
    Agency                = var.common_tags["Agency"]
    costcenter            = var.common_tags["costcenter"] # <agency-client-project> format
    access-project        = var.common_tags["access-project"]
    access-team           = var.common_tags["access-team"]
    "Data Classification" = var.common_tags["Data_Classification"]
  })

  # Bucket Policy
  bucket_policies = []

  # Lifecycle Rules
  lifecycle_rule = [
    {
      id                                     = "Delete old incomplete multi-part uploads"
      enabled                                = true
      abort_incomplete_multipart_upload_days = 7
    },
    {
      id      = "artifacts-lifecycle-transition"
      enabled = true
      filter  = {}
      prefix  = ""
      filter  = {}
      transition = [
        {
          days          = 30
          storage_class = "STANDARD_IA"
        },
        {
          days          = 60
          storage_class = "ONEZONE_IA"
        }
      ]
      expiration = {
        days = 120
      }
      # noncurrent_version_expiration = {}
  }]
}
```

Confirm all booleans for remote state match the following, in `main.tf`:

```tf
locals {
  ##...

  # Turn on and off to retrieve remote state from backend
  use_secure_baseline_remote_state = true
  use_vpc_remote_state             = false
  use_sg_remote_state              = false
  use_bootstrap_remote_state       = false
  use_iam_policy_remote_state      = false
}
```

**Step 5: Update `output.tf`**

```tf
# Add additional outputs below
output "buckets" {
  description = "Bucket Details"
  value       = module.infra_buckets[*]
}
```

**Step 6: Update `terraform.tfvars`**

Update the required tags: 

* `Owner`
* `Agency`
* `Client`
* `costcenter`
* `access-team`
* `access-project`
* `Data_Classification`

```tf
common_tags = {
  ##... omitted

  Owner               = "DevOps"                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-infra" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-infra" as default for infra projects.
  access-team         = "devops"                  # Specify 'access-team', else set as devops.
  access-project      = "infra"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```tf
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```tf
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```
> **NOTE:**  Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.

**Step 8: Apply the configuration, and push the feature branch**

Execute below commands:

```bash
$ make ensure_pre_commit
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 9: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch to master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.

 <a href="#top">Back to top</a>
 
---

## 15. Deploy Management SNS topics in each allowed region

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit` command from the root of the repo.
4. Follows the steps below to create the resources.

**Step 1: Create a directory for the new configuration under the root of the account repo.**

Repeat this entire section for each enabled region in the account.

To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1' at the root level of the directory.

```bash
$ mkdir -p aue1/sns-topic
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```bash
$ cd aue1/sns-topic
$ ${pylect-infra-terraform update}
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

> **Notes:**
>  1. Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required.
>  2. If no locals need to be added, skip this step.

```tf
# Additional locals
locals {}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

### 15.1 Create `RDS` SNS topic for rds notifications

**rds_events.tf**

```tf
module "rds_events" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/sns-topic?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  # Labels
  name         = "rds-events"
  display_name = "${var.account_attr}-rds-events"
  attributes   = ["sns", "topic"]
  label_order  = ["name", "attributes"]

  sqs_dlq_enabled    = false
  fifo_topic         = false
  fifo_queue_enabled = false

  subscribers = {
    sub1 = {
      protocol               = "email"
      endpoint               = "devops@annalect.com"
      endpoint_auto_confirms = true
      raw_message_delivery   = false
    }
    sub2 = {
      protocol               = "email"
      endpoint               = "shashikant.kuwar@annalect.com"
      endpoint_auto_confirms = true
      raw_message_delivery   = false
    }
  }

  tags = local.standard_tags_map.infra

  # SNS Resource Policy
  sns_policies = [{
    sid     = "SNSPublishPermission"
    actions = ["SNS:Publish"]
    effect  = "Allow"
    principals = [{
      type        = "Service"
      identifiers = ["events.amazonaws.com", "events.rds.amazonaws.com", "cloudwatch.amazonaws.com"]
    }]
    condition = [{
      test     = "StringEquals"
      variable = "AWS:SourceAccount"
      values   = concat([local.aws_account_id])
    }]
  }]

}
```

### 15.2 Create a `RedShift` SNS topic for redshift events

**redshift_events.tf**

```tf
module "redshift_events" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/sns-topic?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  # Labels
  name         = "redshift-events"
  display_name = "${var.account_attr}-redshift-events"
  attributes   = ["sns", "topic"]
  label_order  = ["name", "attributes"]

  sqs_dlq_enabled    = false
  fifo_topic         = false
  fifo_queue_enabled = false

  subscribers = {
    sub1 = {
      protocol               = "email"
      endpoint               = "devops@annalect.com"
      endpoint_auto_confirms = true
      raw_message_delivery   = false
    }
    sub2 = {
      protocol               = "email"
      endpoint               = "shashikant.kuwar@annalect.com"
      endpoint_auto_confirms = true
      raw_message_delivery   = false
    }
  }

  tags = local.standard_tags_map.infra

  # SNS Resource Policy
  sns_policies = [{
    sid     = "SNSPublishPermission"
    actions = ["SNS:Publish"]
    effect  = "Allow"
    principals = [{
      type        = "Service"
      identifiers = ["events.amazonaws.com", "redshift.amazonaws.com", "cloudwatch.amazonaws.com"]
    }]
    condition = [{
      test     = "StringEquals"
      variable = "AWS:SourceAccount"
      values   = concat([local.aws_account_id])
    }]
  }]

}
```

### 15.3 Create the `SUPPORT` SNS topic to alert Annalect DevOps

**support.tf**

```tf
module "devops_support" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/sns-topic?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  # Labels
  name         = "${var.account_attr}-support"
  display_name = "${var.account_attr}-support"
  access_team  = local.access_team
  attributes   = ["annalect"]
  label_order  = ["attributes", "access_team"]

  kms_master_key_id = "alias/ann/sns"

  subscribers = {
    sub1 = {
      protocol               = "email"
      endpoint               = "devops@annalect.com"
      endpoint_auto_confirms = true
      raw_message_delivery   = false
    }
  }

  tags = local.standard_tags_map.infra
}
```

**main.tf**

Confirm all booleans for remote state match the following, in `main.tf`:

```tf
locals {
  ##...

  # Turn on and off to retrieve remote state from backend
  use_secure_baseline_remote_state = false
  use_vpc_remote_state             = false
  use_sg_remote_state              = false
  use_bootstrap_remote_state       = false
  use_iam_policy_remote_state      = false
}
```

**Step 5: Update Outputs**

Verify all created sns topics are available to other modules.

```tf
# Add additional outputs below
output "sns_topic" {
  description = "SNS Topic"
  value = {
    (module.devops_support.sns_topic_name)  = { arn = module.devops_support.sns_topic_arn, subscriptions : module.devops_support.aws_sns_topic_subscriptions }
    (module.redshift_events.sns_topic_name) = { arn = module.redshift_events.sns_topic_arn, subscriptions : module.redshift_events.aws_sns_topic_subscriptions }
    (module.rds_events.sns_topic_name)      = { arn = module.rds_events.sns_topic_arn, subscriptions : module.rds_events.aws_sns_topic_subscriptions }
  }
}
```

**Step 6: Update `terraform.tfvars`**

Update the required tags: 

* `Owner`
* `Agency`
* `Client`
* `costcenter`
* `access-team`
* `access-project`

```tf
common_tags = {
  ##... omitted

  Owner               = "DevOps"                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-infra" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-infra" as default for infra projects.
  access-team         = "devops"                  # Specify 'access-team', else set as devops.
  access-project      = "infra"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```tf
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```tf
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```
> **NOTE:**  Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.

**Step 8: Apply the configuration, and push the feature branch**
Execute below commands:

```bash
$ make ensure_pre_commit
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 9: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch to master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.


 <a href="#top">Back to top</a>
 
---

## 16. Deploy standard KMS CMKs in each allowed region

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit` command from the root of the repo.
4. Follows the steps below to create the resources.

**Step 1: Create a directory for the new configuration under the root of the account repo.**

Repeat this entire section for each enabled region for the account.

To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1' at the root level of the directory.

```bash
$ mkdir -p aue1/kms
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```bash
$ cd aue1/kms
$ ${pylect-infra-terraform update}
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

> **Notes:**
>  1. Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required.
>  2. If no locals need to be added, skip this step.

```tf
# Additional locals
locals {}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

Create the supporting modules needed to setup Annalect's kms keys.

**rds.tf**

```tf
module "rds" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/kms?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  # Labels
  name           = "rds"
  access_project = "infra"
  tags           = merge(local.standard_tags_map["infra"], { Project = "annalect-internal-infra" })

  description             = "Infra RDS KMS CMK key"
  deletion_window_in_days = 7
  enable_key_rotation     = true
  alias                   = "ann/rds"

  # Key Policy (default=[])
  key_policy = [
    {
      sid       = "Allow access through RDS for all principals in the account that are authorized to use RDS"
      effect    = "Allow"
      resources = ["*"]
      actions = [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:CreateGrant",
        "kms:ListGrants",
        "kms:DescribeKey",
      ]
      principals = [{
        type        = "AWS"
        identifiers = ["*"]
      }]
      condition = [{
        test     = "StringEquals"
        variable = "kms:CallerAccount"
        values   = [local.aws_account_id]
        },
        {
          test     = "StringEquals"
          variable = "kms:ViaService"
          values   = ["rds.${var.region}.amazonaws.com"]
      }]
    },
    {
      sid       = "Allow direct access to key metadata to the account"
      effect    = "Allow"
      resources = ["*"]
      actions = [
        "kms:Describe*",
        "kms:Get*",
        "kms:List*",
        "kms:RevokeGrant"
      ]
      principals = [{
        type        = "AWS"
        identifiers = ["arn:aws:iam::${local.aws_account_id}:root"]
      }]
    },

  ]
}

```

**redshift.tf**

```tf
module "redshift" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/kms?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  # Labels
  name           = "redshift"
  access_project = "infra"

  description             = "Redshift KMS CMK key"
  deletion_window_in_days = 7
  enable_key_rotation     = true
  alias                   = "ann/redshift"

  tags = merge(local.standard_tags_map["infra"], { Project = "annalect-internal-infra" })

  # Key Policy (default=[])
  key_policy = [
    {
      sid       = "Allow access through RedShift for all principals in the account that are authorized to use RedShift"
      effect    = "Allow"
      resources = ["*"]
      actions = [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:CreateGrant",
        "kms:DescribeKey",
      ]
      principals = [{
        type        = "AWS"
        identifiers = ["*"]
      }]
      condition = [{
        test     = "StringEquals"
        variable = "kms:CallerAccount"
        values   = [local.aws_account_id]
        },
        {
          test     = "StringEquals"
          variable = "kms:ViaService"
          values   = ["redshift.${var.region}.amazonaws.com"]
      }]
    },
    {
      sid       = "Allow direct access to key metadata to the account"
      effect    = "Allow"
      resources = ["*"]
      actions = [
        "kms:Describe*",
        "kms:Get*",
        "kms:List*",
        "kms:RevokeGrant"
      ]
      principals = [{
        type        = "AWS"
        identifiers = [local.aws_account_id]
      }]
    },
    {
      sid       = "Allow access through RedShift-Serverless for all principals in the account that are authorized to use RedShift-Serverless"
      effect    = "Allow"
      resources = ["*"]

      actions = [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:CreateGrant",
        "kms:DescribeKey"
      ]
      principals = [{
        type        = "AWS"
        identifiers = ["*"]
      }]
      condition = [{
        test     = "StringEquals"
        variable = "kms:ViaService"
        values   = ["redshift-serverless.${var.region}.amazonaws.com"]
        },
        {
          test     = "StringEquals"
          variable = "kms:CallerAccount"
          values   = [local.aws_account_id]
      }]
    },


  ]
}
```

**s3.tf**

```tf
module "s3" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/kms?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  # Labels
  name           = "s3"
  access_project = "infra"
  tags           = merge(local.standard_tags_map["infra"], { Project = "annalect-internal-infra" })

  description             = "Infra S3 KMS CMK key"
  deletion_window_in_days = 7
  enable_key_rotation     = true
  alias                   = "ann/s3"

  # Key Policy (default=[])
  key_policy = [
    {
      sid       = "Allow access through S3 for all principals in the account that are authorized to use S3",
      effect    = "Allow"
      resources = ["*"]
      actions = [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:CreateGrant",
        "kms:DescribeKey"
      ]
      principals = [{
        type        = "AWS"
        identifiers = ["*"]
      }]
      condition = [{
        test     = "StringEquals"
        variable = "kms:CallerAccount"
        values   = [local.aws_account_id]
        },
        {
          test     = "StringEquals"
          variable = "kms:ViaService"
          values   = ["s3.${var.region}.amazonaws.com"]
      }]
    },
    {
      sid       = "Allow direct access to key metadata to the account"
      effect    = "Allow"
      resources = ["*"]
      actions = [
        "kms:Describe*",
        "kms:Get*",
        "kms:List*"
      ]
      principals = [{
        type        = "AWS"
        identifiers = ["arn:aws:iam::${local.aws_account_id}:root"]
      }]
    },
    {
      sid       = "AllowCloudFrontServicePrincipalSSE-KMS"
      effect    = "Allow"
      resources = ["*"]
      actions = [
        "kms:Decrypt",
        "kms:Encrypt",
        "kms:GenerateDataKey*"
      ]
      principals = [{
        type        = "AWS"
        identifiers = ["arn:aws:iam::${local.aws_account_id}:root"]
        },
        {
          type        = "Service"
          identifiers = ["cloudfront.amazonaws.com"]
      }]
    },
  ]
}

```

**dynamodb.tf**

```tf
module "dynamodb" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/kms?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  # Labels
  name           = "dynamodb"
  access_project = "infra"
  tags           = merge(local.standard_tags_map["infra"], { Project = "annalect-internal-infra" })

  description             = "Infra DynamoDB KMS CMK key"
  deletion_window_in_days = 7
  enable_key_rotation     = true
  alias                   = "ann/dynamodb"

  # Key Policy (default=[])
  key_policy = [
    {
      sid       = "Allow access through Amazon DynamoDB for all principals in the account that are authorized to use Amazon DynamoDB",
      effect    = "Allow"
      resources = ["*"]
      actions = [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:DescribeKey",
        "kms:CreateGrant"
      ]
      principals = [{
        type        = "AWS"
        identifiers = ["*"]
      }]
      condition = [{
        test     = "StringEquals"
        variable = "kms:CallerAccount"
        values   = [local.aws_account_id]
        },
        {
          test     = "StringEquals"
          variable = "kms:ViaService"
          values   = ["dynamodb.${var.region}.amazonaws.com"]
      }]
    },
    {
      sid       = "Allow direct access to key metadata to the account"
      effect    = "Allow"
      resources = ["*"]
      actions   = ["kms:Describe*", "kms:Get*", "kms:List*", "kms:RevokeGrant"],
      principals = [{
        type        = "AWS"
        identifiers = ["arn:aws:iam::${local.aws_account_id}:root"]
      }]
    },
    {
      sid       = "Allow DynamoDB Service with service principal name dynamodb.amazonaws.com to describe the key directly"
      effect    = "Allow"
      resources = ["*"]
      actions   = ["kms:Describe*", "kms:Get*", "kms:List*"]
      principals = [{
        type        = "Service"
        identifiers = ["dynamodb.amazonaws.com"]
      }]
    },
  ]
}

```

**Step 5: Update standard files**

Ensure all customer managed keys are available to other modules.

> **Note:**
>  You can add project specific CMKs for each resources if required.

**outputs.tf**

```tf
# Add additional outputs below
output "kms" {
  description = "CMKs"
  value = [
    # {
    # name  = module.pvtdev.alias_name # optional
    # arn   = module.pvtdev.key_arn # optional
    # id    = module.pvtdev.key_id # optional 
    # alias = module.pvtdev.alias_arn # optional
    # },
    {
      name  = module.s3.alias_name
      arn   = module.s3.key_arn
      id    = module.s3.key_id
      alias = module.s3.alias_arn
    },
    {
      name  = module.dynamodb.alias_name
      arn   = module.dynamodb.key_arn
      id    = module.dynamodb.key_id
      alias = module.dynamodb.alias_arn
    },
    {
      name  = module.rds.alias_name
      arn   = module.rds.key_arn
      id    = module.rds.key_id
      alias = module.rds.alias_arn
    },
    {
      name  = module.redshift.alias_name
      arn   = module.redshift.key_arn
      id    = module.redshift.key_id
      alias = module.redshift.alias_arn
  }]
}
```

**main.tf**

Confirm all booleans for remote state match the following, in `main.tf`:

```tf
locals {
  ##...

  # Turn on and off to retrieve remote state from backend
  use_secure_baseline_remote_state = false
  use_vpc_remote_state             = false
  use_sg_remote_state              = false
  use_bootstrap_remote_state       = false
  use_iam_policy_remote_state      = false
}
```

**Step 6: Update `terraform.tfvars`**

Update the required tags: 

* `Owner`
* `Agency`
* `Client`
* `costcenter`
* `access-team`
* `access-project`

```tf
common_tags = {
  ##... omitted

  Owner               = "DevOps"                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-infra" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-infra" as default for infra projects.
  access-team         = "devops"                  # Specify 'access-team', else set as devops.
  access-project      = "infra"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```tf
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```tf
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```
> **NOTE:**  Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.

**Step 8: Apply the configuration, and push the feature branch**
Execute below commands:

```bash
$ make ensure_pre_commit
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 9: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch to master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.

 <a href="#top">Back to top</a>
 
---

## 17. Deploy Cloud Intelligence Dashboards on AWS QuickSight

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit` command from the root of the repo.
4. Follows the steps below to create the resources.

**Step 1: Create a directory for the new configuration under the root of the account repo.**

To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1' at the root level of the directory.

```bash
$ mkdir -p aue1/env-mgmt/services/cudos
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```bash
$ cd aue1/env-mgmt/services/cudos
$ ${pylect-infra-terraform update}
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

> **Notes:**
>  1. Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required.
>  2. If no locals need to be added, skip this step.

```tf
locals {
  governance_cudos_account_id = var.trusted_account["id"]
  governance_cudos_s3_region  = "us-east-1"
  governance_cudos_s3_bucket  = "annalect-billing-cost-usage-reports"
  governance_account          = false
  use_existing_s3_bucket      = false
}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

Prepare the AWS account to interface with the cudos dashboard.

**cudos.tf**

```tf
module "cur" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/aws-cudos?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  governance_account           = local.governance_account
  use_existing_s3_bucket       = local.use_existing_s3_bucket
  governance_cudos_kms_key_arn = "arn:aws:kms:us-east-1:${data.aws_caller_identity.current.account_id}:key/bab6bb05-06fd-41cb-9974-46a60bd02102"
  governance_cudos_s3_bucket   = local.governance_cudos_s3_bucket
  governance_cudos_s3_region   = local.governance_cudos_s3_region
  governance_cudos_account_id  = local.governance_cudos_account_id

  s3_kms_key_alias       = "alias/ann/cudos-member"
  member_cudos_s3_region = var.region
  member_cudos_s3_bucket = "${var.account_attr}-billing-cur"

  # Glue/Athena
  s3_bucket_report_prefix     = var.account_attr
  report_name                 = "cur-data"
  force_destroy               = true
  report_frequency            = "HOURLY"
  report_additional_artifacts = ["ATHENA"]
  report_format               = "Parquet"
  report_compression          = "Parquet"
  report_versioning           = "OVERWRITE_REPORT"

  logging_bucket = local.logging_bucket
}
```
**main.tf**

Confirm all booleans for remote state match the following, in `main.tf`:

```tf
locals {
  ##...

  # Turn on and off to retrieve remote state from backend
  use_secure_baseline_remote_state = true
  use_vpc_remote_state             = false
  use_sg_remote_state              = false
  use_bootstrap_remote_state       = true 
  use_iam_policy_remote_state      = false
}
```

**terraform.tfvars**
**Step 5: Update `terraform.tfvars`**

Update the required tags: 

* `Owner`
* `Agency`
* `Client`
* `costcenter`
* `access-team`
* `access-project`

```tf
common_tags = {
  ##... omitted

  Owner               = "DevOps"                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-infra" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-infra" as default for infra projects.
  access-team         = "devops"                  # Specify 'access-team', else set as devops.
  access-project      = "infra"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 6: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```tf
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```tf
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```
> **NOTE:**  Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.

**Step 7: Apply the configuration, and push the feature branch**
Execute below commands:

```bash
$ make ensure_pre_commit
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 8: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch to master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.

 <a href="#top">Back to top</a>
 
---

## 18. Build temporary EC2 instances in a VPC and validate AD, DNS and VPN connectivity

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit` command from the root of the repo.
4. Add new AWS account number to list of permitted accounts on ann01tio:
  
    4.0.  You will create a new branch for each folder changed (2 PR's).

    4.1.  Create a feature branch from terraform_live's master branch.
    
    4.2.  Navigate to `terraform_live/aws/accounts/ann01-tioprod/aue1/env-mgmt/backup/ami`.
    
    4.3.  Append the new account number to the `cross_account_identifiers` block in `variables.tf`.

    4.4.   Navigate to `terraform_live/aws/accounts/ann01-tioprod/aue1/env-mgmt/kms`. 

    4.5.  Append the new account number to the `cross_account_identifiers` block in `variables.tf`.

    4.6.  **Run `make test` before creating the PRs, this should prevent the pipeline from failing.**

    4.7.   Create a PR for the new feature branch and merge it into master branch. The PR will need to be approved, so make sure to notify DevOps asap before running any make commands.

5. Upload the public ec2 key to your AWS account via the console (EC2 - keypairs). See module "ec2" key_name for the naming structure. Upload the private key to the ann01tio-prod `Technical_Infrastructure` bucket. The key name should end in `.pem`. The s3 path should look similar to:

    `s3:\\Technical_Infrastructure/Keys/EC2_Keys`


**Step 1: Create a directory for the new configuration under the root of the account repo.**

Repeat this entire section for each enabled region for the account.

To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1' at the root level of the directory.

```bash
$ mkdir -p aue1/env-dev/compute-tier/ec2/ec2-demo-golden-ami-test
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```bash
$ cd aue1/env-dev/compute-tier/ec2/ec2-demo-golden-ami-test
$ ${pylect-infra-terraform update}
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

Add additional variables as local block to locals.tf and replace the local variable value with actual value if required.

**locals.tf**

> **Notes:** 
> Replace the public_key with one generated on your local machine. 
> Make sure to upload the private key to the ann01-tio `s3/Technical_Infrastructure/Keys`. Affix `.pem` to the end of the private key before uploading it.
> Update the provider to use the correct region, if necessary.
> Run `make destroy` to terminate the EC2 instance once connectivity confirmed.


```tf
data "aws_ami" "amd" {
  provider    = aws.aue1
  most_recent = true
  owners      = ["661095214357"]

  filter {
    name   = "name"
    values = ["ann-golden-ami-amd-*"]
  }
}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

Create a test ec2 instance in the aue1 region.

**ec2.tf**

```tf
module "ec2" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/ec2?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  # Labels
  name           = "onboarding-test-instance"
  access_project = local.access_project
  environment    = var.environment
  label_order    = ["account_attr", "environment", "access_project", "name"]

  ami           = data.aws_ami.amd.id
  subnet_id     = data.terraform_remote_state.vpc[0].outputs.private_az_subnet_ids["dev"][0]
  instance_type = "t2.micro"
  key_name      = "${var.account_attr}-${var.region_abbr}-infra-ec2-key" # create EC2 key following this naming convention
  #termination_protection = false
  vpc_security_group_ids = [
    data.terraform_remote_state.sg[0].outputs.security_groups.main.web["dev"],
    data.terraform_remote_state.sg[0].outputs.security_groups.main.sgadmin["mixed"]
  ]
  associate_public_ip_address = false
  tags                        = merge(local.standard_tags_map["ec2"], { monitoring = "none", tower = "unixub" })
  volume_tags                 = merge(local.standard_tags_map["ebs"], { Backup = true })
  user_data                   = templatefile("${path.module}/user_data_ec2.yaml", { APP_NAME = "poc", SVC_NAME = "web", ENV_NAME = "dev", ad_user = "annalectww.ad.joiner" })
  root_block_device           = [{ delete_on_termination = "true", encrypted = "true", volume_size = "30", volume_type = "gp3" }]
  #iam_role_policy_attachment_arn = ["arn:aws:iam::${local.aws_account_id}:policy/admin/${var.account_attr}-infra-ec2-automation-policy"]
  permissions_boundary_arn    = data.terraform_remote_state.bootstrap[0].outputs.boundary_policy_arn["AccountBoundaryPolicy"]
  restart_on_system_failure   = true
  restart_on_instance_failure = true
}
```

**user_data_ec2.yaml**

```tf
#cloud-config
# Ref:https://cloudinit.readthedocs.io/en/latest/topics/examples.html

# Install system updates with cloud-init
apt_update: true
apt_upgrade: true

# Install additional packages on first boot
package_update: true
packages:
  - jq

# runcmd only runs during the first boot
# runcmd contains a list of either lists or a stringruncmd:
runcmd:

# Hostname Parameters
 - APP_NAME=${APP_NAME}
 - SVC_NAME=${SVC_NAME}
 - ENV_NAME=${ENV_NAME}

# SSM Agent Update
 - sudo snap refresh amazon-ssm-agent

# CloudWatch Agent Installation
#  - sudo aws s3 --region us-east-1 cp s3://Automationscripts/Hardening_Ubuntu/ubuntu20.04/installcwagent.sh /root/hardening/installcwagent.sh
#  - sudo sh /root/hardening/installcwagent.sh

# # SentinelOne Agent Installation
#  - curl -sSL https://s3.amazonaws.com/omc-cloud/OS-Repo/Latest_SentinelOne_Installer | bash

# # FireEye Agent Installation
#  - curl -sSL https://s3.amazonaws.com/omc-cloud/OS-Repo/Latest_FireEye_Installer | bash

# # Outpost24 Agent Installation
#  - curl -sSL https://s3.amazonaws.com/omc-cloud/OS-Repo/Latest_Outpost24_Installer | bash

# # Join machine to domain
#  - username=${ad_user}
#  - sudo sh /root/hardening/host_domain_join

# examine the cloud-init directives output log file at /var/log/cloud-init-output.log
```

**main.tf**

Confirm all booleans for remote state match the following, in `main.tf`:

```tf
locals {
  ##...

  # Turn on and off to retrieve remote state from backend
  use_secure_baseline_remote_state = false
  use_vpc_remote_state             = true
  use_sg_remote_state              = true
  use_bootstrap_remote_state       = true
  use_iam_policy_remote_state      = false
}
```

**Step 5: Update Outputs**

Ensure all instance information are available to other modules.

**outputs.tf**

```tf
# Add additional outputs below
output "ec2_values" {
  description = "EC2 Instance Output Values"
  value = {
    "ec2_id"                           = module.ec2.id,
    "ec2_arn"                          = module.ec2.arn,
    "ec2_instance_state"               = module.ec2.instance_state,
    "ec2_primary_network_interface_id" = module.ec2.primary_network_interface_id,
    "ec2_private_dns"                  = module.ec2.private_dns,
    "ec2_public_dns"                   = module.ec2.public_dns,
    "ec2_public_ip"                    = module.ec2.public_ip
  }
}
```

**Step 6: Update `terraform.tfvars`**

Update the required tags: 

* `Owner`
* `Agency`
* `Client`
* `costcenter`
* `access-team`
* `access-project`
* `Data_Classification`

```tf
common_tags = {
  ##... omitted

  Owner               = "DevOps"                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-infra" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-infra" as default for infra projects.
  access-team         = "devops"                  # Specify 'access-team', else set as devops.
  access-project      = "infra"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```tf
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```tf
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```
> **NOTE:**  Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.

**Step 8: Apply the configuration, and push the feature branch**
Execute below commands:

```bash
$ make ensure_pre_commit
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 9: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch to master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.

 <a href="#top">Back to top</a>
 
---

## 19 Deploy infrastructure related lambda functions

**19.1. Deploy `Cloudwatch to Loggly` Lambda Function**

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit` command from the root of the repo.
4. Follows the steps below to create the resources.

**Step 1: Create a directory for the new configuration under the root of the account repo.**

Repeat this entire section for each enabled region for the account.

To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1' at the root level of the directory.

```bash
$ mkdir -p aue1/env-mgmt/lambda/cloudwatch2loggly
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```bash
$ cd aue1/env-mgmt/lambda/cloudwatch2loggly
$ ${pylect-infra-terraform update}
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

> **Notes:**
>  1. Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required.
>  2. If no locals need to be added, skip this step.

```tf
locals {
  s3_object_tags = merge(
    {
      Environment    = var.common_tags["Environment"]
      Client         = var.common_tags["Client"]
      Agency         = var.common_tags["Agency"]
      costcenter     = var.common_tags["costcenter"]
      access-project = var.common_tags["access-project"]
      access-team    = var.common_tags["access-team"]
  })
}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

Create the lambda function to link cloudwatch and loggly.

**lambda_function.tf**


```tf
module "lambda_function" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/lambda?ref=tags/v3.2.120"
  providers = { aws = aws.aue1 }

  description = "Pushes logs from Cloudwatch to Loggly"
  handler     = "index.handler"
  runtime     = "nodejs14.x"

  memory_size = 128
  publish     = true # publish new version if code updated
  timeout     = 30

  # Aliases
  alias_name                      = "release"
  refresh_alias                   = false #map alias to new published version
  create_alias                    = true
  create_version_allowed_triggers = false

  role_permissions_boundary = data.terraform_remote_state.bootstrap[0].outputs.boundary_policy_arn["AccountBoundaryPolicy"]

  source_path = "./context"

  store_on_s3 = true
  s3_bucket   = "${var.account_attr}-${var.region_abbr}-build-artifacts"
  hash_extra  = "forcing commit"

  # Labels
  name                = "lambda"
  environment         = var.environment
  access_project      = "infra"
  attributes          = [var.environment, "cloudwatch2loggly"]
  label_order         = ["access_project", "attributes"]
  s3_object_tags_only = true
  s3_object_tags      = local.s3_object_tags

  # Additional policies
  attach_policy_statements = true
  policy_statements = {
    FullLogsS3Access = {
      effect    = "Allow",
      actions   = ["s3:GetObject"],
      resources = ["arn:aws:s3:::${var.account_attr}-${var.region_abbr}-infra-logs/*"]
    },
    SecretsAccessLoggly = {
      effect  = "Allow",
      actions = ["secretsmanager:GetSecretValue"],
      resources = [
        "arn:aws:secretsmanager:${var.region}:${var.trusted_account["id"]}:secret:mgmt-secrets-*",
      ]
    },
    KMSDecryptSecret = {
      effect  = "Allow",
      actions = ["kms:Decrypt"],
      resources = [
        "arn:aws:kms:${var.region}:${var.trusted_account["id"]}:key/*",
      ],
      condition = [{
        test     = "ForAnyValue:StringLike"
        variable = "kms:ResourceAliases"
        values   = ["alias/ann/infra-shared-mrk"]
      }]
    },
  }

  environment_variables = {
    logglyHostName = "logs-01.loggly.com"
  }

  layers = ["arn:aws:lambda:us-east-1:${var.trusted_account["id"]}:layer:ann-infra-layer-cloudwatch2loggly:32"]
}

```

**main.tf**

Confirm all booleans for remote state match the following, in `main.tf`:

```tf
locals {
  ##...

  # Turn on and off to retrieve remote state from backend
  use_secure_baseline_remote_state = false
  use_vpc_remote_state             = false
  use_sg_remote_state              = false
  use_bootstrap_remote_state       = true
  use_iam_policy_remote_state      = false
}
```

**context/index.js**

Create a folder named context, and place `index.js` inside. 

`index.js` contents:

```js
'use strict';

/*
 *  To encrypt your secrets use the following steps:
 *
 *  1. Create an SSM Secure String /loggly/customer-token with Loggly's customer-token
*/

const cloudwatch2loggly = require('cloudwatch2loggly');

// entry point
exports.handler = (event, context, callback) => {
  cloudwatch2loggly.parse_and_post_to_loggly(event, context, callback);
};
```


**Step 5: Update Outputs**

Ensure all lambda information are available to other modules.

**outputs.tf**

```hcl
# Add additional outputs below
output "lambda_function" {
  description = "Lambda Function"
  value = {
    name       = module.lambda_function.lambda_function_name
    arn        = module.lambda_function.lambda_function_arn
    invoke_arn = module.lambda_function.lambda_function_invoke_arn
    # alias_arn  = module.lambda_function.lambda_alias_arn
    role = module.lambda_function.lambda_role_arn
  }
}
```

**Step 6: Update `terraform.tfvars`**

Update the required tags: 

* `Owner`
* `Agency`
* `Client`
* `costcenter`
* `access-team`
* `access-project`

```tf
common_tags = {
  ##... omitted

  Owner               = "DevOps"                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-infra" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-infra" as default for infra projects.
  access-team         = "devops"                  # Specify 'access-team', else set as devops.
  access-project      = "infra"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```tf
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```tf
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```
> **NOTE:** Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.

**Step 8: Apply the configuration, and push the feature branch**
Execute below commands:

```bash
$ make ensure_pre_commit
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 9: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch to master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.

**19.2. Deploy `Cloudwatch Log Retention` Lambda Function**

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit` command from the root of the repo.
4. Follows the steps below to create the resources.

**Step 1: Create a directory for the new configuration under the root of the account repo.**

Repeat this entire section for each enabled region for the account.

To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1' at the root level of the directory.

```bash
$ mkdir -p aue1/env-mgmt/lambda/cloudwatch-logs-retention
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```bash
$ cd aue1/env-mgmt/lambda/cloudwatch-logs-retention
$ ${pylect-infra-terraform update}
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

> **Notes:**
>  1. Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required.
>  2. If no locals need to be added, skip this step.

```tf
# Additional locals
locals {
  lambda_deployment_bucket = "${var.account_attr}-aue1-build-artifacts"
}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

Configure the lambda function to determine the cloudwatch retention period.

**lambda_cloudwatch_retention.tf**

```tf
module "lambda_function" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/lambda?ref=v3.2.120"
  providers = { aws = aws.aue1 }

  description = "set-cloudwatch-log-retention-period"
  handler     = "set-cloudwatch-log-retention-period.lambda_handler"
  runtime     = "python3.8"
  publish     = true

  # permissions
  role_permissions_boundary = data.terraform_remote_state.bootstrap[0].outputs.boundary_policy_arn["AccountBoundaryPolicy"]
  role_name                 = "set-cw-log-retention-period-lambda-role"

  source_path = "./context"

  store_on_s3 = true
  s3_bucket   = local.lambda_deployment_bucket


  # Labels
  name           = "lambda"
  environment    = var.environment
  access_project = "infra"
  attributes     = ["set-cloudwatch-log-retention-period"]
  label_order    = ["access_project", "attributes"]
  tags           = local.tags

  s3_object_tags_only = true

  cloudwatch_event_rules = {
    scheduled = {
      name                = "cloudwatch-log-retention-period-lambda-rule"
      description         = "Automate CloudWatch Logs retention period to 30 days"
      type                = "cloudwatch-event"
      schedule_expression = "rate(15 days)" # "cron(05 21 * * ? *)"
      is_enabled          = true
    }
  }

  # Additional policies
  attach_policy_statements = true
  policy_statements = {
    CloudwatchLogs = {
      effect    = "Allow",
      actions   = ["logs:DescribeLogGroups", "logs:PutRetentionPolicy"],
      resources = ["*"]
    },
  }

}
```

**main.tf**

Confirm all booleans for remote state match the following, in `main.tf`:

```tf
locals {
  ##...

  # Turn on and off to retrieve remote state from backend
  use_secure_baseline_remote_state = false
  use_vpc_remote_state             = false
  use_sg_remote_state              = false
  use_bootstrap_remote_state       = true
  use_iam_policy_remote_state      = false
}
```

**context/set-cloudwatch-log-retention-period.py**

Create a folder named context, and place `set-cloudwatch-log-retention-period.py` inside. 

`set-cloudwatch-log-retention-period.py` contents:

```python
import json
import boto3
from pprint import pprint
import time

def lambda_handler(event, context):
    expire_cloudwatch_events()

    return {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }

def expire_cloudwatch_events():
    client = boto3.client('logs')
    client = boto3.client('logs')
    paginator = client.get_paginator('describe_log_groups')

    response_iterator = paginator.paginate()

    for pages in response_iterator:
        for loggroup in pages.get('logGroups'):
            if not loggroup.get('retentionInDays'):
                retentionInDays = 30
                response = client.put_retention_policy(
                    logGroupName=loggroup['logGroupName'],
                    retentionInDays=retentionInDays
                )
                time.sleep(0.1)
                pprint(f"Retention policy of {loggroup['logGroupName']} changed from {loggroup.get('retentionInDays')} to {retentionInDays}")

```


**Step 5: Update Outputs**

Ensure all lambda information are available to other modules.

**outputs.tf**

```hcl
# Add additional outputs below
# Lambda Function
output "lambda" {
  description = "Security Headers with AWS Lambda@Edge"
  value = {
    cloudwatchlog_retention = {
      "lambda_function_arn"                = module.lambda_function.lambda_function_arn
      "lambda_function_name"               = module.lambda_function.lambda_function_name
      "this_lambda_function_qualified_arn" = module.lambda_function.lambda_function_qualified_arn
      "lambda_function_version"            = module.lambda_function.lambda_function_version
    }
  }
}
```

**Step 6: Update `terraform.tfvars`**

Update the required tags: 

* `Owner`
* `Agency`
* `Client`
* `costcenter`
* `access-team`
* `access-project`

```tf
common_tags = {
  ##... omitted

  Owner               = "DevOps"                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-infra" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-infra" as default for infra projects.
  access-team         = "devops"                  # Specify 'access-team', else set as devops.
  access-project      = "infra"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```tf
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```tf
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```
> **NOTE:** Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.

**Step 8: Apply the configuration, and push the feature branch**
Execute below commands:

```bash
$ make ensure_pre_commit
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 9: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch to master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.

 <a href="#top">Back to top</a>
 
---

## 20. Deploy DLM (Data Lifecycle Manager) Module

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit` command from the root of the repo.
4. Follows the steps below to create the resources.

**Step 1: Create a directory for the new configuration under the root of the account repo.**

Repeat this entire section for each enabled region for the account.

To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1' at the root level of the directory.

```bash
$ mkdir -p aue1/env-mgmt/backup/dlm
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```bash
$ cd aue1/env-mgmt/backup/dlm
$ ${pylect-infra-terraform update}
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

> **Notes:**
>  1. Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required.
>  2. If no locals need to be added, skip this step.

```tf
# Additional locals
locals {}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

Set up the monthly and daily lifecycle management for the account data.

**dlm.tf**

```tf
module "dlm_lifecycle_daily" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/data-lifecycle-manager?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  tags = merge(local.standard_tags_map["infra"], {
    Name = "automated-daily-snapshots"

  })
  target_tags = {
    "Backup" = "true"
  }
  dlm_policies = [
    { state       = "ENABLED",
      description = "Daily Automated EBS Snapshot - Backup for 7 days",
      name        = "Automated-Daily-Snapshots",
      times       = "03:00", interval = 24, count = 7,
      copy_tags   = "true"
    }
  ]

  role_permissions_boundary_arn = data.terraform_remote_state.bootstrap[0].outputs.boundary_policy_arn["AccountBoundaryPolicy"]

}

module "dlm_lifecycle_monthly_dev" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/data-lifecycle-manager?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  execution_role_arn = "arn:aws:iam::${local.aws_account_id}:role/${var.account_attr}-${var.region_abbr}-mixed-dlm-role"

  tags = merge(local.standard_tags_map["infra"], {
    Name = "monthly-snapshot-before-patching-linux-dev"

  })
  target_tags = {
    "Patch Group" = "linux-dev"
  }
  dlm_policies = [
    { state       = "DISABLED",
      description = "Monthly Automated EBS Snapshot - Linux Dev",
      name        = "Monthly-Snapshot-before-patching",
      times       = "22:00", interval = 24, count = 3,
    copy_tags = "true" }
  ]
}
```

**main.tf**

Confirm all booleans for remote state match the following, in `main.tf`:

```tf
locals {
  ##...

  # Turn on and off to retrieve remote state from backend
  use_secure_baseline_remote_state = false
  use_vpc_remote_state             = false
  use_sg_remote_state              = false
  use_bootstrap_remote_state       = true
  use_iam_policy_remote_state      = false
}
```

**Step 5: Update Outputs**

Ensure all dlm-related information are available to other modules.

**outputs.tf**

```tf
# Add additional outputs below
output "iam_role_name" {
  description = "Name of the role created"
  value = [
    module.dlm_lifecycle_daily.*.iam_role_name,
    module.dlm_lifecycle_monthly_dev.*.iam_role_name,
  ]
}

output "dlm_id" {
  description = "ID of the policies created in DLM"
  value = [
    module.dlm_lifecycle_daily.*.dlm_id,
    module.dlm_lifecycle_monthly_dev.*.dlm_id,
  ]
}

output "dlm_arn" {
  description = "ARN of the policies created in DLM"
  value = [
    module.dlm_lifecycle_daily.*.dlm_arn,
    module.dlm_lifecycle_monthly_dev.*.dlm_arn,
  ]
}
```

**Step 6: Update `terraform.tfvars`**

Update the required tags: 

* `Owner`
* `Agency`
* `Client`
* `costcenter`
* `access-team`
* `access-project`

```tf
common_tags = {
  ##... omitted

  Owner               = "DevOps"                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-infra" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-infra" as default for infra projects.
  access-team         = "devops"                  # Specify 'access-team', else set as devops.
  access-project      = "infra"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```tf
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```tf
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```
> **NOTE:** Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.

**Step 8: Apply the configuration, and push the feature branch**
Execute below commands:

```bash
$ make ensure_pre_commit
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 9: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch to master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.

 <a href="#top">Back to top</a>
 
---

## 21. Deploy required AWS System Manager Resources for Patching and Maintenance

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit` command from the root of the repo.
4. Follows the steps below to create the resources.

**Step 1: Create a directory for the new configuration under the root of the account repo.**

Repeat this entire section for each enabled region for the account.

To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1' at the root level of the directory.

```bash
$ mkdir -p aue1/env-mgmt/maintenance/ssm
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```bash
$ cd aue1/env-mgmt/maintenance/ssm
$ ${pylect-infra-terraform update}
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

> **Notes:**
>  1. Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required.
>  2. If no locals need to be added, skip this step.

```tf
# Additional locals
locals {}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

Create the patching and maintenance tasks needed for the aws account.

**ssm.tf**

```tf
module "ssm_patch_management" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/ssm-maintenance?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  # maintenance window details
  install_maintenance_window_schedule = "cron(0 30 4 ? * FRI *)"
  tags                                = merge(local.tags_infra, { Patch_Group = "linux-dev" }, { Name = "linux-dev" })

  # register target
  patch_group = ["linux-dev"]

  # register tasks
  task_arn = "AWS-RunPatchBaseline"

  enable_diskcleanup   = true
  task_arn_diskcleanup = "arn:aws:ssm:us-east-1:661095214357:document/ANLCT-Multi-Platform-Generic-Disk-Cleanup"

  service_role_arn = "arn:aws:iam::${local.aws_account_id}:role/service-role/${var.account_attr}-infra-ssm-maintenance-role"

  s3_bucket_name   = "${var.account_attr}-${var.region_abbr}-infra-logs"
  s3_bucket_prefix = "ssm-patching-logs"

  enable_notification_install = true
  notification_arn            = "arn:aws:sns:us-east-1:${local.aws_account_id}:annalect-devops"
  notification_events         = ["Success", "Failed"]

  task_invocation_run_command_parameters = [
    {
      name   = "Operation"
      values = ["Install"]
    },
    {
      name   = "RebootOption"
      values = ["RebootIfNeeded"]
    }
  ]
}
```

**main.tf**

Confirm all booleans for remote state match the following, in `main.tf`:

```tf
locals {
  ##...

  # Turn on and off to retrieve remote state from backend
  use_secure_baseline_remote_state = false
  use_vpc_remote_state             = false
  use_sg_remote_state              = false
  use_bootstrap_remote_state       = false
  use_iam_policy_remote_state      = false
}
```

**Step 5: Update Outputs**

Ensure all patch and maintenance are available to other modules.

**outputs.tf**

```tf
# Add additional outputs below
output "ssm_maintenance_window" {
  description = "Maintenance Window Info"
  value = {
    name : module.ssm_patch_management.maintenance_window_name
    id : module.ssm_patch_management.maintenance_window_id
    target_id : module.ssm_patch_management.maintenance_window_target_id
    service_role_arn : module.ssm_patch_management.service_role_arn
    service_role_name : module.ssm_patch_management.service_role_name
  }
}
```

**Step 6: Update `terraform.tfvars`**

Update the required tags: 

* `Owner`
* `Agency`
* `Client`
* `costcenter`
* `access-team`
* `access-project`

```tf
common_tags = {
  ##... omitted

  Owner               = "DevOps"                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-infra" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-infra" as default for infra projects.
  access-team         = "devops"                  # Specify 'access-team', else set as devops.
  access-project      = "infra"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```tf
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```tf
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```
> **NOTE:** Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.

**Step 8: Apply the configuration, and push the feature branch**
Execute below commands:

```bash
$ make ensure_pre_commit
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 9: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch to master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.

 <a href="#top">Back to top</a>
 
---

## 22. Export the ACM certificate ARN to the Terraform Remote State (Optional)

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit` command from the root of the repo.
4. Ensure the SSL/TLS certificate for required domain is imported into AWS Certificate Manager using AWS Management Console.

Follows the steps below to export the certificate arn into terraform remote state.

**Step 1: Create a directory for the new configuration under the root of the account repo.**

Repeat this entire section for each enabled region for the account.

To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1' at the root level of the directory.

```bash
$ mkdir -p aue1/acm
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```bash
$ cd aue1/acm
$ ${pylect-infra-terraform update}
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

> **Notes:**
>  1. Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required.
>  2. If no locals need to be added, skip this step.

```tf
# Additional locals
locals {}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

**acm.tf**

> **Notes:**
>  1. Ensure the SSL/TLS certificate for required domain is imported into AWS Certificate Manager using AWS Management Console.
>  2. If the certificate is imported into ACM, then uncomment the specific data block for particular domain in order to export the certificate arn into terraform remote state. Otherwise, keep the code as is and proceed to next step.

```tf
# Find a RSA 2048 bit certificate in us-east-1 region
# data "aws_acm_certificate" "annalect_com" {
#   provider    = aws.aue1
#   domain      = "*.annalect.com"
#   key_types   = ["RSA_2048"]
#   most_recent = true
# }

# data "aws_acm_certificate" "oneomg_com" {
#   provider    = aws.aue1
#   domain      = "*.oneomg.com"
#   key_types   = ["RSA_2048"]
#   most_recent = true
# }

# data "aws_acm_certificate" "poc_annalect_com" {
#   provider    = aws.aue1
#   domain      = "*.poc.annalect.com"
#   key_types   = ["RSA_2048"]
#   most_recent = true
# }

# data "aws_acm_certificate" "api_annalect_com" {
#   provider    = aws.aue1
#   domain      = "*.api.annalect.com"
#   key_types   = ["RSA_2048"]
#   most_recent = true
# }


# data "aws_acm_certificate" "annalectww_com" {
#   provider    = aws.aue1
#   domain      = "*.annalectww.com"
#   key_types   = ["RSA_2048"]
#   most_recent = true
# }
```

**main.tf**

Confirm all booleans for remote state match the following, in `main.tf`:

```tf
locals {
  ##...

  # Turn on and off to retrieve remote state from backend
  use_secure_baseline_remote_state = false
  use_vpc_remote_state             = false
  use_sg_remote_state              = false
  use_bootstrap_remote_state       = false
  use_iam_policy_remote_state      = false
}
```

**Step 5: Update Outputs**

Ensure all customer managed keys are available to other modules.

**outputs.tf**

```tf
# Add additional outputs below
output "arn" {
  description = "tls_cert_arn"
  value = {
    cloudfront = {
      annalect_com = try(data.aws_acm_certificate.annalect_com.arn, [])
      # oneomg_com          = try(data.aws_acm_certificate.oneomg_com.arn, [])
      # poc_annalect_com    = try(data.aws_acm_certificate.poc_annalect_com.arn, [])
      # api_annalect_com    = try(data.aws_acm_certificate.api_annalect_com.arn, [])
      # annalectww_com      = try(data.aws_acm_certificate.annalectww_com.arn, [])
    },
    us-east-1 = {
      annalect_com = try(data.aws_acm_certificate.annalect_com.arn, [])
      # oneomg_com          = try(data.aws_acm_certificate.oneomg_com.arn, [])
      # poc_annalect_com    = try(data.aws_acm_certificate.poc_annalect_com.arn, [])
      # api_annalect_com    = try(data.aws_acm_certificate.api_annalect_com.arn, [])
      # annalectww_com      = try(data.aws_acm_certificate.annalectww_com.arn, [])
    },
    # eu-west-1 = {
    #   annalect_com = try(data.aws_acm_certificate.annalect_com_aew1.arn, [])
    # },
    # me-south-1 = {
    #   oneomg_com = try(data.aws_acm_certificate.oneomg_com_ams1.arn, [])
    # }
  }
}
```

**Step 6: Update `terraform.tfvars`**

Update the required tags: 

* `Owner`
* `Agency`
* `Client`
* `costcenter`
* `access-team`
* `access-project`

```tf
common_tags = {
  ##... omitted

  Owner               = "DevOps"                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-infra" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-infra" as default for infra projects.
  access-team         = "devops"                  # Specify 'access-team', else set as devops.
  access-project      = "infra"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```tf
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```tf
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```
> **NOTE:** Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.

**Step 8: Apply the configuration, and push the feature branch**
Execute below commands:

```bash
$ make ensure_pre_commit
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 9: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch to master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.

 <a href="#top">Back to top</a>
 
---

## 23. Deploy federated okta roles for data engineers [DATAMESH]

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit` command from the root of the repo.
4. Follows the steps below to create Annalect managed IAM policies.

**Step 1: Create/modify the files needed for the configuration code.**

Append the following to the bottom of `aue1.tf`. 

If the AWS account has multiple regions, append the file contents to the bottom of another region file (ex: aew1.tf) and adjust the providers as needed.

**aue1.tf**

> **Notes:**
>  1. There are TWO module sources here, especially for datamesh account, one more datateam and another for project managers.
>  2. Replace any mention of `elde2` with the project name (ex: auds, orad, amd, fsd). 


Add project specific okta roles for data engineers and project managers under `global/iam-roles/okta`

```hcl
module "elde2_datateam" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-assumable-roles-with-saml?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  create_trusted_role = true

  trusted_role_arns                     = ["arn:aws:iam::${var.trusted_account["id"]}:root"]
  max_session_duration                  = 43200
  trusted_role_permissions_boundary_arn = data.terraform_remote_state.bootstrap[0].outputs.boundary_policy_arn["AccountBoundaryPolicy"]
  standard_tags                         = merge(local.tags, { access-project = "elde2", access-team = "datateam" })
  trusted_role_name                     = "elde2-datateam"
  trusted_role_policy_arns = [
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-aws-us-east-1-region-boundary-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-service-control-policy-scp1",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-service-control-policy-scp2",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-service-control-policy-scp3",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac-iam-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac1-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac2-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac3-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac4-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac5-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac6-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac7-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-airflow-scopedown-policy",
  ]
  saml_provider_id = data.terraform_remote_state.bootstrap[0].outputs.saml_provider_id
}

module "elde2_project_managers" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-assumable-roles-with-saml?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  create_trusted_role = true

  trusted_role_arns                     = ["arn:aws:iam::${var.trusted_account["id"]}:root"]
  max_session_duration                  = 43200
  trusted_role_permissions_boundary_arn = data.terraform_remote_state.bootstrap[0].outputs.boundary_policy_arn["AccountBoundaryPolicy"]
  standard_tags                         = merge(local.tags, { access-project = "elde2", access-team = "pm" })
  trusted_role_name                     = "elde2-project-managers"
  trusted_role_policy_arns = [
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-aws-us-east-1-region-boundary-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-service-control-policy-scp1",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-service-control-policy-scp2",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-service-control-policy-scp3",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac1-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac2-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac3-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac4-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac5-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac6-policy",
    "arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-abac7-policy",
  ]
  saml_provider_id = data.terraform_remote_state.bootstrap[0].outputs.saml_provider_id
}
```

> **Note:**
>  If account is enabled for us-east-1 and eu-west-1 regions, then replace the region boundary policy with below -
>  `"arn:aws:iam::${local.aws_account_id}:policy/admin/${local.account_attr}-infra-aws-aue1-aew1-regions-boundary-policy"`



**Step 2: Update `outputs.tf`.**

Append the following to the bottom of outputs.

```tf
output "datateam_okta_roles" {
  description = "Okta IAM Role ARNs"
  value = [
    module.elde2_datateam.trusted_iam_role_arn,
    module.elde2_project_managers.trusted_iam_role_arn,
  ]
}
```

**Step 3: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```hcl
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

**Step 4: Replace the roles in `terraform.tfvars`**

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```hcl
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```

> **NOTE:** Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.


**Step 5: Apply the configuration, and push the feature branch**
Execute below commands:

```bash
$ make ensure_pre_commit
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 6: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch -> master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.

 <a href="#top">Back to top</a>
 
---

## 24. Add job and service specific managed IAM policies in each region [DATAMESH]

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit` command from the root of the repo.
4. Follows the steps below to create managed IAM policies (service and execution roles) for datamesh project.
5. The below code creates policies for 'elde2' datamesh project (access-project=elde2)

**Step 1: Create a directory for the new configuration under the root of the account repo.**

```bash
# replace project-elde2 with your account project name
$ mkdir -p global/iam-policies/project-elde2
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```bash
$ cd global/iam-policies/project-elde2
$ ${pylect-infra-terraform update}
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required.

```hcl
# Additional locals
locals {}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

> **Notes:**
>  1. Replace any mention of `elde2` with your account name (ex: auds, orad, amd, fsd). 

**execution_role_policies.tf**

Create service and execution role policies for datamesh project (example, elde2).

```hcl
module "execution_role_policies" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-policy?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  enabled = true

  description = "Annalect managed IAM policies for execution roles for runtime job."
  path        = "/service-role/"

  # Labels
  name           = "execution"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["execution", "role", "policy"]
  label_order    = ["account_attr", "region_abbr", "access_project"]
  tags           = merge(local.tags_infra, { protected = "true" })

  create_execution_role_policy = true

  execution_role_policies = {

    emrserverless = {
      emrserverless_s3_data_source = []
      emrserverless_s3_data_target = [
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-datateam",
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-athena",
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-raw",
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-curated"
        ]
      emrserverless_logs_bucket    = []
    }

    lakeformation = {
      lf_passrole_arns = ["arn:aws:iam::${local.aws_account_id}:role/${local.account_attr}-${local.access_project}-lakeformation-workflow-role"]
    },

    mwaa = {
      mwaa_s3_data_target = [
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-datateam",
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-raw",
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-curated"
      ]
      mwaa_s3_data_source = ["${local.account_attr}-${var.region_abbr}-${local.access_project}-mwaa-*"]
      mwaa_deny_buckets = [
        format("%v-%v", local.aws_account_name, "audit-bucket"),
        format("%v-%v", local.aws_account_name, "terraform-state"),
        format("%v-%v", local.aws_account_name, "terraform-state-replica"),
        format("%v-%v", var.account_attr, "billing-cur"),
      ]
      mwaa_kms_key_id            = ["arn:aws:kms:${var.region}:${local.aws_account_id}:key/xxxxxxxx-xxxxx-4b32-8508-74649cc0651a"]
      mwaa_batch_job_definitions = []
      mwaa_environment_name      = ["${local.account_attr}-${local.access_project}-airflow", "${local.access_project}-airflow-*"]
      athena_datalake_regions    = ["us-east-1"]
      athena_workgroup_name      = [local.access_project]
      athena_query_results_bucket = [
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-athena"
      ]
    },

    lambda = {
      account_id                = local.aws_account_id
      lambda_security_group_ids = [data.terraform_remote_state.sg[0].outputs.security_groups.main.lambda["dev"]],
      lambda_subnet_ids         = [data.terraform_remote_state.vpc[0].outputs.private_az_subnet_ids["dev"]],
      vpc_ids                   = [data.terraform_remote_state.vpc[0].outputs.vpcs.main.id]
      lambda_s3_data_target = [
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-datateam",
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-athena",
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-raw",
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-curated"
      ]
      lambda_s3_data_source     = []
      dynamodb_tables_readonly  = []
      dynamodb_tables_readwrite = []
    },

    batch = {
      batch_execution_role_arns = ["arn:aws:iam::${local.aws_account_id}:role/service-role/${local.account_attr}-${local.access_project}-${var.region_abbr}-batch-execution-role"]
      batch_s3_data_source      = []
      batch_s3_data_target = [
         "${local.account_attr}-${var.region_abbr}-${local.access_project}-athena",
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-datateam",
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-raw",
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-curated",
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-mwaa-*"
      ]
      batch_parameter_store_key      = []
      batch_secretsmanager_key       = ["arn:aws:secretsmanager:${var.region}:${data.aws_caller_identity.trusted.account_id}:secret:bitbucket-deployment-key-*"]
      batch_kms_decrypt_key_id       = ["arn:aws:kms:${var.region}:${data.aws_caller_identity.trusted.account_id}:key/mrk-24eaf2d5b53a4387acc4b522463b9afc"]
      batch_passrole_arns            = []
      batch_ecr_pull_repository_arns = []
    },

    bastion_dataconsumer = {
      bastion_consumer_source_buckets         = []
      bastion_consumer_glue_database_readonly = []
    },

    bastion_dataprovider = {
      bastion_provider_source_buckets         = []
      bastion_provider_glue_database_readonly = []
    },
    
    sagemaker = {
      account_id = local.aws_account_id
    },

    s3batchops = {
      s3batchops_target_buckets = [
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-athena",
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-datateam",
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-raw",
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-curated"
      ]
      s3batchops_source_buckets   = []
      s3batchops_manifest_buckets = []
      s3batchops_report_buckets   = []
    },

    glue = {
      glue_s3_data_target = [
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-datateam",
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-raw",
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-curated"
      ]
      glue_s3_data_source     = []
      athena_datalake_regions = ["us-east-1"]
      athena_workgroup_name   = [local.access_project]
      athena_query_results_bucket = [
        "${local.account_attr}-${var.region_abbr}-${local.access_project}-athena"
      ]
    }
  }
}
```

**s3readwrite.tf**

```hcl
  source    = "git::ssh://git@gitlabssh.annalect.com:5000/devops/terraform_aws_modules//modules/iam/iam-policy?ref=v3.2.110"
  providers = { aws = aws.aue1 }

  description = "S3 Read Write Access Policy"

  # Labels
  name           = "s3"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["readwrite-access"]
  label_order    = ["account_attr", "access_project", "region_abbr", "name", "attributes"]
  tags           = local.standard_tags_map.default


  # If you're creating IAM policy for s3 bucket, 'create_s3_policy' must be set 'true'
  create_s3_policy = true

  # Enable access based on 'true' or 'false' boolean values 
  datateam_access = true


  # Allow ReadWrite (with List) access to s3 bucket(s) with or without objectprefix and w/o forwarding slash
  s3_object_readwrite_access = [
    "${local.account_attr}-${var.region_abbr}-${local.access_project}-mwaa-*"
  ]

  # Allow ReadOnly (with List) access to s3 bucket(s) with or without object prefix and w/o forwarding slash
  s3_object_readonly_access = []

  # Allow ListOnly access to whole bucket(s) w/o forwarding slash
  s3_bucket_list_access = []

  # Restrict access to S3 object prefixes (from root of bucket) with /* suffix
  s3_bucket_list_prefix_only = []
  # s3_bucket_list_prefix_only = [
  #  { bucket = "annalect-example-list-prefix3", prefix = ["x/y/z/*", "a/b/c/*"] },
  # ]

  # Deny access to s3 bucket(s) with or without prefix
  s3_object_explicit_deny_access = []
  # s3_object_explicit_deny_access = [
  #  { bucket = "annalect-example-deny1", prefix = ["*"] },
  #  { bucket = "annalect-example-deny2", prefix = ["lz/*"] },
  # ]

  role_names = [
    "${local.account_attr}-${local.access_project}-${var.region_abbr}-airflow-execution-role",
    "${local.account_attr}-${local.access_project}-${var.region_abbr}-batch-job-role",
    "${local.account_attr}-${local.access_project}-${var.region_abbr}-glue-service-role",
    "${local.account_attr}-${local.access_project}-${var.region_abbr}-emr-ec2-role",
    "${local.account_attr}-${local.access_project}-${var.region_abbr}-emrserverless-execution-role"
  ]
}
```

**misc_access.tf**

```hcl
# This policy can be used to grant additional or missing access to project specific resources (not team okta role)
module "misc_access" {
  source    = "git::ssh://git@gitlabssh.annalect.com:5000/devops/terraform_aws_modules//modules/iam/iam-policy?ref=v3.2.108"
  providers = { aws = aws.aue1 }
  enabled   = true

  description = "Additional access to ${local.access_project} resources"

  # Labels
  name           = "misc"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["sns", "access"]
  label_order    = ["account_attr", "access_project", "access_team", "name", "attributes"]
  tags           = local.standard_tags_map.default

  policy_statements = [
    {
      sid = "MiscAccess"
      actions = [
        "account:ListRegions",
      ]
      effect    = "Allow"
      resources = ["*"]
      condition = []
  }]

  role_names = []
}
```

**Step 5: Update `outputs.tf`.**

Append the following to the bottom of outputs.

```hcl
output "execution_role_policies" {
  description = "Annalect managed service role policy ARNs"
  value       = module.execution_role_policies.oarn[*]
}

output "policy_arns" {
  description = "IAM Policy ARNs"
  value = [
    module.misc_access.arn,
    module.s3readwrite.arn
  ]
}
```

**Step 6: Update `terraform.tfvars`.**

Update values for : `Owner`, `Agency`, `Client`, `costcenter`, `access-team`, and `access-project`.

```hcl
common_tags = {
  ##... omitted

  Owner               = "DevOps"                   # Owner/Requester of the resource
  Owner2              = ""                         # Owner/Requester of the resource
  Agency              = "ANN"                      # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                 # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-infra" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-infra" as default for infra projects.
  access-team         = "datateam"                   # Specify 'access-team', else set as devops.
  access-project      = "elde2"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Configure the remote state**

Note: Before executing terraform commands, make sure the following booleans are set to 'true' in main.tf to retrieve remote state data from Terraform backend.

```hcl
# Make sure following boolean are turned on to activate ‘terraform_remote_state’ data source configuration.
use_vpc_remote_state             = true
use_sg_remote_state              = true
use_bootstrap_remote_state       = true
```
**Step 8: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```hcl
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```hcl
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```

> **NOTE:** Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.

**Step 9: Apply the configuration, and push the feature branch**
Execute below commands:

```bash
$ make ensure_pre_commit
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 10: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch -> master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.

 <a href="#top">Back to top</a>
 
---

## 25. Add fine-grained IAM roles for project specific jobs and services in each region [DATAMESH]

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit` command from the root of the repo.
4. Follows the steps below to create managed IAM policies (service and execution roles) for datamesh project.
5. The below code creates policies for 'elde2' datamesh project (access-project=elde2)

**Step 1: Create a directory for the new configuration under the root of the account repo.**

To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1'.

```bash
$ mkdir -p global/iam-roles/project-elde2
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```bash
$ cd global/iam-roles/project-elde2
$ pip3 install git+ssh://git@bitbucket.org/annalect/pylect-infra-terraform@v.0.x --upgrade
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required.

```
# Additional locals
locals {}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

**airflow_mwaa.tf**

Create apache mwaa execution role for datamesh project.

```hcl
module "airflow_mwaa" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-assumable-role?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  create_role = true

  description           = "AWS Airflow Service Role"
  trusted_role_services = ["airflow.amazonaws.com", "airflow-env.amazonaws.com"]

  role_path = "/service-role/"

  # Labels
  name           = "airflow"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["execution-role"]
  label_order    = ["account_attr", "access_project", "region_abbr", "name", "attributes"]


  custom_role_policy_arns = [
    "arn:aws:iam::${local.aws_account_id}:policy/service-role/${local.account_attr}-${var.region_abbr}-${local.access_project}-mwaa-execution-role-policy",
  ]
  role_permissions_boundary_arn = data.terraform_remote_state.bootstrap[0].outputs.boundary_policy_arn["AccountBoundaryPolicy"]
  tags                          = merge(local.standard_tags_map.iam, { protected = "true" })

  # force_detach_policies = true
  # inline_policy_statements = []
}
```
**aurora_postgresql.tf**

Add aurora postgresql s3 import and export roles for datamesh project.

```hcl
module "rds_s3import" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-assumable-role?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  description           = "Allows Import data from Amazon S3 to Aurora PostgreSQL"
  trusted_role_services = ["rds.amazonaws.com"]
  role_path             = "/service-role/"

  # Labels
  name                    = "rds"
  access_project          = local.access_project
  access_team             = local.access_team
  attributes              = ["aurora", "s3import", "service", "role"]
  label_order             = ["account_attr", "access_project", "region_abbr", "name", "attributes"]
  custom_role_policy_arns = []

  role_permissions_boundary_arn = data.terraform_remote_state.bootstrap[0].outputs.boundary_policy_arn["AccountBoundaryPolicy"]
  tags                          = merge(local.standard_tags_map.iam, { protected = "true" })
}

module "rds_s3export" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-assumable-role?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  description           = "Allows Export data from Aurora PostgreSQL to Amazon S3"
  trusted_role_services = ["rds.amazonaws.com"]
  role_path             = "/service-role/"

  # Labels
  name                    = "rds"
  access_project          = local.access_project
  access_team             = local.tags_infra["access-team"]
  attributes              = ["aurora", "s3export", "service", "role"]
  label_order             = ["account_attr", "access_project", "region_abbr", "name", "attributes"]
  custom_role_policy_arns = []

  role_permissions_boundary_arn = data.terraform_remote_state.bootstrap[0].outputs.boundary_policy_arn["AccountBoundaryPolicy"]
  tags                          = merge(local.standard_tags_map.iam, { protected = "true" })
}
```

**batch.tf**

Add batch execution role and job role for datamesh project.

```hcl
module "batch_execution_role" {
  source      = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-assumable-role?ref=tags/v3.2.110"
  providers   = { aws = aws.aue1 }
  create_role = true

  description           = "AWS Batch Execution Role"
  trusted_role_services = ["ecs-tasks.amazonaws.com"]

  role_path = "/service-role/"

  # Labels
  name           = "batch"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["execution-role"]
  label_order    = ["account_attr", "access_project", "region_abbr", "name", "attributes"]

  custom_role_policy_arns       = ["arn:aws:iam::${local.aws_account_id}:policy/service-role/${local.account_attr}-${var.region_abbr}-${local.access_project}-batch-execution-role-policy"]
  role_permissions_boundary_arn = data.terraform_remote_state.bootstrap[0].outputs.boundary_policy_arn["AccountBoundaryPolicy"]
  tags                          = merge(local.standard_tags_map.iam, { protected = "true" })

  # force_detach_policies = true
  # inline_policy_statements = []
}

module "batch_job_role" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-assumable-role?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  create_role = true

  description           = "AWS Batch Job Role"
  trusted_role_services = ["ecs-tasks.amazonaws.com"]

  role_path = "/service-role/"
  # Labels
  name           = "batch"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["job-role"]
  label_order    = ["account_attr", "access_project", "region_abbr", "name", "attributes"]

  custom_role_policy_arns       = ["arn:aws:iam::${local.aws_account_id}:policy/service-role/${local.account_attr}-${var.region_abbr}-${local.access_project}-batch-execution-role-policy"]
  role_permissions_boundary_arn = data.terraform_remote_state.bootstrap[0].outputs.boundary_policy_arn["AccountBoundaryPolicy"]
  tags                          = merge(local.standard_tags_map.iam, { protected = "true" })

  # force_detach_policies = true
  inline_policy_statements = [{
    statements = [{
      sid = "ReadFromOutputAndInputBuckets"
      actions = [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      effect = "Allow",
      resources = [
          "arn:aws:s3:::${local.account_attr}-${var.region_abbr}-${local.access_project}-datateam",
          "arn:aws:s3:::${local.account_attr}-${var.region_abbr}-${local.access_project}-raw",
          "arn:aws:s3:::${local.account_attr}-${var.region_abbr}-${local.access_project}-curated",
          "arn:aws:s3:::${local.account_attr}-${var.region_abbr}-${local.access_project}-mwaa-*",
          "arn:aws:s3:::aws-logs-${local.aws_account_id}-${var.region}",
      ],
      not_actions   = [],
      not_resources = [],
      principals    = [],
      condition     = []
      },
      {
        sid = "WriteToOutputDataBucket"
        actions = [
          "s3:GetObject",
          "s3:PutObject",
          "s3:ListBucket",
          "s3:DeleteObject"
        ],
        effect = "Allow",
        resources = [
          "arn:aws:s3:::${local.account_attr}-${var.region_abbr}-${local.access_project}-datateam/*",
          "arn:aws:s3:::${local.account_attr}-${var.region_abbr}-${local.access_project}-raw/*",
          "arn:aws:s3:::${local.account_attr}-${var.region_abbr}-${local.access_project}-curated/*",
          "arn:aws:s3:::${local.account_attr}-${var.region_abbr}-${local.access_project}-mwaa-*/*",
          "arn:aws:s3:::aws-logs-${local.aws_account_id}-${var.region}/*",
        ],
        not_actions   = [],
        not_resources = [],
        principals    = [],
        condition     = []
    }]
  }]
}
```
**emr.tf**

Add emr cluster and emr-ec2 service roles for datamesh project.


```hcl
module "emr_cluster" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-assumable-role?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  create_role = true

  description           = "EMR Service Role for Heterogeneous Clusters"
  trusted_role_services = ["elasticmapreduce.amazonaws.com"]

  role_path = "/service-role/"

  # Labels
  name           = "emr"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["service-role"]
  label_order    = ["account_attr", "access_project", "name", "attributes"]

  custom_role_policy_arns = ["arn:aws:iam::${local.aws_account_id}:policy/service-role/${local.account_attr}-infra-emr-servicerole-policy"]
  tags = merge(local.standard_tags_map.iam, { protected = "true" })

  # force_detach_policies = true
  # inline_policy_statements = []
}

module "emr_ec2" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-assumable-role?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  create_role             = true
  create_instance_profile = true

  description           = "EC2 IAM Role for EMR Cluster"
  trusted_role_services = ["ec2.amazonaws.com"]

  # Labels
  name           = "emr-ec2"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["role"]
  label_order    = ["account_attr", "access_project", "region_abbr", "name", "attributes"]

  custom_role_policy_arns       = ["arn:aws:iam::${local.aws_account_id}:policy/service-role/${local.account_attr}-${var.region_abbr}-${local.access_project}-emrserverless-execution-role-policy"]
  role_permissions_boundary_arn = data.terraform_remote_state.bootstrap[0].outputs.boundary_policy_arn["AccountBoundaryPolicy"]
  tags                          = merge(local.standard_tags_map.iam, { protected = "true" })

  # force_detach_policies = true
  inline_policy_statements = []
}


module "emr_serverless_job" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-assumable-role?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  create_role = true

  description           = "EMR serverless job execution role"
  trusted_role_services = ["emr-serverless.amazonaws.com"]

  role_path = "/service-role/"

  name           = "emrserverless"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["execution-role"]
  label_order    = ["account_attr", "access_project", "region_abbr", "name", "attributes"]

  custom_role_policy_arns = ["arn:aws:iam::${local.aws_account_id}:policy/service-role/${local.account_attr}-${var.region_abbr}-${local.access_project}-emrserverless-execution-role-policy"]
  tags                    = merge(local.standard_tags_map.iam, { protected = "true" })

  # force_detach_policies = true
  inline_policy_statements = []
}
```
**glue.tf**

Add glue service role for datamesh project.

```hcl
module "glue" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-assumable-role?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  create_role = true

  description           = "AWS Glue Service Role"
  trusted_role_services = ["glue.amazonaws.com"]

  role_path = "/service-role/"

  # Labels
  name           = "glue"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["service-role"]
  label_order    = ["account_attr", "access_project", "region_abbr", "name", "attributes"]

  custom_role_policy_arns       = ["arn:aws:iam::${local.aws_account_id}:policy/service-role/${local.account_attr}-${var.region_abbr}-${local.access_project}-glue-execution-role-policy"]
  role_permissions_boundary_arn = data.terraform_remote_state.bootstrap[0].outputs.boundary_policy_arn["AccountBoundaryPolicy"]
  tags                          = merge(local.standard_tags_map.iam, { protected = "true" })

  # force_detach_policies = true
  inline_policy_statements = [{
    statements = [{
      sid = "VisualEditor5"
      actions = [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      effect = "Allow",
      resources = [
        "arn:aws:s3:::aws-glue-*/*",
        "arn:aws:s3:::*/*aws-glue-*/*"
      ],
      not_actions   = [],
      not_resources = [],
      principals    = [],
      condition     = []
    }]
  }]
}
```
**lakeformation.tf**

Add lakeformation workflow role for datamesh project.

```hcl
module "lakeformation_workflow" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-assumable-role?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  create_role = true

  description           = "AWS Lakeformatioin Workflow Service Role"
  trusted_role_services = ["glue.amazonaws.com"]

  role_path = "/service-role/"

  # Labels
  name           = "lakeformation"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["workflow-role"]
  label_order    = ["account_attr", "access_project", "region_abbr", "name", "attributes"]

  custom_role_policy_arns       = ["arn:aws:iam::aws:policy/service-role/AWSGlueServiceRole"]
  role_permissions_boundary_arn = data.terraform_remote_state.bootstrap[0].outputs.boundary_policy_arn["AccountBoundaryPolicy"]
  tags                          = merge(local.standard_tags_map.iam, { protected = "true" })

  # force_detach_policies = true
  inline_policy_statements = [
    {
      statements = [{
        sid = "VisualEditor0"
        actions = [
          "logs:DescribeLogGroups"
        ],
        effect        = "Allow",
        resources     = ["*"],
        not_actions   = [],
        not_resources = [],
        principals    = [],
        condition     = []
      }]
    }
  ]
}
```

**lambda.tf**

Create lambda execution role for datamesh project.

```hcl
module "lambda" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-assumable-role?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  create_role = true

  description           = "AWS Lambda Execution Role"
  trusted_role_services = ["lambda.amazonaws.com"]

  role_path = "/service-role/"

  # Labels
  name           = "lambda"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["execution-role"]
  label_order    = ["account_attr", "access_project", "region_abbr", "name", "attributes"]

  custom_role_policy_arns       = ["arn:aws:iam::${local.aws_account_id}:policy/service-role/${local.account_attr}-${var.region_abbr}-${local.access_project}-lambda-execution-role-policy"]
  role_permissions_boundary_arn = data.terraform_remote_state.bootstrap[0].outputs.boundary_policy_arn["AccountBoundaryPolicy"]
  tags                          = merge(local.standard_tags_map.iam, { protected = "true" })

  # force_detach_policies = true
  #   inline_policy_statements = []
}
```

**redshift_serverless.tf**

Create Redshift serverless iam role for datamesh project.

```hcl
module "redshift_serverless" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-assumable-role?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  description           = "Amazon Redshift Serverless IAM Role"
  trusted_role_services = ["redshift-serverless.amazonaws.com", "redshift.amazonaws.com"]
  role_path             = "/service-role/"

  # Labels
  name                    = "redshift"
  access_project          = local.access_project
  access_team             = local.access_team
  attributes              = ["serverless", "service", "role"]
  label_order             = ["account_attr", "access_project", "region_abbr", "name", "attributes"]
  custom_role_policy_arns = []

  role_permissions_boundary_arn = data.terraform_remote_state.bootstrap[0].outputs.boundary_policy_arn["AccountBoundaryPolicy"]
  tags                          = merge(local.standard_tags_map.iam, { protected = "true" })

  # force_detach_policies = true
  inline_policy_statements = [{
    statements = [{
      sid = "S3ReadAccess"
      actions = [
        "s3:GetObject",
        "s3:ListBucket",
      ],
      effect = "Allow",
      resources = [
        "arn:aws:s3:::${local.account_attr}-${var.region_abbr}-${local.access_project}-curated",
        "arn:aws:s3:::${local.account_attr}-${var.region_abbr}-${local.access_project}-curated/*",
        "arn:aws:s3:::${local.account_attr}-${var.region_abbr}-${local.access_project}-raw",
        "arn:aws:s3:::${local.account_attr}-${var.region_abbr}-${local.access_project}-raw/*",
        "arn:aws:s3:::${local.account_attr}-${var.region_abbr}-${local.access_project}-datateam",
        "arn:aws:s3:::${local.account_attr}-${var.region_abbr}-${local.access_project}-datateam/*"
      ],
      not_actions   = [],
      not_resources = [],
      principals    = [],
      condition     = []
      },
    ]
  }]
}
```

**s3batchops.tf**

Create s3batchops service role for datamesh project.

```hcl
module "s3batchops" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-assumable-role?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  create_role = true

  description           = "AWS S3 Batch Ops Service Role"
  trusted_role_services = ["batchoperations.s3.amazonaws.com"]

  role_path = "/service-role/"

  # Labels
  name           = "s3batchops"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["service-role"]
  label_order    = ["account_attr", "access_project", "region_abbr", "name", "attributes"]

  custom_role_policy_arns       = ["arn:aws:iam::${local.aws_account_id}:policy/service-role/${local.account_attr}-${var.region_abbr}-${local.access_project}-s3batchops-execution-role-policy"]
  role_permissions_boundary_arn = data.terraform_remote_state.bootstrap[0].outputs.boundary_policy_arn["AccountBoundaryPolicy"]
  tags                          = merge(local.standard_tags_map.iam, { protected = "true" })

  # force_detach_policies = true
  # inline_policy_statements = []
}
```

**sagemaker.tf**

Create sagemaker service role for datamesh project.

```hcl
module "sagemaker" {
  source      = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-assumable-role?ref=tags/v3.2.110"
  providers   = { aws = aws.aue1 }
  create_role = true

  description           = "AWS SageMaker Service Role"
  trusted_role_services = ["sagemaker.amazonaws.com"]

  role_path = "/service-role/"

  # Labels
  name           = "sagemaker"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["service-role"]
  label_order    = ["account_attr", "access_project", "region_abbr", "name", "attributes"]

  custom_role_policy_arns       = ["arn:aws:iam::${local.aws_account_id}:policy/service-role/${local.account_attr}-${var.region_abbr}-${local.access_project}-sagemaker-execution-role-policy"]
  role_permissions_boundary_arn = data.terraform_remote_state.bootstrap[0].outputs.boundary_policy_arn["AccountBoundaryPolicy"]
  tags                          = merge(local.standard_tags_map.iam, { protected = "true" })

  # force_detach_policies = true
  # inline_policy_statements = []
}
```

**Step 5: Update `outputs.tf`.**

```hcl
output "service_role_arns" {
  description = "Service Role ARNs"
  value = compact([
    try(module.glue.this_iam_role_arn, []),
    try(module.lakeformation_workflow.this_iam_role_arn, []),
    try(module.lambda.this_iam_role_arn, []),
    try(module.airflow_mwaa.this_iam_role_arn, []),
    try(module.batch_execution_role.this_iam_role_arn, []),
    try(module.batch_job_role.this_iam_role_arn, []),
    try(module.emr_cluster.this_iam_role_arn, []),
    try(module.emr_serverless_job.this_iam_role_arn, []),
    try(module.emr_ec2.this_iam_role_arn, []),
    try(module.s3batchops.this_iam_role_arn, []),
    try(module.rds_s3import.this_iam_role_arn, []),
    try(module.rds_s3export.this_iam_role_arn, []),
    try(module.redshift_serverless.this_iam_role_arn, []),
     try(module.sagemaker.this_iam_role_arn, []),
  ])
}
```

**Step 6: Update `terraform.tfvars`.**

Replace the value for `costcenter` and `access-project`. 

Keep or update the other tags as necessary.

```hcl
common_tags = {
  ##... omitted

  Owner               = "DevOps"                   # Owner/Requester of the resource
  Owner2              = ""                         # Owner/Requester of the resource
  Agency              = "ANN"                      # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                 # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-infra" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-infra" as default for infra projects.
  access-team         = "datateam"                   # Specify 'access-team', else set as devops.
  access-project      = "elde2"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Configure the remote state**

Note: Before executing terraform commands, make sure the following booleans are set to 'true' in main.tf to retrieve remote state data from Terraform backend.

```hcl
# Make sure following boolean are turned on to activate ‘terraform_remote_state’ data source configuration.
use_vpc_remote_state             = true
use_sg_remote_state              = true
use_bootstrap_remote_state       = true
```
**Step 8: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```hcl
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```hcl
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```
> **NOTE:** Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.

**Step 9: Apply the configuration, and push the feature branch**
Execute below commands:

```bash
$ make ensure_pre_commit
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 10: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch -> master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.

 <a href="#top">Back to top</a>
 
---

## 26. Create fine-grained IAM policies for data engineering team [DATAMESH]

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit` command from the root of the repo.
4. Follows the steps below to create managed IAM policies (service and execution role policies) for datamesh project.
5. The below code create policies for 'elde2' datamesh team where access-project=elde2 and access-team=datateam.

**Step 1: Create a directory for the new configuration under the root of the account repo.**

```bash
$ mkdir -p global/iam-policies/okta-elde2-datateam
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```bash
$ cd global/iam-policies/okta-elde2-datateam
$ pip3 install git+ssh://git@bitbucket.org/annalect/pylect-infra-terraform@v.0.x --upgrade
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required.

```hcl
# Additional locals
locals {
  emr_cluster_id = "j-1JK4Q8N6NYEBM"
  emr_studio_id = "es-5F3R9027FP0IMKVHXB9S2QFFC"
}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

**assume_roles.tf**

Create iam role policy for datamesh team to assume roles with sts_setsourceidentity.

```hcl
module "sourceidentity_assume_roles" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-policy?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  description = "Allows to assume roles"

  # Labels
  name           = "setsourceidentity"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["assumeroles"]
  label_order    = ["account_attr", "access_project", "access_team", "name", "attributes"]
  tags           = local.standard_tags_map.default

  role_requires_setsourceidentity = true
  assume_role_arns                = ["arn:aws:iam::${local.aws_account_id}:role/lakeformation-data-engineer"]
  role_sts_setsourceidentity      = ["$${aws:PrincipalTag/login}"]
}

module "assume_roles" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-policy?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  description = "Allows to assume roles"

  # Labels
  name           = "sts"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["assumeroles"]
  label_order    = ["account_attr", "access_project", "access_team", "name", "attributes"]
  tags           = local.standard_tags_map.default

  assume_role_arns = ["arn:aws:iam::091021251638:role/ann04-sandbox-terraform"]
}
```
**glue_athena.tf**

Add iam role policy for datamesh team to allow access to glue and athena resources.

```hcl
module "glue_athena" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-policy?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  description = "Allows access to Glue"

  # Labels
  name           = "glue"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["access"]
  label_order    = ["account_attr", "access_project", "access_team", "name", "attributes"]
  tags           = local.standard_tags_map.default


  # If you're creating custom policy, the boolean below must be set to 'true'
  create_template_based_custom_policy = true

  datalake_access_policy_vars = {
    lakeformation_get_data_access      = true
    athena_workgroup                   = [local.access_project]
    glue_database_readonly             = []
    cross_account_glue_databases       = []
    glue_database_readwrite            = []
    glue_connections                   = []
    glue_db_slash_userdefinedfunctions = []
    athena_query_output_bucket         = ["${local.account_attr}-${var.region_abbr}-${local.access_project}-athena"]
    glue_db_full_access_by_name_prefix = ["${local.access_project}_*"]
  }
}
```

**emr.tf**

Add iam role policy for datamesh team to allow access to EMR resources.

> **Note:**
> Replace the emr studio and cluster id arn as appropriate.

```hcl
module "emr" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-policy?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  enabled = true

  description = "Allows access to EMR resources"

  # Labels
  name           = "emr"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["access"]
  label_order    = ["account_attr", "access_project", "access_team", "name", "attributes"]
  tags           = local.standard_tags_map.default

  policy_statements = [{
    sid = "DenyAccess"
    actions = [
      "emr-serverless:CreateApplication",
      "elasticmapreduce:RunJobFlow",
      "emr-serverless:UpdateApplication",
      "emr-serverless:DeleteApplication",
    ]
    effect    = "Deny"
    resources = ["*"]
    condition = []
    },
    {
      sid = "CreateEditor"
      actions = [
        "elasticmapreduce:CreateEditor"
      ]
      effect = "Deny"
      not_resources = [
        "arn:aws:elasticmapreduce:${var.region}:${local.aws_account_id}:studio/${local.emr_studio_id}",
        "arn:aws:elasticmapreduce:${var.region}:${local.aws_account_id}:cluster/${local.emr_cluster_id}"
      ]
      condition = []
  }]
}
```

**kms_sm_parameters.tf**

Add iam role policy for datamesh team to allow access to AWS KMS Key, SSM Parameter store and SecretsManager keys.

```hcl
module "secrets" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-policy?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  enabled = true

  description = "IAM Policy for granting access to SSM, SecretsManager and KMS Keys"

  # Labels
  name           = "secrets"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["kms-sm-parameter-key-access"]
  label_order    = ["account_attr", "access_project", "access_team", "name", "attributes"]
  tags           = local.standard_tags_map.default

  # If you're creating custom policy, the boolean below must be set to 'true'
  create_template_based_custom_policy = true

  secrets_parameters_and_kms_vars = {
    # Enable access based on 'true' or 'false' boolean values
    describe_secretmanager_secrets = true
    describe_ssm_parameters        = true
    get_secretsmanager_secret_arn  = ["arn:aws:secretsmanager:${var.region}:${local.aws_account_id}:secret:/${local.access_project}/*"]
    get_ssm_parameter_arn          = ["arn:aws:ssm:${var.region}:${local.aws_account_id}:parameter/${local.access_project}/*"]
    kmskey_to_decrypt_secret       = ["arn:aws:kms:${var.region}:${local.aws_account_id}:key/xxxxxx-xxxx-4b32-8508-74649cc0651a"]
    kmskey_to_allow_s3sse_kms      = ["arn:aws:kms:${var.region}:${local.aws_account_id}:key/xxxxxx-xxxx-4b32-8508-74649cc0651a"]
  }
}
```

**misc_access.tf**

Add iam role policy for datamesh team to allow access to misc AWS resources (temp fix for missing permissions).

```hcl
module "misc_access" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-policy?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  enabled = true

  description = "Additional access to AWS resources"

  # Labels
  name           = "misc"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["access"]
  label_order    = ["account_attr", "access_project", "access_team", "name", "attributes"]
  tags           = local.standard_tags_map.default

  policy_statements = [
    {
      sid = "MiscAccess"
      actions = [
        "account:ListRegions",
      ]
      effect    = "Allow"
      resources = ["*"]
      condition = []
  }]

  role_names = ["${local.access_project}-datateam"]
}
```

**s3readonly.tf**

Add iam role policy for datamesh team to allow ReadOnly access to AWS S3 Buckets.

```hcl
module "s3readonly" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-policy?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  enabled     = true
  description = "S3 Read Only Access Policy"

  # Labels
  name           = "s3"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["readonly-access"]
  label_order    = ["account_attr", "access_project", "access_team", "name", "attributes"]
  tags           = local.standard_tags_map.default


  # If you're creating IAM policy for s3 bucket, 'create_s3_policy' must be set 'true'
  create_s3_policy = true

  # Enable access based on 'true' or 'false' boolean values
  multiple_upload_access = false
  get_object_version     = true
  get_object_acl         = true
  get_object_tagging     = true


  # Allow ReadWrite (with List) access to s3 bucket(s) with or without objectprefix and w/o forwarding slash
  s3_object_readwrite_access = []

  # Allow ReadOnly (with List) access to s3 bucket(s) with or without object prefix and w/o forwarding slash
  s3_object_readonly_access = [
    "annalect-example-ro1/abcd",
    "annalect-example-ro2",
  ]

  # Allow ListOnly access to whole bucket(s) w/o forwarding slash
  s3_bucket_list_access = []

  # Restrict access to S3 object prefixes (from root of bucket) with /* suffix
  s3_bucket_list_prefix_only = [
    { bucket = "annalect-example-list-prefix3", prefix = ["x/y/z/*", "a/b/c/*"] },
  ]

  # Deny access to s3 bucket(s) with or without prefix
  s3_object_explicit_deny_access = [
    { bucket = "annalect-example-deny1", prefix = ["*"] },
    { bucket = "annalect-example-deny2", prefix = ["lz/*"] },
  ]
}
```

**s3readwrite.tf**

Add iam role policy for datamesh team to allow ReadWrite access to AWS S3 Buckets.

```hcl
module "s3readwrite" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-policy?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  description = "S3 Read Write Access Policy"

  # Labels
  name           = "s3"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["readwrite-access"]
  label_order    = ["account_attr", "access_project", "access_team", "name", "attributes"]
  tags           = local.standard_tags_map.default


  # If you're creating IAM policy for s3 bucket, 'create_s3_policy' must be set 'true'
  create_s3_policy = true

  # Enable access based on 'true' or 'false' boolean values 
  datateam_access = true


  # Allow ReadWrite (with List) access to s3 bucket(s) with or without objectprefix and w/o forwarding slash
  s3_object_readwrite_access = [
    "aws-logs-${local.aws_account_id}-${var.region}",
    "${local.account_attr}-${var.region_abbr}-${local.access_project}-athena",
    "${local.account_attr}-${var.region_abbr}-${local.access_project}-datateam",
    "${local.account_attr}-${var.region_abbr}-${local.access_project}-mwaa-*",
    "${local.account_attr}-${var.region_abbr}-${local.access_project}-curated"
    "${local.account_attr}-${var.region_abbr}-${local.access_project}-raw",
  ]

  # Allow ReadOnly (with List) access to s3 bucket(s) with or without object prefix and w/o forwarding slash
  s3_object_readonly_access = []

  # Allow ListOnly access to whole bucket(s) w/o forwarding slash
  s3_bucket_list_access = []

  # Restrict access to S3 object prefixes (from root of bucket) with /* suffix
  s3_bucket_list_prefix_only = []
  # s3_bucket_list_prefix_only = [
  #  { bucket = "annalect-example-list-prefix3", prefix = ["x/y/z/*", "a/b/c/*"] },
  # ]

  # Deny access to s3 bucket(s) with or without prefix
  s3_object_explicit_deny_access = []
  # s3_object_explicit_deny_access = [
  #  { bucket = "annalect-example-deny1", prefix = ["*"] },
  #  { bucket = "annalect-example-deny2", prefix = ["lz/*"] },
  # ]
}
```
**aggregated_policies.tf**

Add aggregated iam policies.

```hcl
module "aggregated_s3_policy" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-policy-document-aggregator?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  description = "Aggregated IAM Policy"

  # Labels
  name           = "aggregated"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["s3", "policy"]
  label_order    = ["account_attr", "access_project", "access_team", "name", "attributes"]
  tags           = local.standard_tags_map.default

  source_documents = [
    module.s3readonly.policy,
    module.s3readwrite.policy,
  ]
  role_names = ["${local.access_project}-datateam"]
}

module "aggregated_policy1" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/iam/iam-policy-document-aggregator?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  description = "Aggregated IAM Policy"

  # Labels
  name           = "aggregated"
  access_project = local.access_project
  access_team    = local.access_team
  attributes     = ["policy", "1"]
  label_order    = ["account_attr", "access_project", "access_team", "name", "attributes"]
  tags           = local.standard_tags_map.default

  source_documents = [
    module.assume_roles.policy,
    module.sourceidentity_assume_roles.policy,
    module.glue_athena.policy,
    module.secrets.policy,
    module.emr.policy,
  ]
  role_names = ["${local.access_project}-datateam"]
}
```

**Step 5: Update `outputs.tf`.**

Append the following to the bottom of outputs.

```hcl
output "okta_role_policy_arns" {
  description = "Okta Role Policy ARNs"
  value = compact([
    try(module.glue_athena.arn, []),
    try(module.secrets.arn, []),
    try(module.emr.arn, []),
    try(module.misc_access.arn, []),
    try(module.s3readonly.arn, []),
    try(module.s3readwrite.arn, []),
    try(module.sourceidentity_assume_roles.arn, []),
    try(module.assume_roles.arn, []),
    try(module.aggregated_s3_policy.arn, []),
    try(module.aggregated_policy1.arn, []),
  ])
}
```

**Step 6: Update `terraform.tfvars`.**

Update values for : `Owner`, `Agency`, `Client`, `costcenter`, `access-team`, and `access-project`.

```hcl
common_tags = {
  ##... omitted

  Owner               = "DevOps"                   # Owner/Requester of the resource
  Owner2              = ""                         # Owner/Requester of the resource
  Agency              = "ANN"                      # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                 # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-interal" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-infra" as default for infra projects.
  access-team         = "datateam"                   # Specify 'access-team', else set as devops.
  access-project      = "elde2"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Configure the remote state**

Note: Before executing terraform commands, make sure the following booleans are set to 'true' in main.tf to retrieve remote state data from Terraform backend.

```hcl
# Make sure following boolean are turned on to activate ‘terraform_remote_state’ data source configuration.
use_vpc_remote_state             = true
use_sg_remote_state              = true
use_bootstrap_remote_state       = true
```

**Step 8: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```hcl
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```hcl
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```
> **NOTE:** Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.

**Step 9: Apply the configuration, and push the feature branch**
Execute below commands:

```bash
$ make ensure_pre_commit
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 10: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch -> master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.

 <a href="#top">Back to top</a>
 
---

## 27. Deploy Athena for data engineering team [DATAMESH]

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit ` command from the root of the repo.
4. The below code creates the athena for s3 for the `auds` datamesh team where access-project=auds, and access-team=datateam. Change the access-project to match your own account.

**Step 1: Create a directory for the new configuration under the root of the account repo.**

Repeat this entire section for each enabled region for the account.

To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1' at the root level of the directory. Replace ${access-project} with your AWS account access project (ex: `auds`).

```
$ mkdir -p aue1/athena/project-${access-project}
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```
$ cd aue1/athena/project-${access-project}
$ ${pylect-infra-terraform update}
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

> **Notes:**
>  1. Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required.
>  2. If no locals need to be added, skip this step.

```
# Additional locals
locals {}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

**workgroup.tf**

Describe the athena workgroup.

```
module "athena_workgroup" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/athena?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }
  enabled   = true

  description = "The Athena Workgroup for ${local.access_project} ${local.access_team}."

  database_s3_location            = "${var.account_attr}-${var.region_abbr}-${local.access_project}-athena"
  workgroup_name                  = local.access_project
  query_output_location           = format("s3://%s/%s", "${var.account_attr}-${var.region_abbr}-${local.access_project}-athena", "output/")
  enforce_workgroup_configuration = true
  bytes_scanned_cutoff_per_query  = 1000000000 # 1GB
  tags = merge(local.standard_tags_map["default"],
    {
      access_project = local.access_project
      access_team    = local.access_team
      Project        = "annalect-internal-${local.access_project}"
  })
}
```

**main.tf**

Confirm all booleans for remote state match the following, in `main.tf`:

```
locals {
  ##...

  # Turn on and off to retrieve remote state from backend
  use_secure_baseline_remote_state = false
  use_vpc_remote_state             = false
  use_sg_remote_state              = false
  use_bootstrap_remote_state       = false
  use_iam_policy_remote_state      = false
}
```

**Step 5: Update Outputs**

Ensure all customer managed keys are available to other modules.

**outputs.tf**

```
# Add additional outputs below
# Athena WorkGroups
output "athena_workgroups" {
  description = "Athena workgroup Names"
  value = concat([
    module.athena_workgroup.workgroup,
    ],
  )
}
```

**Step 6: Update `terraform.tfvars`**

Update the required tags: 

* `Owner`
* `Agency`
* `Client`
* `costcenter`
* `access-team`
* `access-project`

Make sure to use the correct costcenter and access-project.

```hcl
common_tags = {
  ##... omitted

  Owner               = "datateam"                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-auds" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-auds" as default for infra projects.
  access-team         = "datateam"                  # Specify 'access-team', else set as devops.
  access-project      = "auds"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```hcl
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```hcl
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```
> **NOTE:** Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.

**Step 8: Apply the configuration, and push the feature branch**
Execute below commands:

```bash
$ make ensure_pre_commit
$ make fmt
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 9: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch -> master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.

 <a href="#top">Back to top</a>
 
---

## 28. Deploy AWS batch compute requirements [DATAMESH]

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit ` command from the root of the repo.
4. The below code creates the compute requirements for the `auds` datamesh team where access-project=auds, and access-team=datateam. Change the access-project to match your own account.

**Step 1: Create a directory for the new configuration under the root of the account repo.**

Repeat this entire section for each enabled region for the account.

To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1' at the root level of the directory. 

```
$ mkdir -p aue1/aws-batch/compute-environments
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```
$ cd aue1/aws-batch/compute-environments
$ ${pylect-infra-terraform update}
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

> **Notes:**
>  1. Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required.
>  2. If no locals need to be added, skip this step.

```
locals {
  key_pair_name = "${local.account_attr}-${var.region_abbr}-infra-ec2-key"
}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

Deploy the compute environments for the AWS Batch.

**compute-environments.tf**

```
module "batch_ec2_compute_environment1" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/aws-batch/compute-environment?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }
  enabled   = true

  compute_environment_type = "MANAGED"              # MANAGED | UNMANAGED
  state                    = "ENABLED"              # ENABLED | DISABLED
  compute_resource_type    = "EC2"                  # EC2 | SPOT | FARGATE | FARGATE_SPOT
  allocation_strategy      = "BEST_FIT_PROGRESSIVE" # BEST_FIT | BEST_FIT_PROGRESSIVE | SPOT_CAPACITY_OPTIMIZED
  instance_type            = "optimal"
  min_vcpus                = 0
  desired_vcpus            = 0
  max_vcpus                = 64

  service_role  = "arn:aws:iam::${local.aws_account_id}:role/service-role/${local.account_attr}-infra-batch-service-role"
  instance_role = "arn:aws:iam::${local.aws_account_id}:instance-profile/service-role/${local.account_attr}-infra-batch-instance-role"

  subnet_ids = data.terraform_remote_state.vpc[0].outputs["private_az_subnet_ids"]["dev"]
  security_group_ids = [
    data.terraform_remote_state.sg[0].outputs["security_groups"]["main"]["ecs"]["dev"],
    data.terraform_remote_state.sg[0].outputs["security_groups"]["main"]["ecs"]["mgmt"]
  ]

  metadata_http_endpoint_enabled = true
  metadata_http_tokens_required  = true # set to false to restore the use of IMDSv1
  key_pair_name                  = "${local.account_attr}-${var.region_abbr}-infra-ec2-key"

  # Labels
  name           = "batch"
  access_project = var.account_attr
  attributes     = ["ec2", "01"]
  label_order    = ["access_project", "name", "attributes"]

  tags                  = merge(local.tags, { Project = var.common_tags.costcenter, access-project = "infra" })
  compute_resource_tags = merge(local.tags, { Project = var.common_tags.costcenter, access-project = local.access_project })

  block_device_mappings = [
    {
      device_name  = "/dev/xvda"
      no_device    = null
      virtual_name = null
      ebs = {
        delete_on_termination = true
        encrypted             = true
        volume_size           = 200
        volume_type           = "gp2"
        iops                  = null
        kms_key_id            = null
        snapshot_id           = null
      }
    }
  ]
}

module "batch_ec2_compute_environment2" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/aws-batch/compute-environment?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }
  enabled   = true

  compute_environment_type = "MANAGED"              # MANAGED | UNMANAGED
  state                    = "ENABLED"              # ENABLED | DISABLED
  compute_resource_type    = "EC2"                  # EC2 | SPOT | FARGATE | FARGATE_SPOT
  allocation_strategy      = "BEST_FIT_PROGRESSIVE" # BEST_FIT | BEST_FIT_PROGRESSIVE | SPOT_CAPACITY_OPTIMIZED
  instance_type            = "optimal"
  min_vcpus                = 0
  desired_vcpus            = 0
  max_vcpus                = 64

  service_role  = "arn:aws:iam::${local.aws_account_id}:role/service-role/${local.account_attr}-infra-batch-service-role"
  instance_role = "arn:aws:iam::${local.aws_account_id}:instance-profile/service-role/${local.account_attr}-infra-batch-instance-role"

  subnet_ids = data.terraform_remote_state.vpc[0].outputs["private_az_subnet_ids"]["dev"]
  security_group_ids = [
    data.terraform_remote_state.sg[0].outputs["security_groups"]["main"]["ecs"]["dev"],
    data.terraform_remote_state.sg[0].outputs["security_groups"]["main"]["ecs"]["mgmt"]
  ]

  metadata_http_endpoint_enabled = true
  metadata_http_tokens_required  = true # set to false to restore the use of IMDSv1
  key_pair_name                  = "${local.account_attr}-${var.region_abbr}-infra-ec2-key"

  # Labels
  name           = "batch"
  access_project = var.account_attr
  attributes     = ["ec2", "02"]
  label_order    = ["access_project", "name", "attributes"]

  tags                  = merge(local.tags, { Project = var.common_tags.costcenter, access-project = "infra" })
  compute_resource_tags = merge(local.tags, { Project = var.common_tags.costcenter, access-project = local.access_project })

  block_device_mappings = [
    {
      device_name  = "/dev/xvda"
      no_device    = null
      virtual_name = null
      ebs = {
        delete_on_termination = true
        encrypted             = true
        volume_size           = 200
        volume_type           = "gp2"
        iops                  = null
        kms_key_id            = null
        snapshot_id           = null
      }
    }
  ]
}

module "batch_ec2_spot_compute_environment" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/aws-batch/compute-environment?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }
  enabled   = true

  compute_environment_type = "MANAGED" # MANAGED | UNMANAGED
  state                    = "ENABLED" # ENABLED | DISABLED
  compute_resource_type    = "SPOT"    # EC2 | SPOT | FARGATE | FARGATE_SPOT
  bid_percentage           = 100
  allocation_strategy      = "BEST_FIT_PROGRESSIVE" # BEST_FIT | BEST_FIT_PROGRESSIVE | SPOT_CAPACITY_OPTIMIZED
  instance_type            = "optimal"
  min_vcpus                = 0
  desired_vcpus            = 0
  max_vcpus                = 64

  service_role        = "arn:aws:iam::${local.aws_account_id}:role/service-role/${local.account_attr}-infra-batch-service-role"
  instance_role       = "arn:aws:iam::${local.aws_account_id}:instance-profile/service-role/${local.account_attr}-infra-batch-instance-role"
  spot_iam_fleet_role = "arn:aws:iam::${local.aws_account_id}:role/${local.account_attr}-infra-batch-spot-fleet-role"

  subnet_ids = data.terraform_remote_state.vpc[0].outputs["private_az_subnet_ids"]["dev"]
  security_group_ids = [
    data.terraform_remote_state.sg[0].outputs["security_groups"]["main"]["ecs"]["dev"],
    data.terraform_remote_state.sg[0].outputs["security_groups"]["main"]["ecs"]["mgmt"]
  ]

  metadata_http_endpoint_enabled = true
  metadata_http_tokens_required  = false # set to false to restore the use of IMDSv1
  key_pair_name                  = "${local.account_attr}-${var.region_abbr}-infra-ec2-key"

  # Labels
  name           = "batch"
  access_project = var.account_attr
  attributes     = ["ec2-spot"]
  label_order    = ["access_project", "name", "attributes"]

  tags                  = merge(local.tags, { Project = var.common_tags.costcenter, access-project = "infra" })
  compute_resource_tags = merge(local.tags, { Project = var.common_tags.costcenter, access-project = local.access_project })
}
```

Deploy the fargate compute environments for AWS Batch

**fargate_comp_env.tf**


```
module "batch_fargate_compute_environment1" {
  #source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/aws-batch/compute-environment?ref=tags/v3.2.110"
  source    = "../../../modules/aws-batch/compute-environment/"
  providers = { aws = aws.aue1 }
  enabled   = true

  compute_environment_type = "MANAGED" # MANAGED | UNMANAGED
  state                    = "ENABLED" # ENABLED | DISABLED
  compute_resource_type    = "FARGATE" # EC2 | SPOT | FARGATE | FARGATE_SPOT
  max_vcpus                = 64

  #service_role  = "arn:aws:iam::${local.aws_account_id}:role/service-role/${local.account_attr}-infra-batch-service-role"
  instance_role = "arn:aws:iam::${local.aws_account_id}:instance-profile/service-role/${local.account_attr}-infra-batch-instance-role"

  subnet_ids = data.terraform_remote_state.vpc[0].outputs["private_az_subnet_ids"]["dev"]
  security_group_ids = [
    data.terraform_remote_state.sg[0].outputs["security_groups"]["main"]["ecs"]["dev"],
    data.terraform_remote_state.sg[0].outputs["security_groups"]["main"]["ecs"]["mgmt"]
  ]

  # Labels
  name           = "batch"
  access_project = var.account_attr
  attributes     = ["fargate", "01"]
  label_order    = ["access_project", "name", "attributes"]

  tags = merge(local.tags, { Project = var.common_tags.costcenter, access-project = "infra" })
}


module "batch_fargate_compute_environment2" {
  #source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/aws-batch/compute-environment?ref=tags/v3.2.110"
  source    = "../../../modules/aws-batch/compute-environment/"
  providers = { aws = aws.aue1 }
  enabled   = true

  compute_environment_type = "MANAGED" # MANAGED | UNMANAGED
  state                    = "ENABLED" # ENABLED | DISABLED
  compute_resource_type    = "FARGATE" # EC2 | SPOT | FARGATE | FARGATE_SPOT
  max_vcpus                = 64

  #service_role  = "arn:aws:iam::${local.aws_account_id}:role/service-role/${local.account_attr}-infra-batch-service-role"
  instance_role = "arn:aws:iam::${local.aws_account_id}:instance-profile/service-role/${local.account_attr}-infra-batch-instance-role"

  subnet_ids = data.terraform_remote_state.vpc[0].outputs["private_az_subnet_ids"]["dev"]
  security_group_ids = [
    data.terraform_remote_state.sg[0].outputs["security_groups"]["main"]["ecs"]["dev"],
    data.terraform_remote_state.sg[0].outputs["security_groups"]["main"]["ecs"]["mgmt"]
  ]

  # Labels
  name           = "batch"
  access_project = var.account_attr
  attributes     = ["fargate", "02"]
  label_order    = ["access_project", "name", "attributes"]

  tags = merge(local.tags, { Project = var.common_tags.costcenter, access-project = "infra" })

}

module "batch_fargate_spot_compute_environment" {
  #source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/aws-batch/compute-environment?ref=tags/v3.2.110"
  source    = "../../../modules/aws-batch/compute-environment/"
  providers = { aws = aws.aue1 }
  enabled   = true

  compute_environment_type = "MANAGED"      # MANAGED | UNMANAGED
  state                    = "ENABLED"      # ENABLED | DISABLED
  compute_resource_type    = "FARGATE_SPOT" # EC2 | SPOT | FARGATE | FARGATE_SPOT
  max_vcpus                = 64

  #service_role  = "arn:aws:iam::${local.aws_account_id}:role/service-role/${local.account_attr}-infra-batch-service-role"
  instance_role = "arn:aws:iam::${local.aws_account_id}:instance-profile/service-role/${local.account_attr}-infra-batch-instance-role"

  subnet_ids = data.terraform_remote_state.vpc[0].outputs["private_az_subnet_ids"]["dev"]
  security_group_ids = [
    data.terraform_remote_state.sg[0].outputs["security_groups"]["main"]["ecs"]["dev"],
    data.terraform_remote_state.sg[0].outputs["security_groups"]["main"]["ecs"]["mgmt"]
  ]

  # Labels
  name           = "batch"
  access_project = var.account_attr
  attributes     = ["fargate-spot"]
  label_order    = ["access_project", "name", "attributes"]

  tags = merge(local.tags, { Project = var.common_tags.costcenter, access-project = "infra" })

}
```

**main.tf**

Confirm all booleans for remote state match the following, in `main.tf`:

```
locals {
  ##...

  # Turn on and off to retrieve remote state from backend
  use_secure_baseline_remote_state = false
  use_vpc_remote_state             = true
  use_sg_remote_state              = true
  use_bootstrap_remote_state       = true
  use_iam_policy_remote_state      = false
}
```


**Step 5: Update Outputs**

Ensure all customer managed keys are available to other modules.

**outputs.tf**

```
# Add additional outputs below
output "batch_compute_environment" {
  description = "AWS Batch Compute Environment"
  value = {
    (module.batch_ec2_compute_environment1.name) = {
      arn : module.batch_ec2_compute_environment1.arn,
      ecs_instance_role_arn : module.batch_ec2_compute_environment1.ecs_instance_role_arn,
      launch_template_id : module.batch_ec2_compute_environment1.launch_template_id,
      status : module.batch_ec2_compute_environment1.status,
    },
    (module.batch_ec2_compute_environment2.name) = {
      arn : module.batch_ec2_compute_environment2.arn,
      ecs_instance_role_arn : module.batch_ec2_compute_environment2.ecs_instance_role_arn,
      launch_template_id : module.batch_ec2_compute_environment2.launch_template_id,
      status : module.batch_ec2_compute_environment2.status,
    },
    (module.batch_ec2_spot_compute_environment.name) = {
      arn : module.batch_ec2_spot_compute_environment.arn,
      ecs_instance_role_arn : module.batch_ec2_spot_compute_environment.ecs_instance_role_arn,
      launch_template_id : module.batch_ec2_spot_compute_environment.launch_template_id,
      status : module.batch_ec2_spot_compute_environment.status,
    },
    (module.batch_fargate_compute_environment1.name) = {
      arn : module.batch_fargate_compute_environment1.arn,
      ecs_instance_role_arn : module.batch_fargate_compute_environment1.ecs_instance_role_arn,
      launch_template_id : module.batch_fargate_compute_environment1.launch_template_id,
      status : module.batch_fargate_compute_environment1.status,
    },
    (module.batch_fargate_compute_environment2.name) = {
      arn : module.batch_fargate_compute_environment2.arn,
      ecs_instance_role_arn : module.batch_fargate_compute_environment2.ecs_instance_role_arn,
      launch_template_id : module.batch_fargate_compute_environment2.launch_template_id,
      status : module.batch_fargate_compute_environment2.status,
    },
    (module.batch_fargate_spot_compute_environment.name) = {
      arn : module.batch_fargate_spot_compute_environment.arn,
      ecs_instance_role_arn : module.batch_fargate_spot_compute_environment.ecs_instance_role_arn,
      launch_template_id : module.batch_fargate_spot_compute_environment.launch_template_id,
      status : module.batch_fargate_spot_compute_environment.status,
    },
  }
}
```

**Step 6: Update `terraform.tfvars`**

Update the required tags: 

* `Owner`
* `Agency`
* `Client`
* `costcenter`
* `access-team`
* `access-project`
* `Data_Classification`

Make sure to use the correct costcenter and access-project.

```hcl
common_tags = {
  ##... omitted
  Data_Classification = "internal"               # Definition of data classification and confidentiality, allowed values: public, internal,  confidential, restricted, pii, phi
  Owner               = ""                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-auds" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-auds" as default for infra projects.
  access-team         = "devops"                  # Specify 'access-team', else set as devops.
  access-project      = "infra"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```hcl
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```hcl
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```
> **NOTE:** Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.

**Step 8: Apply the configuration, and push the feature branch**
Execute below commands:

```bash
$ make ensure_pre_commit
$ make fmt
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 9: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch -> master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.

 <a href="#top">Back to top</a>
 
---

## 29. Deploy AWS batch job definitions [DATAMESH] 

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit ` command from the root of the repo.
4. The below code creates the job definitions for the `auds` datamesh team where access-project=auds, and access-team=datateam. Change the access-project to match your own account.

**Step 1: Create a directory for the new configuration under the root of the account repo.**

Repeat this entire section for each enabled region for the account.

To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1' at the root level of the directory. 

```
$ mkdir -p aue1/aws-batch/job-definitions
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```
$ cd aue1/aws-batch/job-definitions
$ ${pylect-infra-terraform update}
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

> **Notes:**
>  1. Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required.
>  2. If no locals need to be added, skip this step.

```
data "aws_kms_key" "ssm" {
  provider = aws.aue1
  key_id   = "alias/aws/ssm"
}
data "aws_kms_key" "secretsmanager" {
  provider = aws.aue1
  key_id   = "alias/aws/secretsmanager"
}

data "aws_kms_key" "ssm_dev" {
  provider = aws.aue1
  key_id   = "alias/aws/ssm"
}
data "aws_kms_key" "ssm_qa" {
  provider = aws.aue1
  key_id   = "alias/aws/ssm"
}

data "aws_kms_key" "ssm_stg" {
  provider = aws.aue1
  key_id   = "alias/aws/ssm"
}

data "aws_kms_key" "ssm_prod" {
  provider = aws.aue1
  key_id   = "alias/aws/ssm"
}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

Define the ec2 batch job.

**auds-batch-ec2-job-defn.tf**

> **Notes:**
>  1. Replace the `auds` in the filename with your access-project.

```
module "auds_jobs_ec2" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/aws-batch/job-definition?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  platform_capabilities = ["EC2"] # FARGATE|EC2

  # Labels
  enabled        = true
  name           = "batch"
  name_prefix    = "auds_"
  access_project = "auds"
  environment    = "dev"
  attributes     = ["ec2"]
  label_order    = ["attributes"]

  # container_properties
  image                   = "661095214357.dkr.ecr.us-east-1.amazonaws.com/annalect/awsbatch_fetch_and_run:release"
  command_parameters      = { "arg1" : "5", "arg2" : "val2", "arg3" : "val3", "arg4" : "val4", "arg5" : "val5", "arg6" : "val6", }
  command                 = tolist(["myjob.sh", "Ref::arg1", "Ref::arg2", "Ref::arg3", "Ref::arg4", "Ref::arg5", "Ref::arg6"])
  vcpu                    = "1"
  memory                  = "1024"
  job_timeout             = 18000
  retry_strategy_attempts = "1"
  job_role_arn            = "arn:aws:iam::${local.aws_account_id}:role/service-role/${local.account_attr}-${var.region_abbr}-auds-batch-job-role"
  ecs_exec_role_arn       = "arn:aws:iam::${local.aws_account_id}:role/service-role/${local.account_attr}-${var.region_abbr}-auds-batch-execution-role"
  privileged              = false
  volumes                 = [{ "host" : { "sourcePath" : "/datastore" }, "name" : "datastore" }]
  mount_points            = [{ "sourceVolume" : "datastore", "containerPath" : "/datastore", "readOnly" : false }]
  ulimits                 = []
  linux_parameters        = {}
  secrets = [
    { "name" = "BITBUCKET_DEPLOYMENT_KEY", "valueFrom" = "arn:aws:secretsmanager:${var.region}:${data.aws_caller_identity.trusted.account_id}:secret:bitbucket-deployment-key-39fyem" }
  ]

  environment_variables = [
    { "name" : "AWS_DEFAULT_REGION", "value" : var.region },
    { "name" : "ECS_ENABLE_AWSLOGS_EXECUTIONROLE_OVERRIDE", "value" : "true" },
    { "name" : "APP_NAME", "value" : "auds" },
    { "name" : "BUILD_ENV", "value" : "dev" },
    { "name" : "BATCH_FILE_TYPE", "value" : "script" },
    { "name" : "BATCH_FILE_NAME", "value" : "test.sh" },
    { "name" : "NOTIFY_ON_SUCCESS", "value" : "0" },
    { "name" : "BATCH_FILE_REMOTE_URL", "value" : "" },
    { "name" : "CONFIG_PATH", "value" : "/serveroverride.cfg" },
    { "name" : "RUN_SECRET_ENTRYPOINT", "value" : "false" },
    { "name" : "SERVEROVERRIDE_KEY", "value" : "" },
    { "name" : "GET_APPS_SECRETS", "value" : "false" },
    { "name" : "GET_ENV_SECRETS", "value" : "false" },
    { "name" : "KMS_KEY_ID_ARN", "value" : data.aws_kms_key.ssm_dev.arn },
  ]

  tags = local.tags
}
```

Define the AWS project fargate job definitions.

**auds-batch-fargate-job.defn.tf**

> **Notes:**
>  1. Replace the `auds` in the filename with your access-project.

```
module "auds_jobs_fargate" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/aws-batch/job-definition?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  platform_capabilities = ["FARGATE"] # FARGATE|EC2

  # Labels
  enabled        = true
  name           = "batch"
  name_prefix    = "auds_"
  access_project = "auds"
  environment    = "dev"
  attributes     = ["fargate"]
  label_order    = ["attributes"]

  # container_properties
  image                   = "661095214357.dkr.ecr.us-east-1.amazonaws.com/annalect/awsbatch_fetch_and_run:release"
  command_parameters      = { "arg1" : "5", "arg2" : "val2", "arg3" : "val3", "arg4" : "val4", "arg5" : "val5", "arg6" : "val6", }
  command                 = tolist(["myjob.sh", "Ref::arg1", "Ref::arg2", "Ref::arg3", "Ref::arg4", "Ref::arg5", "Ref::arg6"])
  vcpu                    = "0.5"
  memory                  = "1024"
  job_timeout             = 3600
  retry_strategy_attempts = "1"
  job_role_arn            = "arn:aws:iam::${local.aws_account_id}:role/service-role/${local.account_attr}-${var.region_abbr}-auds-batch-job-role"
  ecs_exec_role_arn       = "arn:aws:iam::${local.aws_account_id}:role/service-role/${local.account_attr}-${var.region_abbr}-auds-batch-execution-role"
  volumes                 = []
  mount_points            = []
  privileged              = false
  ulimits                 = []
  linux_parameters        = {}
  secrets = [
    { "name" = "BITBUCKET_DEPLOYMENT_KEY", "valueFrom" = "arn:aws:secretsmanager:${var.region}:${data.aws_caller_identity.trusted.account_id}:secret:bitbucket-deployment-key-39fyem" }
  ]

  environment_variables = [
    { "name" : "AWS_DEFAULT_REGION", "value" : var.region },
    { "name" : "ECS_ENABLE_AWSLOGS_EXECUTIONROLE_OVERRIDE", "value" : "true" },
    { "name" : "APP_NAME", "value" : "auds" },
    { "name" : "BUILD_ENV", "value" : "dev" },
    { "name" : "BATCH_FILE_TYPE", "value" : "script" },
    { "name" : "BATCH_FILE_NAME", "value" : "test.sh" },
    { "name" : "NOTIFY_ON_SUCCESS", "value" : "0" },
    { "name" : "BATCH_FILE_REMOTE_URL", "value" : "" },
    { "name" : "CONFIG_PATH", "value" : "/serveroverride.cfg" },
    { "name" : "RUN_SECRET_ENTRYPOINT", "value" : "false" },
    { "name" : "SERVEROVERRIDE_KEY", "value" : "" },
    { "name" : "GET_APPS_SECRETS", "value" : "false" },
    { "name" : "GET_ENV_SECRETS", "value" : "false" },
    { "name" : "KMS_KEY_ID_ARN", "value" : data.aws_kms_key.ssm_dev.arn },
  ]

  tags = local.tags
}
```

Define the infra batch job ec2 definition.



**main.tf**

Confirm all booleans for remote state match the following, in `main.tf`:

```
locals {
  ##...

  # Turn on and off to retrieve remote state from backend
  use_secure_baseline_remote_state = false
  use_vpc_remote_state             = true
  use_sg_remote_state              = true
  use_bootstrap_remote_state       = true
  use_iam_policy_remote_state      = false
}
```


**Step 5: Update Outputs**

Ensure all customer managed keys are available to other modules.

**outputs.tf**

```
# Add additional outputs below
output "batch_compute_environment" {
  description = "AWS Batch Compute Environment"
  value = {
    (module.batch_ec2_compute_environment1.name) = {
      arn : module.batch_ec2_compute_environment1.arn,
      ecs_instance_role_arn : module.batch_ec2_compute_environment1.ecs_instance_role_arn,
      launch_template_id : module.batch_ec2_compute_environment1.launch_template_id,
      status : module.batch_ec2_compute_environment1.status,
    },
    (module.batch_ec2_compute_environment2.name) = {
      arn : module.batch_ec2_compute_environment2.arn,
      ecs_instance_role_arn : module.batch_ec2_compute_environment2.ecs_instance_role_arn,
      launch_template_id : module.batch_ec2_compute_environment2.launch_template_id,
      status : module.batch_ec2_compute_environment2.status,
    },
    (module.batch_ec2_spot_compute_environment.name) = {
      arn : module.batch_ec2_spot_compute_environment.arn,
      ecs_instance_role_arn : module.batch_ec2_spot_compute_environment.ecs_instance_role_arn,
      launch_template_id : module.batch_ec2_spot_compute_environment.launch_template_id,
      status : module.batch_ec2_spot_compute_environment.status,
    },
    (module.batch_fargate_compute_environment1.name) = {
      arn : module.batch_fargate_compute_environment1.arn,
      ecs_instance_role_arn : module.batch_fargate_compute_environment1.ecs_instance_role_arn,
      launch_template_id : module.batch_fargate_compute_environment1.launch_template_id,
      status : module.batch_fargate_compute_environment1.status,
    },
    (module.batch_fargate_compute_environment2.name) = {
      arn : module.batch_fargate_compute_environment2.arn,
      ecs_instance_role_arn : module.batch_fargate_compute_environment2.ecs_instance_role_arn,
      launch_template_id : module.batch_fargate_compute_environment2.launch_template_id,
      status : module.batch_fargate_compute_environment2.status,
    },
    (module.batch_fargate_spot_compute_environment.name) = {
      arn : module.batch_fargate_spot_compute_environment.arn,
      ecs_instance_role_arn : module.batch_fargate_spot_compute_environment.ecs_instance_role_arn,
      launch_template_id : module.batch_fargate_spot_compute_environment.launch_template_id,
      status : module.batch_fargate_spot_compute_environment.status,
    },
  }
}
```

**Step 6: Update `terraform.tfvars`**

Update the required tags: 

* `Owner`
* `Agency`
* `Client`
* `costcenter`
* `access-team`
* `access-project`
* `Data_Classification`

Make sure to use the correct costcenter and access-project.

```hcl
common_tags = {
  ##... omitted
  Data_Classification = "internal"               # Definition of data classification and confidentiality, allowed values: public, internal,  confidential, restricted, pii, phi
  Owner               = ""                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-auds" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-auds" as default for infra projects.
  access-team         = "devops"                  # Specify 'access-team', else set as devops.
  access-project      = "infra"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```hcl
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```hcl
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```
> **NOTE:** Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.

**Step 8: Apply the configuration, and push the feature branch**
Execute below commands:

```bash
$ make ensure_pre_commit
$ make fmt
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 9: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch -> master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time. save it.**

 <a href="#top">Back to top</a>
 
---

## 30. Deploy AWS batch job queues [DATAMESH] 

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit ` command from the root of the repo.
4. The below code creates the job queues for the `auds` datamesh team where access-project=auds, and access-team=datateam. Change the access-project to match your own account.

**Step 1: Create a directory for the new configuration under the root of the account repo.**

Repeat this entire section for each enabled region for the account.

To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1' at the root level of the directory. 

```
$ mkdir -p aue1/aws-batch/job-queues
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```
$ cd aue1/aws-batch/job-queues
$ ${pylect-infra-terraform update}
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

> **Notes:**
>  1. Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required.
>  2. If no locals need to be added, skip this step.

```
# Additional locals
locals {}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

Create the ec2 batch queue.

**ec2_queue.tf**


```
module "batch_queue_ec2" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/aws-batch/job-queue?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  state    = "ENABLED" # ENABLED|DISABLED
  priority = 500
  compute_environments = [
    "arn:aws:batch:${var.region}:${local.aws_account_id}:compute-environment/${local.account_attr}-batch-ec2-01-compute-env",
    "arn:aws:batch:${var.region}:${local.aws_account_id}:compute-environment/${local.account_attr}-batch-ec2-02-compute-env"
  ]

  # Labels
  #enabled        = true
  name           = "batch"
  access_project = "infra"
  environment    = "mgmt"
  attributes     = ["ec2"]
  label_order    = ["access_project", "attributes"]

  tags = local.tags

}


module "batch_queue_ec2_100" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/aws-batch/job-queue?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  state    = "ENABLED" # ENABLED|DISABLED
  priority = 100
  compute_environments = [
    "arn:aws:batch:${var.region}:${local.aws_account_id}:compute-environment/${local.account_attr}-batch-ec2-01-compute-env",
    "arn:aws:batch:${var.region}:${local.aws_account_id}:compute-environment/${local.account_attr}-batch-ec2-02-compute-env"
  ]

  # Labels
  #enabled        = true
  name           = "batch"
  access_project = "infra"
  environment    = "mgmt"
  attributes     = ["ec2", "p100"]
  label_order    = ["access_project", "attributes"]

  tags = local.tags

}

module "batch_queue_ec2_200" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/aws-batch/job-queue?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  state    = "ENABLED" # ENABLED|DISABLED
  priority = 200
  compute_environments = [
    "arn:aws:batch:${var.region}:${local.aws_account_id}:compute-environment/${local.account_attr}-batch-ec2-01-compute-env",
    "arn:aws:batch:${var.region}:${local.aws_account_id}:compute-environment/${local.account_attr}-batch-ec2-02-compute-env"
  ]

  # Labels
  #enabled        = true
  name           = "batch"
  access_project = "infra"
  environment    = "mgmt"
  attributes     = ["ec2", "p200"]
  label_order    = ["access_project", "attributes"]

  tags = local.tags

}

module "batch_queue_ec2_spot" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/aws-batch/job-queue?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  state                = "ENABLED" # ENABLED|DISABLED
  priority             = 500
  compute_environments = ["arn:aws:batch:${var.region}:${local.aws_account_id}:compute-environment/${local.account_attr}-batch-ec2-spot-compute-env"]

  # Labels
  #enabled        = true
  name           = "batch"
  access_project = "infra"
  environment    = "mgmt"
  attributes     = ["ec2-spot"]
  label_order    = ["access_project", "attributes"]

  tags = local.tags

}
```

Define the fargate batch queue

**fargate_queue.tf**

```
module "batch_queue_fargate" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/aws-batch/job-queue?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  state    = "ENABLED" # ENABLED|DISABLED
  priority = 500
  compute_environments = [
    "arn:aws:batch:${var.region}:${local.aws_account_id}:compute-environment/${local.account_attr}-batch-fargate-01-compute-env",
    "arn:aws:batch:${var.region}:${local.aws_account_id}:compute-environment/${local.account_attr}-batch-fargate-02-compute-env"
  ]

  # Labels
  #enabled        = true
  name           = "batch"
  access_project = "infra"
  environment    = "mgmt"
  attributes     = ["fargate"]
  label_order    = ["access_project", "attributes"]

  tags = local.tags

}

module "batch_queue_fargate100" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/aws-batch/job-queue?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  state    = "ENABLED" # ENABLED|DISABLED
  priority = 100
  compute_environments = [
    "arn:aws:batch:${var.region}:${local.aws_account_id}:compute-environment/${local.account_attr}-batch-fargate-01-compute-env",
    "arn:aws:batch:${var.region}:${local.aws_account_id}:compute-environment/${local.account_attr}-batch-fargate-02-compute-env"
  ]

  # Labels
  #enabled        = true
  name           = "batch"
  access_project = "infra"
  environment    = "mgmt"
  attributes     = ["fargate", "p100"]
  label_order    = ["access_project", "attributes"]

  tags = local.tags

}
module "batch_queue_fargate200" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/aws-batch/job-queue?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  state    = "ENABLED" # ENABLED|DISABLED
  priority = 200
  compute_environments = [
    "arn:aws:batch:${var.region}:${local.aws_account_id}:compute-environment/${local.account_attr}-batch-fargate-01-compute-env",
    "arn:aws:batch:${var.region}:${local.aws_account_id}:compute-environment/${local.account_attr}-batch-fargate-02-compute-env"
  ]

  # Labels
  #enabled        = true
  name           = "batch"
  access_project = "infra"
  environment    = "mgmt"
  attributes     = ["fargate", "p200"]
  label_order    = ["access_project", "attributes"]

  tags = local.tags

}
module "batch_queue_fargate_spot" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/aws-batch/job-queue?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  state                = "ENABLED" # ENABLED|DISABLED
  priority             = 500
  compute_environments = ["arn:aws:batch:${var.region}:${local.aws_account_id}:compute-environment/${local.account_attr}-batch-fargate-spot-compute-env"]

  # Labels
  #enabled        = true
  name           = "batch"
  access_project = "infra"
  environment    = "mgmt"
  attributes     = ["fargate-spot"]
  label_order    = ["access_project", "attributes"]

  tags = local.tags

}
```

**main.tf**

Confirm all booleans for remote state match the following, in `main.tf`:

```
locals {
  ##...

  # Turn on and off to retrieve remote state from backend
  use_secure_baseline_remote_state = false
  use_vpc_remote_state             = true
  use_sg_remote_state              = true
  use_bootstrap_remote_state       = true
  use_iam_policy_remote_state      = false
}
```

**Step 5: Update Outputs**

Ensure all customer managed keys are available to other modules.

**outputs.tf**

```
# Add additional outputs below
output "batch_job_queue" {
  description = "Batch Job Queue"
  value = {
    (module.batch_queue_fargate.name[0])      = module.batch_queue_fargate[*].*
    (module.batch_queue_fargate_spot.name[0]) = module.batch_queue_fargate_spot[*].*
    (module.batch_queue_ec2.name[0])          = module.batch_queue_ec2[*].*
    (module.batch_queue_ec2_spot.name[0])     = module.batch_queue_ec2_spot[*].*
    (module.batch_queue_ec2_100.name[0])      = module.batch_queue_ec2_100[*].*
    (module.batch_queue_ec2_200.name[0])      = module.batch_queue_ec2_200[*].*
    (module.batch_queue_fargate100.name[0])   = module.batch_queue_fargate100[*].*
    (module.batch_queue_fargate200.name[0])   = module.batch_queue_fargate200[*].*
    # (module.s3cross_region_ec2.name[0])       = module.s3cross_region_ec2[*].*
    # (module.s3cross_region_ec2_spot.name[0])  = module.s3cross_region_ec2_spot[*].*
  }
}
```

**Step 6: Update `terraform.tfvars`**

Update the required tags: 

* `Agency`
* `Client`
* `costcenter`
* `access-team`
* `access-project`
* `Data_Classification`

Make sure to use the correct costcenter and access-project.

```hcl
common_tags = {
  ##... omitted

  Data_Classification = "internal"               # Definition of data classification and confidentiality, allowed values: public, internal,  confidential, restricted, pii, phi
  Owner               = ""                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-auds" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-auds" as default for infra projects.
  access-team         = "devops"                  # Specify 'access-team', else set as devops.
  access-project      = "infra"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```hcl
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```hcl
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```
> **NOTE:** Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.

**Step 8: Apply the configuration, and push the feature branch**
Execute below commands:

```bash
$ make ensure_pre_commit
$ make fmt
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 9: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch -> master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.

 <a href="#top">Back to top</a>
 
---

## 31. Set up the hosting service for Apache Airflow [DATAMESH]

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit ` command from the root of the repo.
4. The below code creates the MWAA environment for the `auds` datamesh team where access-project=auds, and access-team=datateam. Change the access-project to match your own account.

**Step 1: Create a directory for the new configuration under the root of the account repo.**

Repeat this entire section for each enabled region for the account.

To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1' at the root level of the directory. 

```
$ mkdir -p aue1/env-dev/services/airflow
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```
$ cd aue1/env-dev/services/airflow 
$ ${pylect-infra-terraform update}
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

> **Notes:**
>  1. Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required.
>  2. If no locals need to be added, skip this step.

```
# Additional locals
locals {}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

Create the managed airflow service.

**airflow01.tf** 

```
module "airflow01" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/airflow?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  name           = "airflow"
  access_project = local.access_project
  attributes     = ["01"]
  label_order    = ["access_project", "name", "attributes"]
  tags           = local.standard_tags_map.default


  airflow_version    = "2.2.2"
  dag_s3_path        = "dags"
  source_bucket_arn  = module.airflow1_dags_bucket.s3_bucket_arn
  execution_role_arn = "arn:aws:iam::${local.aws_account_id}:role/service-role/${local.account_attr}-${local.access_project}-${var.region_abbr}-airflow-execution-role"
  max_workers        = 2
  min_workers        = 1
  environment_class  = "mw1.medium"
  #plugins_s3_path      = "plugins.zip"
  #requirements_s3_path = "requirements.txt"
  security_group_ids = [data.terraform_remote_state.sg[0].outputs.security_groups.main.airflow["mixed"]]
  subnet_ids         = [data.terraform_remote_state.vpc[0].outputs.dev_private_az_subnet_ids["us-east-1a"], data.terraform_remote_state.vpc[0].outputs.dev_private_az_subnet_ids["us-east-1b"]]

  weekly_maintenance_window_start = "SUN:01:00"
  # The airflow_configuration_options parameter specifies airflow override options
  airflow_configuration_options = {
    "celery.sync_parallelism"           = 1
    "celery.worker_autoscale"           = "4,4"
    "core.dag_concurrency"              = 20
    "core.dag_file_processor_timeout"   = 600
    "core.dagbag_import_timeout"        = 600
    "core.parallelism"                  = 20
    "core.sql_alchemy_max_overflow"     = -1
    "core.sql_alchemy_pool_size"        = 0
    "core.store_dag_code"               = "false"
    "scheduler.processor_poll_interval" = 10
  }

  logging_configuration = [{

    dag_enabled   = true,
    dag_log_level = "ERROR",

    scheduler_logs_enabled = true,
    scheduler_log_level    = "ERROR",

    task_logs_enabled = true,
    task_log_level    = "INFO",

    webserver_logs_enabled = true,
    webserver_log_level    = "ERROR",

    worker_logs_enabled = true,
    worker_log_level    = "ERROR",
  }]
  depends_on = [
    module.airflow1_dags_bucket,
    aws_s3_bucket_object.dags1,
  ]
}

##Use s3 module to create new bucket and map output airflow module.

module "airflow1_dags_bucket" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/s3-bucket?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  bucket = "${var.account_attr}-${var.region_abbr}-${local.access_project}-mwaa-01"


  # S3 Server Logging, Inventory  and Analytics
  logging_bucket      = local.logging_bucket
  analytics_bucket    = local.analytics_bucket
  inventory_bucket    = local.inventory_bucket
  versioning          = { enabled = true }
  enable_s3_inventory = true
  enable_s3_analytics = true

  # Tags
  tags = merge(local.standard_tags_map["s3"], {
    Owner                 = "devops@annalect.com"
    Environment           = var.common_tags["Environment"]
    Client                = var.common_tags["Client"]
    Agency                = var.common_tags["Agency"]
    costcenter            = var.common_tags["costcenter"]
    access-project        = var.common_tags["access-project"]
    access-team           = var.common_tags["access-team"]
    "Data Classification" = var.common_tags["Data_Classification"]
  })

  # Bucket Policy
  bucket_policies = []

  # Lifecycle Rules
  lifecycle_rule = [{
    id                                     = "Delete old incomplete multi-part uploads"
    enabled                                = true
    abort_incomplete_multipart_upload_days = 7
  }]
}


resource "aws_s3_bucket_object" "dags1" {
  provider = aws.aue1
  for_each = fileset("dags/", "*.py")
  bucket   = module.airflow1_dags_bucket.s3_bucket_id
  key      = "dags/${each.value}"
  source   = "dags/${each.value}"
  etag     = filemd5("dags/${each.value}")
}
```

Add the python definition file in the DAGs folder

```
$ mkdir dags && cd dags
```

Define the DAGs in the python file within the subfolder `dags`

**tutorial.py**

```
# -*- coding: utf-8 -*-
#
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

"""
### Tutorial Documentation
Documentation that goes along with the Airflow tutorial located
[here](https://airflow.apache.org/tutorial.html)
"""
# [START tutorial]
# [START import_module]
from datetime import timedelta

# The DAG object; we'll need this to instantiate a DAG
from airflow import DAG
# Operators; we need this to operate!
from airflow.operators.bash_operator import BashOperator
from airflow.utils.dates import days_ago

# [END import_module]

# [START default_args]
# These args will get passed on to each operator
# You can override them on a per-task basis during operator initialization
default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': days_ago(2),
    'email': ['airflow@example.com'],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
    # 'queue': 'bash_queue',
    # 'pool': 'backfill',
    # 'priority_weight': 10,
    # 'end_date': datetime(2016, 1, 1),
    # 'wait_for_downstream': False,
    # 'dag': dag,
    # 'sla': timedelta(hours=2),
    # 'execution_timeout': timedelta(seconds=300),
    # 'on_failure_callback': some_function,
    # 'on_success_callback': some_other_function,
    # 'on_retry_callback': another_function,
    # 'sla_miss_callback': yet_another_function,
    # 'trigger_rule': 'all_success'
}
# [END default_args]

# [START instantiate_dag]
dag = DAG(
    'tutorial',
    default_args=default_args,
    description='A simple tutorial DAG',
    schedule_interval=timedelta(days=1),
)
# [END instantiate_dag]

# t1, t2 and t3 are examples of tasks created by instantiating operators
# [START basic_task]
t1 = BashOperator(
    task_id='print_date',
    bash_command='date',
    dag=dag,
)

t2 = BashOperator(
    task_id='sleep',
    depends_on_past=False,
    bash_command='sleep 5',
    retries=3,
    dag=dag,
)
# [END basic_task]

# [START documentation]
dag.doc_md = __doc__

t1.doc_md = """\
#### Task Documentation
You can document your task using the attributes `doc_md` (markdown),
`doc` (plain text), `doc_rst`, `doc_json`, `doc_yaml` which gets
rendered in the UI's Task Instance Details page.
![img](http://montcs.bloomu.edu/~bobmon/Semesters/2012-01/491/import%20soul.png)
"""
# [END documentation]

# [START jinja_template]
templated_command = """
{% for i in range(5) %}
    echo "{{ ds }}"
    echo "{{ macros.ds_add(ds, 7)}}"
    echo "{{ params.my_param }}"
{% endfor %}
"""

t3 = BashOperator(
    task_id='templated',
    depends_on_past=False,
    bash_command=templated_command,
    params={'my_param': 'Parameter I passed in'},
    dag=dag,
)
# [END jinja_template]

t1 >> [t2, t3]
# [END tutorial]
```

**main.tf**

Confirm all booleans for remote state match the following, in `main.tf`:

```
locals {
  ##...

  # Turn on and off to retrieve remote state from backend
  use_secure_baseline_remote_state = true
  use_vpc_remote_state             = true
  use_sg_remote_state              = true
  use_bootstrap_remote_state       = false
  use_iam_policy_remote_state      = false
}
```

**Step 5: Update Outputs**

Ensure all customer managed keys are available to other modules.

**outputs.tf**

```
# Add additional outputs below
output "airflow" {
  description = "Apache Airflow (MWAA)"
  value = {
    ("airflow01") = {
      arn : module.airflow01.arn,
      service_role : module.airflow01.service_role_arn,
      webserver_url : module.airflow01.webserver_url,
      source_bucket_arn : try(module.airflow01.source_bucket_arn, "")
      name : try(module.airflow01.name, "")
    }
  }
}

output "airflow_dags_bucket" {
  description = "Bucket Details"
  value = {
    (module.airflow1_dags_bucket.s3_bucket_id) = {
      name : module.airflow1_dags_bucket.s3_bucket_id,
      region : module.airflow1_dags_bucket.s3_bucket_region,
      arn : module.airflow1_dags_bucket.s3_bucket_arn,
    },
  }
}
```

**Step 6: Update `terraform.tfvars`**

Update the required tags: 

* `Owner`
* `Agency`
* `Client`
* `costcenter`
* `access-team`
* `access-project`
* `Data_Classification`

Make sure to use the correct costcenter and access-project.

```hcl
common_tags = {
  ##... omitted
  Data_Classification = "internal"               # Definition of data classification and confidentiality, allowed values: public, internal,  confidential, restricted, pii, phi
  Owner               = "datateam"                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-auds" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-auds" as default for infra projects.
  access-team         = "datateam"                  # Specify 'access-team', else set as devops.
  access-project      = "auds"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```hcl
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```hcl
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```
> **NOTE:** Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.

**Step 8: Apply the configuration, and push the feature branch**
Execute below commands:

```bash
$ make ensure_pre_commit
$ make fmt
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 9: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch -> master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.

 <a href="#top">Back to top</a>
 
---

## 32. Deploy the managed emr solution [DATAMESH]

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit ` command from the root of the repo.
4. The below code will prepare an emr serverless setup for the `auds` datamesh team where access-project=auds, and access-team=datateam. Change the access-project to match your own account.

**Step 1: Create a directory for the new configuration under the root of the account repo.**

Repeat this entire section for each enabled region for the account.

To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1' at the root level of the directory.

```
$ mkdir -p aue1/env-dev/services/emr-serverless
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```
$ cd aue1/env-dev/services/emr-serverless
$ ${pylect-infra-terraform update}
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

> **Notes:**
>  1. Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required.
>  2. If no locals need to be added, skip this step.

```
# Additional locals
locals {}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

Define the terraform configuration for Apache Hive.

**emr_serverless_hive.tf**

```
module "emr_serverless_hive" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/emr-serverless?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  # labels
  name           = "serverless-emr"
  access_project = local.access_project
  attributes     = ["serverless", "hive"]
  label_order    = ["access_project", "attributes"]
  tags           = local.tags

  release_label     = "emr-6.7.0"
  applicatioin_type = "hive"

  pre_initialized_capacity_configuration = {
    capacity_type = "HiveDriver"
    worker_count  = 1
    worker_configuration = {
      cpu    = "4 vCPU"
      memory = "16 GB"
      disk   = "20 GB"
    }
  }

  maximum_capacity_worker_configuration = {
    cpu    = "64 vCPU"
    memory = "128 GB"
    disk   = "1024 GB"
  }

  subnet_ids = data.terraform_remote_state.vpc[0].outputs["private_az_subnet_ids"]["dev"]
  security_group_ids = [
    data.terraform_remote_state.sg[0].outputs["security_groups"]["main"]["ecs"]["dev"]
  ]
  auto_start             = true
  auto_stop              = true
  auto_stop_idle_timeout = 15

}


module "emr_serverless_spark" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/emr-serverless?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }


  # labels
  name           = "serverless-emr"
  access_project = local.access_project
  attributes     = ["serverless", "spark"]
  label_order    = ["access_project", "attributes"]
  tags           = local.tags

  release_label     = "emr-6.7.0"
  applicatioin_type = "spark"

  pre_initialized_capacity_configuration = {
    capacity_type = "DRIVER"
    worker_count  = 2
    worker_configuration = {
      cpu    = "4 vCPU"
      memory = "16 GB"
      disk   = "20 GB"
    }
  }


  maximum_capacity_worker_configuration = {
    cpu    = "64 vCPU"
    memory = "128 GB"
    disk   = "1024 GB"
  }

  subnet_ids = data.terraform_remote_state.vpc[0].outputs["private_az_subnet_ids"]["dev"]
  security_group_ids = [
    data.terraform_remote_state.sg[0].outputs["security_groups"]["main"]["ecs"]["dev"]
  ]
  auto_start             = true
  auto_stop              = true
  auto_stop_idle_timeout = 15

}
```

Set up the IDE for EMR.

**emr_studio.tf**

```
module "emr_studio" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/emr-studio?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  description = "upper(${local.access_project}) EMR Studio"

  # labels
  name           = "emr-studio"
  access_project = local.access_project
  attributes     = ["emr", "studio"]
  label_order    = ["access_project", "attributes"]
  tags           = local.tags

  emr_studio_auth_mode         = "IAM"
  emr_studio_workspace_storage = "s3://${var.account_attr}-${var.region_abbr}-emrstudio-workspace-storage/emrstudio"
  engine_security_group_id     = tostring(data.terraform_remote_state.sg[0].outputs.security_groups.main.emr["studio"])
  emr_studio_service_role      = "arn:aws:iam::${local.aws_account_id}:role/service-role/${var.account_attr}-infra-emrstudio-service-role"
  emr_studio_subnet_ids        = data.terraform_reLLmote_state.vpc[0].outputs["private_az_subnet_ids"]["dev"]
  vpc_id                       = tostring(data.terraform_remote_state.vpc[0].outputs.vpcs.main.id)
  workspace_security_group_id  = tostring(data.terraform_remote_state.sg[0].outputs.security_groups.main.emr["workspace"])
}
```

**main.tf**

Confirm all booleans for remote state match the following, in `main.tf`:

```
locals {
  ##...

  # Turn on and off to retrieve remote state from backend
  use_secure_baseline_remote_state = false
  use_vpc_remote_state             = true
  use_sg_remote_state              = true
  use_bootstrap_remote_state       = false
  use_iam_policy_remote_state      = false
}
```

**Step 5: Update Outputs**

Ensure all customer managed keys are available to other modules.

**outputs.tf**

```
# Add additional outputs below
output "serverless_emr_apps" {
  description = "Serverless EMR Applications"
  value = {
    (module.emr_serverless_hive.id)  = { arn = try(module.emr_serverless_hive.arn, []) }
    (module.emr_serverless_spark.id) = { arn = try(module.emr_serverless_spark.arn, []) }
  }
}
```

**Step 6: Update `terraform.tfvars`**

Update the required tags: 

* `Agency`
* `Client`
* `costcenter`
* `access-team`
* `access-project`
* `Data_Classification`

Make sure to use the correct costcenter and access-project.

```hcl
common_tags = {
  ##... omitted

  Data_Classification = "internal"               # Definition of data classification and confidentiality, allowed values: public, internal,  confidential, restricted, pii, phi
  Owner               = "datateam"                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-auds" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-auds" as default for infra projects.
  access-team         = "datateam"                  # Specify 'access-team', else set as devops.
  access-project      = "auds"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```hcl
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```hcl
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```
> **NOTE:** Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.

**Step 8: Apply the configuration, and push the feature branch**
Execute below commands:

```bash
$ make ensure_pre_commit
$ make fmt
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 9: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch -> master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.

 <a href="#top">Back to top</a>
 
---

## 33. Deploy project specific s3 buckets [DATAMESH]

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit ` command from the root of the repo.
4. The below code creates the s3 buckets for the `auds` datamesh team where access-project=auds, and access-team=datateam. Change the access-project to match your own account.

**Step 1: Create a directory for the new configuration under the root of the account repo.**

Repeat this entire section for each enabled region for the account.

To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1' at the root level of the directory. Replace ${access-project} with your AWS account access project (ex: `auds`).

```
$ mkdir -p aue1/s3buckets/projects/${access-project}
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```
$ cd aue1/s3buckets/projects/${access-project}
$ ${pylect-infra-terraform update}
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

> **Notes:**
>  1. Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required.
>  2. If no locals need to be added, skip this step.

```
# Additional locals
locals {}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

Deploy the athena s3 bucket.

**athena.tf**

```
module "athena" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/s3-bucket?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  bucket = "${var.account_attr}-${var.region_abbr}-${local.access_project}-athena"

  # S3 Logging, Inventory and Analytics
  logging_bucket      = local.logging_bucket
  analytics_bucket    = local.analytics_bucket
  inventory_bucket    = local.inventory_bucket
  enable_s3_inventory = true
  enable_s3_analytics = true

  # Tagging (tags to override)
  tags = merge(local.standard_tags_map["s3"], {
    Owner                 = "bharath.shivaiah@annalect.com"
    Agency                = var.common_tags["Agency"]
    Client                = var.common_tags["Client"]
    "Data Classification" = var.common_tags["Data_Classification"]
    "Data Retention"      = "s3_lifecycle_rule"
    costcenter            = var.common_tags["costcenter"]
    Compliance            = var.common_tags["Compliance"]
    Scope                 = var.common_tags["Scope"]
  })

  # Lifecycle Rules
  lifecycle_rule = [{
    id                                     = "Delete old incomplete multi-part uploads"
    enabled                                = true
    abort_incomplete_multipart_upload_days = 7
    },
    {
      id      = "IntelligentTiering"
      enabled = true
      prefix  = ""
      filter  = {}
      transition = [{
        days          = 30
        storage_class = "INTELLIGENT_TIERING"
      }]
      expiration = {}
      # noncurrent_version_expiration = {}
  }]

}
```

Create the s3 buckets for the various data layers.

**data_layers_buckets.tf**

```
module "lz" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/s3-bucket?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  bucket = "${var.account_attr}-${var.region_abbr}-${local.access_project}-lz"

  # S3 Logging, Inventory and Analytics
  logging_bucket      = local.logging_bucket
  analytics_bucket    = local.analytics_bucket
  inventory_bucket    = local.inventory_bucket
  enable_s3_inventory = true
  enable_s3_analytics = true

  # Tagging (tags to override)
  tags = merge(local.standard_tags_map["s3"], {
    Owner                 = "mukesh.ranjan@annalect.com"
    Agency                = var.common_tags["Agency"]
    Client                = var.common_tags["Client"]
    "Data Classification" = var.common_tags["Data_Classification"]
    "Data Retention"      = "s3_lifecycle_rule"
    costcenter            = var.common_tags["costcenter"]
    Compliance            = var.common_tags["Compliance"]
    Scope                 = var.common_tags["Scope"]
  })

  bucket_policies = []

  # Lifecycle Rules
  lifecycle_rule = [{
    id                                     = "Delete old incomplete multi-part uploads"
    enabled                                = true
    abort_incomplete_multipart_upload_days = 7
    },
    {
      id      = "IntelligentTiering"
      enabled = true
      prefix  = ""
      filter  = {}
      transition = [{
        days          = 30
        storage_class = "INTELLIGENT_TIERING"
      }]
      expiration = {}
      # noncurrent_version_expiration = {}
  }]

}

module "raw" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/s3-bucket?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  bucket = "${var.account_attr}-${var.region_abbr}-${local.access_project}-raw"

  # S3 Logging, Inventory and Analytics
  logging_bucket      = local.logging_bucket
  analytics_bucket    = local.analytics_bucket
  inventory_bucket    = local.inventory_bucket
  enable_s3_inventory = true
  enable_s3_analytics = true

  # Tagging (tags to override)
  tags = merge(local.standard_tags_map["s3"], {
    Owner                 = "mukesh.ranjan@annalect.com"
    Agency                = var.common_tags["Agency"]
    Client                = var.common_tags["Client"]
    "Data Classification" = var.common_tags["Data_Classification"]
    "Data Retention"      = "s3_lifecycle_rule"
    costcenter            = var.common_tags["costcenter"]
    Compliance            = var.common_tags["Compliance"]
    Scope                 = var.common_tags["Scope"]
  })

  bucket_policies = []

  # Lifecycle Rules
  lifecycle_rule = [{
    id                                     = "Delete old incomplete multi-part uploads"
    enabled                                = true
    abort_incomplete_multipart_upload_days = 7
    },
    {
      id      = "IntelligentTiering"
      enabled = true
      prefix  = ""
      filter  = {}
      transition = [{
        days          = 30
        storage_class = "INTELLIGENT_TIERING"
      }]
      expiration = {}
      # noncurrent_version_expiration = {}
  }]
}

module "curated" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/s3-bucket?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  bucket = "${var.account_attr}-${var.region_abbr}-${local.access_project}-curated"

  # S3 Logging, Inventory and Analytics
  logging_bucket      = local.logging_bucket
  analytics_bucket    = local.analytics_bucket
  inventory_bucket    = local.inventory_bucket
  enable_s3_inventory = true
  enable_s3_analytics = true

  # Tagging (tags to override)
  tags = merge(local.standard_tags_map["s3"], {
    Owner                 = "mukesh.ranjan@annalect.com"
    Agency                = var.common_tags["Agency"]
    Client                = var.common_tags["Client"]
    "Data Classification" = var.common_tags["Data_Classification"]
    "Data Retention"      = "s3_lifecycle_rule"
    costcenter            = var.common_tags["costcenter"]
    Compliance            = var.common_tags["Compliance"]
    Scope                 = var.common_tags["Scope"]
  })

  bucket_policies = []

  # Lifecycle Rules
  lifecycle_rule = [{
    id                                     = "Delete old incomplete multi-part uploads"
    enabled                                = true
    abort_incomplete_multipart_upload_days = 7
    },
    {
      id      = "IntelligentTiering"
      enabled = true
      prefix  = ""
      filter  = {}
      transition = [{
        days          = 30
        storage_class = "INTELLIGENT_TIERING"
      }]
      expiration = {}
      # noncurrent_version_expiration = {}
  }]

}
```

**main.tf**

Confirm all booleans for remote state match the following, in `main.tf`:

```
locals {
  ##...

  # Turn on and off to retrieve remote state from backend
  use_secure_baseline_remote_state = true
  use_vpc_remote_state             = false
  use_sg_remote_state              = false
  use_bootstrap_remote_state       = false
  use_iam_policy_remote_state      = false
}
```

**Step 5: Update Outputs**

Ensure all customer managed keys are available to other modules.

**outputs.tf**

```
# Add additional outputs below
output "buckets" {
  description = "Bucket Details"
  value = {
    (module.athena.s3_bucket_id) = {
      name : module.athena.s3_bucket_id,
      region : module.athena.s3_bucket_region
    },
    (module.lz.s3_bucket_id) = {
      name : module.lz.s3_bucket_id,
      region : module.lz.s3_bucket_region
    },
    (module.raw.s3_bucket_id) = {
      name : module.raw.s3_bucket_id,
      region : module.raw.s3_bucket_region
    },
    (module.curated.s3_bucket_id) = {
      name : module.curated.s3_bucket_id,
      region : module.curated.s3_bucket_region
    },
  }
}
```

**Step 6: Update `terraform.tfvars`**

Update the required tags: 

* `Owner`
* `Agency`
* `Client`
* `costcenter`
* `access-team`
* `access-project`
* `Data_Classification` 

Make sure to use the correct costcenter and access-project.

```hcl
common_tags = {
  ##... omitted

  Data_Classification = "internal"               # Definition of data classification and confidentiality, allowed values: public, internal,  confidential, restricted, pii, phi
  Owner               = "datateam"                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-auds" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-auds" as default for infra projects.
  access-team         = "datateam"                  # Specify 'access-team', else set as devops.
  access-project      = "auds"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```hcl
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```hcl
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```
> **NOTE:** Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.

**Step 8: Apply the configuration, and push the feature branch**
Execute below commands:

```bash
$ make ensure_pre_commit
$ make fmt
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 9: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch -> master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.

 <a href="#top">Back to top</a>
 
---

## 34. Deploy team specific s3 buckets [DATAMESH]

Prerequisites:

1. Clone a 'terraform root' repository (project) of respective account to your local computer.
2. Create a new feature branch from the `master` branch of the respective account specific repository.
3. Install and activate the pre-commit hooks by executing `make ensure_pre_commit ` command from the root of the repo.
4. The below code creates the s3 buckets for the `auds` datamesh team where access-project=auds, and access-team=datateam. Change the access-project to match your own account.

**Step 1: Create a directory for the new configuration under the root of the account repo.**

Repeat this entire section for each enabled region for the account.

To provision the resources in us-east-1 region, create the directory with region_abbr named 'aue1' at the root level of the directory. Replace ${access-project} with your AWS account access project (ex: `auds`).

```
$ mkdir -p aue1/s3buckets/teams/datateam
```

**Step 2: Change into the directory created in above step and run the `pylect-infra-terraform` commands to generate the initial terraform configuration files.**

```
$ cd aue1/s3buckets/teams/datateam
$ ${pylect-infra-terraform update}
$ pylect-infra-terraform start
```

**Step 3: Declare all the local variables in the locals block.**

> **Notes:**
>  1. Add additional variables as local block to `locals.tf` and replace the local variable value with actual value if required.
>  2. If no locals need to be added, skip this step.

```
# Additional locals
locals {}
```

**Step 4: In the Terraform directory, create the specified Terraform configuration files. Paste the configuration below into respective file and save it.**

Deploy the s3 buckets required for datateam for datamesh projects in each region.

**bucket.tf**

```
module "datateam" {
  source    = "git@bitbucket.org:annalect/terraform_aws_modules//modules/s3-bucket?ref=tags/v3.2.110"
  providers = { aws = aws.aue1 }

  bucket = "${var.account_attr}-${var.region_abbr}-${local.access_project}-datateam"

  # S3 Logging, Inventory and Analytics
  logging_bucket      = local.logging_bucket
  analytics_bucket    = local.analytics_bucket
  inventory_bucket    = local.inventory_bucket
  enable_s3_inventory = true
  enable_s3_analytics = true

  # Tagging (tags to override)
  tags = merge(local.standard_tags_map["s3"], {
    Owner                 = "mukesh.ranjan@annalect.com"
    Agency                = var.common_tags["Agency"]
    Client                = var.common_tags["Client"]
    "Data Classification" = var.common_tags["Data_Classification"]
    "Data Retention"      = "s3_lifecycle_rule"
    costcenter            = var.common_tags["costcenter"]
    Compliance            = var.common_tags["Compliance"]
    Scope                 = var.common_tags["Scope"]
  })

  # Lifecycle Rules
  lifecycle_rule = [{
    id                                     = "Delete old incomplete multi-part uploads"
    enabled                                = true
    abort_incomplete_multipart_upload_days = 7
    },
    {
      id      = "IntelligentTiering"
      enabled = true
      prefix  = ""
      filter  = {}
      transition = [{
        days          = 30
        storage_class = "INTELLIGENT_TIERING"
      }]
      expiration = {}
      # noncurrent_version_expiration = {}
  }]

}
```

**main.tf**

Confirm all booleans for remote state match the following, in `main.tf`:

```
locals {
  ##...

  # Turn on and off to retrieve remote state from backend
  use_secure_baseline_remote_state = true
  use_vpc_remote_state             = false
  use_sg_remote_state              = false
  use_bootstrap_remote_state       = false
  use_iam_policy_remote_state      = false
}
```

**Step 5: Update Outputs**

Ensure all customer managed keys are available to other modules.

**outputs.tf**

```
# Add additional outputs below
output "buckets" {
  description = "Bucket Details"
  value = {
    (module.datateam.s3_bucket_id) = {
      name : module.datateam.s3_bucket_id,
      region : module.datateam.s3_bucket_region
    },
  }
}
```

**Step 6: Update `terraform.tfvars`**

Update the required tags: 

* `Owner`
* `Agency`
* `Client`
* `costcenter`
* `access-team`
* `access-project`
* `Data_Classification` 

Make sure to use the correct costcenter and access-project.

```hcl
common_tags = {
  ##... omitted

  Data_Classification = "internal"               # Definition of data classification and confidentiality, allowed values: public, internal,  confidential, restricted, pii, phi
  Owner               = "datateam"                  # Owner/Requester of the resource
  Owner2              = ""                        # Owner/Requester of the resource
  Agency              = "ANN"                     # Allowed values: ANN, ADY, BBDO, DDB, HAS, OMD, PHD, RES, TBWA
  Client              = "internal"                # Specify client name of the Agency else 'internal'.
  costcenter          = "annalect-internal-auds" # Specify 'costcenter' for tracking the resource billing. It should follow <agency>-<client>-<project> format. Use "annalect-internal-auds" as default for infra projects.
  access-team         = "datateam"                  # Specify 'access-team', else set as devops.
  access-project      = "auds"                   # Specify 'access-project' name, else set as "infra".
}
```

**Step 7: Replace the roles in `backend.hcl` and `terraform.tfvars`**

Replace the currently existing role for `role_arn` in `backend.hcl` 

```hcl
# backend.hcl

##...

- role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
+ role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

##...
```

Replace the `assume_role_arn`, under `trusting_account` in `terraform.tfvars`.

```hcl
# terraform.tfvars

##...

trusting_account { 
  ##...

  - assume_role_arn = "arn:aws:iam::856889162721:role/admin/ann17-private-dev-shared-readonly"
  + assume_role_arn = "arn:aws:iam::856889162721:role/accountonboardingadmin"

  ##...
}
```

> **Note:**
> Save the roles you just replaced, as you will restore the roles to what previously existed before pushing the feature branch and creating a MR.


**Step 8: Apply the configuration, and push the feature branch**
Execute below commands:

```bash
$ make ensure_pre_commit
$ make fmt
$ make prep
$ make test # rerun untill all tests pass
$ make plan
```

Get the code reviewed, then apply the configuration.

```bash
$ make apply
```

**Step 9: Replace the temporary roles and create a MR for the feature branch**

Before pushing the repo to the remote branch, restore the roles in `backend.hcl` and in `terraform.tfvars`. Do not rerun `make apply` after making these changes.

Push and create a MR for the feature branch we just created. Merge should be feature branch -> master. Notify DevOps to get the MR approved/modified ASAP before it gets lost to time.


 <a href="#top">Back to top</a>
 
---

## 35. Cleanup `accountonboardingadmin` role

Begin cleanup of the temporary roles needed to onboard the account.

| :exclamation: Notify [Mocktar Tairou](mailto:mocktar.tairou@annalect.com) to delete the accountonboardingadmin role. |
|-----------------------------------------|

 <a href="#top">Back to top</a>
 
---

# Document Version History

| Version | Changes | Date|
| --- | --- | --- |
| V1.0 | Start of change log | 11-02-22 |
| V1.0.1 | * Fix remote state flags for global/iam-roles/okta. <br />* Comment out unnecessary outputs in KMS CMK step.<br />* Add instructions to run `make test` when updating ann01tioprod KMS and AMI for EC2 step.<br />* Comment out unnecessary outputs in okta saml 2.0 step.<br />* Fix malformed abac6 policy.<br />* Fix EC2 instructions, now state correct keypair name.<br />* Include instructions on how to upload private and public keys.<br />* Fix broken link on step 9 | 11-02-22 |
| V1.0.2 | * Add lambda function: control cloudwatch log retention.<br />* Added steps on how to upload EC2 and EMR keys. | 11-09-22 |
| V1.0.3 | * Add lakeformation_viewer and editor permissions to ABAC policies.<br />* Update source tag for all modules.<br />* Add DevOps members to assume more roles.<br />* Create generic policy to add additional or missing access to project specific resources (NOT team okta role).<br />* Migrated inline_policy_statements where applicable<br />* Replace typo in providers.tf for secure-baseline step| 11-14-22 |
| V1.0.4 | * Fixed missing subscribers list to **cloudtrail_sns_topic_subscribers**<br />* Add explanation for mismatched providers in **secure_baseline**<br />* Allow more roles to access s3readwrite for data engineering team.<br />* Updated vpc privatelink sections<br />* Modified VPC security group ingress rules.<br />* Add note, first step is not done by onboarder. | 11-18-22 |
| V1.0.5 | * Add note to notify user to contact Ayo to add permissions to reviewer (Trenton) | 11-28-22 |
| V1.0.6 | * Address latest SG bug in Step 7. Lock SG source to v3.2.113 until fix is made.<br />* Override default VPC variables in Step 6 so primary subnets are given the right environment name.<br />* Apply fix for lambda function misnaming error.<br />* Disabled cloudtrail_sns_subscription output due to high volume of emails received. | 12-02-22 |