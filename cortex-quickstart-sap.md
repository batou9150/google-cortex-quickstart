# Quick start with [Google Cloud Cortex Framework](https://github.com/GoogleCloudPlatform/cortex-data-foundation) for SAP


Running the following commands in [Cloud Shell](https://cloud.google.com/shell) will:
- create 2 buckets
- create 4 BigQuery Datasets
- check permissions (you should have "BigQuery Admin" & "Storage Object Admin")
- generate data for SAP

## Variables

```sh
export PJID=${GOOGLE_CLOUD_PROJECT} # by default, we use the default project environment variable
export BUCKET_LOGS=${PJID}-cortex-sap-logs
export BUCKET_DAGS=${PJID}-cortex-sap-dags
export DS_RAW=CORTEX_SAP_RAW_LANDING
export DS_CDC=CORTEX_SAP_CDC_PROCESSED
export DS_REPORTING=CORTEX_SAP_REPORTING
export DS_MODELS=CORTEX_SAP_ML
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
bq --location=US mk --dataset ${PJID}:${DS_MODELS}
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
_PJID_SRC=${PJID},_PJID_TGT=${PJID},_DS_RAW=${DS_RAW},_DS_CDC=${DS_CDC},_DS_REPORTING=${DS_REPORTING},_DS_MODELS=${DS_MODELS},_GCS_BUCKET=${BUCKET_LOGS},_TGT_BUCKET=${BUCKET_DAGS},_TEST_DATA=true,_DEPLOY_CDC=true,_GEN_EXT=true,_DEPLOY_SAP=true,_DEPLOY_SFDC=false
```

## Expected results

| dataset | objects |
| --- | --- |
| CORTEX_SAP_RAW_LANDING | 115 tables (e.g. `adr6`, `adrc`, `but000`, ...) | 
| CORTEX_SAP_CDC_PROCESSED | Exactly the same tables as in CORTEX_SAP_RAW_LANDING | 
| CORTEX_SAP_REPORTING | 82 tables (e.g. `AccountingDocuments`, `AddressesMD`, `Billing`, ...) | 
| CORTEX_SAP_ML | 1 model : `clustering_nmodel` | 

## Looker

Import [Looker Dashboards for SAP](https://marketplace.looker.com/marketplace/detail/cortex-sap-operational) from [block-cortex-sap](https://github.com/looker-open-source/block-cortex-sap).

_Instructions for deploying the pre-built Looker blocks can be found [here](https://cloud.google.com/looker/docs/marketplace#installing_a_tool_from_a_git_url)._
