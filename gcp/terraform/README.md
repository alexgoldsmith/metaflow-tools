# A minimal viable Metaflow-on-GCP stack

## What does it do?
It provisions all necessary GCP resources. Main resources are:
* Google Cloud Storage bucket
* GKE cluster
  It will also deploy Metaflow services onto the GKE cluster above.

## Prerequisites

* Install [terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli).
* Install [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl).
* Install [gcloud](https://cloud.google.com/sdk/gcloud) CLI.
* `gcloud auth login` as a GCP account with sufficient privileges to administer necessary resources:
    * GKE
    * Google Cloud Storage bucket
    * Computer networks
    * IAM role assignments
    * ...
    * Note: If you are an owner of your GCP project already you should be good to go.

## Usage
The templates are organized into two modules, `infra` and `services`.

Before you do anything, create a `FILE.tfvars` file with the following content (updating relevant values):

    org_prefix = "<ORG_PREFIX>"
    project = "<GCP_PROJECT_ID>"
    db_generation_number = <DB_GENERATION_NUM>

For `org_prefix`, choose a short and memorable alphanumeric string. It will be used for naming the Google Cloud Storage bucket, whose
name must be globally unique across GCP.

For `GCP_PROJECT_ID`, set the GCP project ID you wish to use.

`DB_GENERATION_NUM` should be updated each time the DB instance is recreated. This helps generate unique DB instance
names.  Backstory: DB instance names cannot repeat within 7 day window on GCP.

You may rename `FILE.tfvars` to a more friendly name appropriate for your project.  E.g. `metaflow.poc.tfvars`.

The variable assignments defined in this file will be passed to `terraform` CLI.

Next, apply the `infra` module (creates GCP resources only).

    terraform apply -target="module.infra" -var-file=FILE.tfvars

Then, apply the `services` module (deploys Metaflow services to GKE)

    terraform apply -target="module.services" -var-file=FILE.tfvars

The step above will output next steps for Metaflow end users.

## (Advanced) Terraform state management
Terraform manages the state of GCP resources in [tfstate](https://www.terraform.io/language/state) files locally by default.

If you plan to maintain the minimal stack for any significant period of time, it is highly
recommended that these state files be stored in cloud storage (e.g. Google Cloud Storage) instead.

Some reasons include:
* More than one person needs to administer the stack (using terraform). Everyone should work off
  a single copy of tfstate.
* You wish to mitigate the risk of data-loss on your local disk.

For more details, see [Terraform docs](https://www.terraform.io/language/settings/backends/configuration).