#### gcloud

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
