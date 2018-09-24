---
title: Upgrade
order: 002
---

# {{ page.title }}

This guide explains how to upgrade your Forseti instance.

For version 2.0.0 and later, we will provide upgrade instructions from one minor
version to the next minor version. This means, if you want to upgrade from
version 2.2.0 to 2.4.0, you should follow the upgrade instruction from
version 2.2.0 to 2.3.0 first, and then follow the upgrade instruction from
version 2.3.0 to 2.4.0.  This ensures the upgrade process is easier to manage
and to test.

---

{% capture 1x_upgrade %}

## Important notes

 * Forseti doesn't support data migration from v1 to v2. If you want to keep the v1 data, you will
   need to manually archive the database.
 * [Forseti v2 configuration]({% link _docs/latest/configure/general/index.md %}) is different than v1. You
   can't replace the v2 configuration file with the v1 configuration file.
 * In v2, all resources are inventoried. You won't be able to configure Inventory to include or exclude resources.

## Upgrade to v2

To upgrade your Forseti instance, it's best to run the v1 and v2 instances
side by side:

1. Create a new project.
1. [Install the latest v2 instance]({% link _docs/latest/setup/install.md %}).
1. Copy all the rule files from the v1 bucket to the v2 bucket.

After you have the v2 instance working as expected, you can shut down and
clean up the v1 instance.


### Copying rule files from v1 to v2

You can re-use the rule files defined for your v1 instance in v2 by copying them to the v2 buckets.

To copy all the rule files from the v1 bucket to the v2 bucket, run the following command:

```bash
# Replace <YOUR_V1_BUCKET> with your v1 Forseti Cloud Storage bucket and
# <YOUR_V2_BUCKET> with your v2 Forseti Cloud Storage bucket.

gsutil cp gs://<YOUR_V1_BUCKET>/rules/*.yaml gs://<YOUR_V2_BUCKET>/rules
```

### Archiving your Cloud SQL Database

The best way to archive your database is to use Cloud SQL to [export the data
to a SQL dump file](https://cloud.google.com/sql/docs/mysql/import-export/exporting#mysqldump).

## Difference between v1 and v2 configuration

Following are the differences between v1.1.11 configuration and v2.0.0 server configuration:

### Deprecated fields in v2.0.0
* global
    * db_host
    * db_user
    * db_name
    * groups_service_account_key_file
    * max_results_admin_api

* inventory
    * pipelines


### New fields in v2.0.0
* inventory
    * api_quota
        * servicemanagement

* notifier
    * resources
        * notifiers
            * configuration
                * data_format
    * violation
    * inventory

### Updated/Renamed fields in v2.0.0

{: .table .table-striped}
| v1.1.11 | v2.0.0 |
|--------|--------|
| global/domain_super_admin_email | inventory/domain_super_admin_email |
| global/max_admin_api_calls_per_100_seconds | inventory/api_quota/admin |
| global/max_appengine_api_calls_per_second | inventory/api_quota/appengine |
| global/max_bigquery_api_calls_per_100_seconds | inventory/api_quota/bigquery |
| global/max_cloudbilling_api_calls_per_60_seconds | inventory/api_quota/cloudbilling |
| global/max_compute_api_calls_per_second | inventory/api_quota/compute |
| global/max_container_api_calls_per_100_seconds | inventory/api_quota/container |
| global/max_crm_api_calls_per_100_seconds | inventory/api_quota/crm |
| global/max_iam_api_calls_per_second | inventory/api_quota/iam |
| global/max_sqladmin_api_calls_per_100_seconds | inventory/api_quota/sqladmin |
| notifier/resources/pipelines/email_violations_pipeline | notifier/resources/notifiers/email_violations |
| notifier/resources/pipelines/gcs_violations_pipeline | notifier/resources/notifiers/gcs_violations |
| notifier/resources/pipelines/slack_webhook_pipeline | notifier/resources/notifiers/slack_webhook |

To learn more about these fields, see [Configure]({% link _docs/latest/configure/general/index.md %}).

{% endcapture %}
{% include site/zippy/item.html title="Upgrading 1.X installations" content=1x_upgrade uid=0 %}

{% capture 2x_upgrade %}

## Upgrade Forseti v2 using deployment manager

If you used the Forseti installer to deploy, the deployment template is in your 
Cloud Storage bucket for the Forseti instance under the `deployment_templates` folder.
The filename will be in the following format: `deploy-forseti-{forseti_instance_type}-{hash}.yaml`,
for example, `deploy-forseti-server-79c4374.yaml`.

### Change deployment properties

1. Review [`deploy-forseti-server.yaml.in`](https://github.com/GoogleCloudPlatform/forseti-security/blob/dev/deployment-templates/deploy-forseti-server.yaml.in) 
and [`deploy-forseti-client.yaml.in`](https://github.com/GoogleCloudPlatform/forseti-security/blob/dev/deployment-templates/deploy-forseti-client.yaml.in) 
for any new properties that you need to copy to your previous deployment template. To compare what's changed, use
the `git diff` command. For example, to see the diff between v2.1.0 and v2.2.0, run:

   ```bash
   $ git diff v2.1.0..v2.2.0 -- deployment-templates/deploy-forseti-server.yaml.in
   ```

1. Edit `deploy-forseti-{forseti_instance_type}-{hash}.yaml` and update the field `forseti-version:` under
section `Compute Engine` to the newest tag. For more information, see [the latest release]({% link releases/index.md %}).

### Run the Deployment Manager update

Run the following update command:

```bash
$ gcloud deployment-manager deployments update DEPLOYMENT_NAME \
  --config path/to/deploy-forseti-{forseti_instance_type}-{HASH}.yaml
```

If you changed the properties in the `deploy-forseti-{forseti_instance_type}-{hash}.yaml` `Compute Engine`
section or the startup script in `forseti-instance.py`, you need to reset the instance for changes 
to take effect:

  ```bash
  $ gcloud compute instances reset COMPUTE_ENGINE_INSTANCE_NAME
  ```

The Compute Engine instance will restart and perform a fresh installation of Forseti. You won't 
need to SSH to the instance to run all the git clone or Python install commands.

Some resources can't be updated in a deployment. If an error displays that you can't
change a certain resource, you'll need to create a new deployment of Forseti.

Learn more about [Updating a Deployment](https://cloud.google.com/deployment-manager/docs/deployments/updating-deployments).

{% capture upgrading_2_0_0_to_2_1_0 %}

1. Open cloud shell when you are in the Forseti project on GCP.
1. Checkout forseti with tag v2.1.0 by running the following commands:
    1. If you already have the forseti-security folder under your cloud shell directory, 
    run command `rm -rf forseti-security` to delete the folder.
    1. Run command `git clone` to clone the forseti-security directory to cloud shell.
    1. Run command `cd forseti-security` to navigate to the forseti-security directory.
    1. Run command `git checkout tags/v2.1.0` to checkout version `v2.1.0` of Forseti Security.
1. Download the latest copy of your Forseti server deployment template file from the Forseti server GCS 
bucket (located under `forseti-server-xxxxxx/deployment_templates`).
1. Update the `forseti-version` inside the deployment template to `v2.1.0`.
1. Create file `forseti_server_v2_1_0.yaml` under `forseti-security/deployment-templates`.
1. Copy and paste the content of the deployment template to file `forseti_server_v2_1_0.yaml`.
1. Upload file `forseti_server_v2_1_0.yaml` back to the GCS bucket (`forseti-server-xxxxxx/deployment_templates`)
1. Navigate to [Deployment Manager](https://console.cloud.google.com/dm/deployments) and 
copy the deployment name for Forseti server.
1. Run command `gcloud deployment-manager deployments update DEPLOYMENT_NAME --config forseti_server_v2_1_0.yaml`
If you see errors while running the deployment manager update command, please refer to below section 
`Error while running deployment manager` for details on how to workaround the error.
1. Repeat step `3-9` for Forseti client.

{% endcapture %}
{% include site/zippy/item.html title="Upgrading 2.0.0 to 2.1.0" content=upgrading_2_0_0_to_2_1_0 uid=2 %}

{% capture upgrading_2_1_0_to_2_2_0 %}

1. Open cloud shell when you are in the Forseti project on GCP.
1. Checkout forseti with tag v2.2.0 by running the following commands:
    1. If you already have the forseti-security folder under your cloud shell directory, 
    run command `rm -rf forseti-security` to delete the folder.
    1. Run command `git clone` to clone the forseti-security directory to cloud shell.
    1. Run command `cd forseti-security` to navigate to the forseti-security directory.
    1. Run command `git checkout tags/v2.2.0` to checkout version `v2.2.0` of Forseti Security.
1. Download the latest copy of your Forseti server deployment template file from the Forseti server GCS 
bucket (located under `forseti-server-xxxxxx/deployment_templates`).
1. Add the following fields to the compute engine section inside your deployment template.  
    `region` - The region of your VM, e.g. us-central1,  
    `vpc-host-project-id` - VPC host project ID, by default if you are not using VPC, 
    you can default it to your Forseti project ID,  
    `vpc-host-network` - VPC host network, by default if you are not using VPC, you can default it to `default`,  
    `vpc-host-subnetwork`- VPC host subnetwork, by default if you are not using VPC, you can default it to `default`  
    ```
    
    # Compute Engine
    - name: forseti-instance-server
      type: forseti-instance-server.py
      properties:
        # GCE instance properties
        image-project: ubuntu-os-cloud
        image-family: ubuntu-1804-lts
        instance-type: n1-standard-2
        
        <pre>
        <b>
        # V2.2.0 newly added fields
        region: **{YOUR FORSETI VM REGION, e.g. us-central1}**
        vpc-host-project-id: {YOUR_FORSETI_PROJECT_ID}
        vpc-host-network: default
        vpc-host-subnetwork: default
        </b>
        </pre>
        ...
        run-frequency: ...
        
    ```
1. Update the `forseti-version` inside the deployment template to `v2.2.0`.
1. Create file `forseti_server_v2_2_0.yaml` under `forseti-security/deployment-templates`.
1. Copy and paste the content of the deployment template to file `forseti_server_v2_2_0.yaml`.
1. Upload file `forseti_server_v2_2_0.yaml` back to the GCS bucket (`forseti-server-xxxxxx/deployment_templates`)
1. Navigate to [Deployment Manager](https://console.cloud.google.com/dm/deployments) and 
copy the deployment name for Forseti server.
1. Run command `gcloud deployment-manager deployments update DEPLOYMENT_NAME --config forseti_server_v2_2_0.yaml`  
If you see errors while running the deployment manager update command, please refer to below section 
`Error while running deployment manager` for details on how to workaround the error.
1. Repeat step `3-10` for Forseti client.

{% endcapture %}
{% include site/zippy/item.html title="Upgrading 2.1.0 to 2.2.0" content=upgrading_2_1_0_to_2_2_0 uid=3 %}

{% capture upgrading_2_2_0_to_2_3_0 %}

1. Open cloud shell when you are in the Forseti project on GCP.
1. Checkout forseti with tag v2.3.0 by running the following commands:
    1. If you already have the forseti-security folder under your cloud shell directory, 
    run command `rm -rf forseti-security` to delete the folder.
    1. Run command `git clone` to clone the forseti-security directory to cloud shell.
    1. Run command `cd forseti-security` to navigate to the forseti-security directory.
    1. Run command `git checkout tags/v2.3.0` to checkout version `v2.3.0` of Forseti Security.
1. Download the latest copy of your Forseti server deployment template file from the Forseti server GCS 
bucket (located under `forseti-server-xxxxxx/deployment_templates`).
1. Update the `forseti-version` inside the deployment template to `v2.3.0`.
1. Create file `forseti_server_v2_3_0.yaml` under `forseti-security/deployment-templates`.
1. Copy and paste the content of the deployment template to file `forseti_server_v2_3_0.yaml`.
1. Upload file `forseti_server_v2_3_0.yaml` back to the GCS bucket (`forseti-server-xxxxxx/deployment_templates`)
1. Navigate to [Deployment Manager](https://console.cloud.google.com/dm/deployments) and 
copy the deployment name for Forseti server.
1. Run command `gcloud deployment-manager deployments update DEPLOYMENT_NAME --config forseti_server_v2_3_0.yaml`
If you see errors while running the deployment manager update command, please refer to below section 
`Error while running deployment manager` for details on how to workaround the error.
1. Repeat step `3-9` for Forseti client.

{% endcapture %}
{% include site/zippy/item.html title="Upgrading 2.2.0 to 2.3.0" content=upgrading_2_2_0_to_2_3_0 uid=4 %}

{% capture upgrading_2_3_0_to_2_4_0 %}

1. Open cloud shell when you are in the Forseti project on GCP.
1. Checkout forseti with tag v2.4.0 by running the following commands:
    1. If you already have the forseti-security folder under your cloud shell directory, 
    run command `rm -rf forseti-security` to delete the folder.
    1. Run command `git clone` to clone the forseti-security directory to cloud shell.
    1. Run command `cd forseti-security` to navigate to the forseti-security directory.
    1. Run command `git checkout tags/v2.4.0` to checkout version `v2.4.0` of Forseti Security.
1. Download the latest copy of your Forseti server deployment template file from the Forseti server GCS 
bucket (located under `forseti-server-xxxxxx/deployment_templates`).
1. Update the `forseti-version` inside the deployment template to `v2.4.0`.
1. Create file `forseti_server_v2_4_0.yaml` under `forseti-security/deployment-templates`.
1. Copy and paste the content of the deployment template to file `forseti_server_v2_4_0.yaml`.
1. Upload file `forseti_server_v2_4_0.yaml` back to the GCS bucket (`forseti-server-xxxxxx/deployment_templates`)
1. Navigate to [Deployment Manager](https://console.cloud.google.com/dm/deployments) and 
copy the deployment name for Forseti server.
1. Run command `gcloud deployment-manager deployments update DEPLOYMENT_NAME --config forseti_server_v2_4_0.yaml`
If you see errors while running the deployment manager update command, please refer to below section 
`Error while running deployment manager` for details on how to workaround the error.
1. Repeat step `3-9` for Forseti client.

{% endcapture %}
{% include site/zippy/item.html title="Upgrading 2.3.0 to 2.4.0" content=upgrading_2_3_0_to_2_4_0 uid=5 %}

{% capture deployment_manager_error %}

If you get the following error while running the deployment manager:
```
The fingerprint of the deployment is .....
Waiting for update [operation-xxx-xxx-xxx-xxx]...failed.
ERROR: (gcloud.deployment-manager.deployments.update) Error in Operation [operation-xxx-xxx-xxx-xxx]: errors:
- code: NO_METHOD_TO_UPDATE_FIELD
  message: No method found to update field 'networkInterfaces' on resource 'forseti-server-vm-xxxxx'
    of type 'compute.v1.instance'. The resource may need to be recreated with the
    new field.
```

You can follow the following steps to workaround this deployment manager problem:
1. Copy the compute engine section inside your deployment template and paste it to a text editor.
```
# Compute Engine
- name: forseti-instance-server
  type: forseti-instance-server.py
  properties:
    # GCE instance properties
    image-project: ubuntu-os-cloud
    image-family: ubuntu-1804-lts
    instance-type: n1-standard-2
    ...
    run-frequency: ...
```
1. Remove the compute engine section from the deployment template.
1. Run the deployment manager uppdate command on the updated deployment template.  
`gcloud deployment-manager deployments update DEPLOYMENT_NAME --config forseti_server_v2_x_x.yaml`  
This will remove your VM instance.
1. Paste the compute engine section back to the deployment template, and run the update command again.  
`gcloud deployment-manager deployments update DEPLOYMENT_NAME --config forseti_server_v2_x_x.yaml`  
This will recreate the VM with updated fields.

{% endcapture %}
{% include site/zippy/item.html title="Error while running deployment manager" content=deployment_manager_error uid=50 %}

{% endcapture %}
{% include site/zippy/item.html title="Upgrading 2.X installations" content=2x_upgrade uid=1 %}

## What's next

* Customize [Inventory]({% link _docs/latest/configure/inventory/index.md %}) and
[Scanner]({% link _docs/latest/configure/scanner/index.md %}).
* Configure Forseti to send
[email notifications]({% link _docs/latest/configure/notifier/index.md %}#email-notifications-with-sendgrid).
* Enable [G Suite data collection]({% link _docs/latest/configure/inventory/gsuite.md %})
for processing by Forseti.