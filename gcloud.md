#### environment variables

```shell
export GCP_PROJECT_ID='GCP_PROJECT_ID'
export SERVICE_ACCOUNT='service-account@example.iam.gserviceaccount.com'
export CLUSTER_NAME='CLUSTER_NAME'
export ZONE='europe-north1-c'        # gcloud compute zones list
export REGION='europe-north1'        # gcloud compute regions list
export MACHINE_TYPE='n1-standard-8'  # gcloud compute machine-types list --zones=$ZONE
```

#### setup gcloud

```bash
gcloud components update
```

```bash
gcloud auth login --no-launch-browser
```

```bash
gcloud config set project $GCP_PROJECT_ID
```

```bash
gcloud config set account $SERVICE_ACCOUNT
```

#### create cluster

```bash
gcloud container clusters create $CLUSTER_NAME \
  --zone $ZONE \
  --machine-type $MACHINE_TYPE \
  --num-nodes 1
```

#### get credentials

```bash
gcloud container clusters get-credentials $CLUSTER_NAME --zone $ZONE --project $GCP_PROJECT_ID
```

#### create address for LoadBalancer

```bash
gcloud compute addresses create $GCP_PROJECT_ID --region $REGION
```

#### ssh into pod

```bash
gcloud compute ssh --zone $ZONE <NODE> --project $GCP_PROJECT_ID --container=<POD>
```

```bash
ssh -J <GCLOUD_USER>@<NODE_EXTERNAL_IP> <USER>@<POD_IP>
```

```bash
ssh -o ProxyCommand='ssh -W %h:%p <GCLOUD_USER>@<NODE_EXTERNAL_IP>' <USER>@<POD_IP>
```
