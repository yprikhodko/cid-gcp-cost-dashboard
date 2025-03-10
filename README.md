# Visualisation of GCP Costs in QuickSight Dashboard
Current status: `ETA Program`

This guide explores the newly announced managed connector for Google BigQuery and demonstrates how to build a modern ETL pipeline for GCP Cost with AWS Glue Studio without writing code.

![gcp-architecture.png](/images/gcp-architecture.png)


## 	Prerequisites


##### 1. Create a GCP Billing Export:

- One export is related to one billing account id in GCP.

- To setup an export, please follow this documentation: https://cloud.google.com/billing/docs/how-to/export-data-bigquery-setup

##### 2. Create a GCP Pricing Export

- When enabling Cloud Billing export, ensure to include Pricing data.

- https://cloud.google.com/billing/docs/how-to/export-data-bigquery-setup#enable-bq-export

##### 3. Configure BigQuery Connection Secret:

- To connect to Google BigQuery from AWS Glue, see and complete points 1-4 in Configuring BigQuery connections. You must create and store your Google Cloud Platform credentials in a Secrets Manager secret, then associate that secret with a Google BigQuery AWS Glue connection.

- When selecting GCP IAM roles for your service account, add or create a role that would grant appropriate permissions to run BigQuery jobs to *both read and write*, and create BigQuery tables in the single billing dataset.

- Video to create the secret: 1:54-2:38 https://youtu.be/n6fkX5LpEYY?t=114


  

At the end of prerequisites, you should have the following values:

|         Value       | Sample                        |
|----------------|-------------------------------|
|1 or more BigQuery billing table|`project.dataset.gcp_billing_export_v1_BILLING_ACCOUNT_ID`|
|1 or more BigQuery pricing table|`project.dataset.cloud_pricing_export`|
|1 Secret in AWS with json key of GCP SA stored in base64 with credentials as key|`bigquery_credentials`|



  

## Infrastructure Deployment Guide

##### 1. Open Cloudformation in your AWS console, in your CID account, or [Click Here](https://eu-central-1.console.aws.amazon.com/cloudformation/home?region=eu-central-1#/stacks/create/review?templateURL=https://github.com/https://github.com/awslabs/cid-gcp-cost-dashboard/GCP-Cost-Dashboard-Stack.yml) for complete Step1&2.

https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filteringText=&filteringStatus=active&viewNested=true

##### 2. Select ‘Stack’, then ‘Create Stack’ and finally ‘Upload a template file’.
- Upload `GCP-Cost-Dashboard-Stack.yaml`, click **Next**

##### 3. Insert parameters:

- `Prefix` => a prefix for the resources created

- `BucketName` => a bucket to store exctracted data. Provide a name based on your naming convention.

- `ScriptBucketName` => a bucket to store glue job code. Provide a name based on your naming convention.

- `GCPFullTableName` => this is a COMMADELIMITEDLIST type, and should includes your billing export tables ID delimited by a “,”. 

    example: value1,value2. to retrieve your table ID, just select the three dots on the right of a table and click “Copy ID”.

- `GCPPricingFullTableName` => your pricing table id. to retrieve your table ID, just select the three dots on the right of a table and click “Copy ID”.

- `GCPConnectionName` => a name for the connection. Default can be good or adapt based on your naming convention needs.

- `KMSOwners` => List of ARNs of Owners of the KMS key that will be created.

- `GlueCrontab` => Time-based schedule for your jobs in AWS Glue. The definition of these schedules uses the Unix-like cronsyntax. For more info check: https://docs.aws.amazon.com/glue/latest/dg/monitor-data-warehouse-schedule.html

- `StartDate` => Start date for data retrieval, leave empty to retrieve all. Format: YYYY-MM-DD HH:MM:SS

- `GCPBillingLocation`, GCPPricingLocation, GCPJobBookmarksKeys and TargetCatalogDBName are reserved and must not be changed. 

##### 5. Click on Next until completing the form.

##### 6. Launch the cloudformation and wait until it’s complete.

##### 7. Test the import:

- Glue Jobs will automatically start once they're created. Check your Glue Jobs results and wait until all Crawlers are completed before proceeding with dashboard installation, otherwise will fail due to missing tables in Athena.

## Dashboards Deployment

1. On CloudShell install cid with command: 
    ```bash
    pip3 install -U cid-cmd
    ```
2. Launch deploy command with parameters:
    ```bash
    cid-cmd deploy \
          --resources https://github.com/awslabs/cid-gcp-cost-dashboard/raw/mainline/GCP-Cost-Dashboard.yaml \
          --dasbhoard-id gcp-cost-dashboard
    ```
3. You can navigate to dashboards in QuickSight once deplyment is finished. You might need to adjust permissions on the role that QuickSight uses to access S3 and Athena.
