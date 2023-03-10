# Quick start with [Google Cloud Cortex Framework](https://github.com/GoogleCloudPlatform/cortex-data-foundation) for Salesforce


Running the following commands in [Cloud Shell](https://cloud.google.com/shell) will:
- create 2 buckets
- create 3 BigQuery Datasets
- check permissions (you should have "BigQuery Admin" & "Storage Object Admin")
- generate data for Salesforce

## Variables

```sh
export PJID=${GOOGLE_CLOUD_PROJECT} # by default, we use the default project environment variable
export BUCKET_LOGS=${PJID}-cortex-sfdc-logs
export BUCKET_DAGS=${PJID}-cortex-sfdc-dags
export DS_RAW=CORTEX_SFDC_RAW_LANDING
export DS_CDC=CORTEX_SFDC_CDC_PROCESSED
export DS_REPORTING=CORTEX_SFDC_REPORTING
```

## Cloning repos

```sh
git clone --recurse-submodules https://github.com/GoogleCloudPlatform/cortex-data-foundation
git clone  https://github.com/lsubatin/mando-checker
```

## Create ressources

```sh
gcloud storage buckets create gs://${BUCKET_LOGS}
gcloud storage buckets create gs://${BUCKET_DAGS}

bq --location=US mk --dataset ${PJID}:${DS_RAW}
bq --location=US mk --dataset ${PJID}:${DS_CDC}
bq --location=US mk --dataset ${PJID}:${DS_REPORTING}
```


## Check

```sh
cd ~/mando-checker

gcloud builds submit \
   --project ${PJID} \
   --substitutions _DEPLOY_PROJECT_ID=${PJID},_DEPLOY_BUCKET_NAME=${BUCKET_DAGS},_LOG_BUCKET_NAME=${BUCKET_LOGS} .
```

## Run

```sh
cd ~/cortex-data-foundation

gcloud builds submit --project ${PJID} \
--substitutions \
_PJID_SRC=${PJID},_PJID_TGT=${PJID},_DS_RAW=${DS_RAW},_DS_CDC=${DS_CDC},_DS_REPORTING=${DS_REPORTING},_DS_MODELS=none,_GCS_BUCKET=${BUCKET_LOGS},_TGT_BUCKET=${BUCKET_DAGS},_TEST_DATA=true,_DEPLOY_CDC=true,_GEN_EXT=true,_DEPLOY_SAP=false,_DEPLOY_SFDC=true
```

## Looker

Import Looker Dashboards for Salesforce from [block-cortex-salesforce](https://github.com/looker-open-source/block-cortex-salesforce).

_Instructions for deploying the pre-built Looker blocks can be found [here](https://cloud.google.com/looker/docs/marketplace#installing_a_tool_from_a_git_url)._

| Accounts with cases | Case Overview & Trends |
|---|---|
| <img src="cortex-looker-salesforce-accounts_with_cases.png" alt="Cortex Looker Salesforce Accounts with cases" width="500" /> | <img src="cortex-looker-salesforce-case_overview_trends.png" alt="Cortex Looker Salesforce Case Overview & Trends" width="500" /> |

| Case Management & Resolution | Leads Capture & Conversion |
|---|---|
| <img src="cortex-looker-salesforce-case_management_resolution.png" alt="Cortex Looker Salesforce Case Management & Resolution" width="500" /> | <img src="cortex-looker-salesforce-leads_capture_conversion.png" alt="Cortex Looker Salesforce Leads Capture & Conversion" width="500" /> |

| Opportunity Trends & Pipeline | Sales Activities & Engagement |
|---|---|
| <img src="cortex-looker-salesforce-opportunity_trends_pipeline.png" alt="Cortex Looker Salesforce Opportunity Trends & Pipeline" width="500" /> | <img src="cortex-looker-salesforce-sales_activities_engagement.png" alt="Cortex Looker Salesforce Sales Activities & Engagement" width="500" /> |
