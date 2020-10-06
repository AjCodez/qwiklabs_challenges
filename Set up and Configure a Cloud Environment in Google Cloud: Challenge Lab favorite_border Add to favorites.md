# Set up and Configure a Cloud Environment in Google Cloud: Challenge Lab

Challenge lab walkthrough:

## Task 1: Create development VPC manually

```yaml
gcloud compute networks create griffin-dev-vpc --subnet-mode custom

gcloud compute networks subnets create griffin-dev-wp --network=griffin-dev-vpc --region us-east1 --range=192.168.16.0/20

gcloud compute networks subnets create griffin-dev-mgmt --network=griffin-dev-vpc --region us-east1 --range=192.168.32.0/20
```

## Task 2: Create production VPC using Deployment Manager

```yaml
gsutil cp -r gs://cloud-training/gsp321/dm .

cd dm

sed -i s/SET_REGION/us-east1/g prod-network.yaml

gcloud deployment-manager deployments create prod-network \
    --config=prod-network.yaml
```

## Task 3: Create bastion host

```yaml
cd ..

gcloud compute instances create bastion --network-interface=network=griffin-dev-vpc,subnet=griffin-dev-mgmt  --network-interface=network=griffin-prod-vpc,subnet=griffin-prod-mgmt --tags=ssh --zone=us-east1-b

gcloud compute firewall-rules create fw-ssh-dev --source-ranges=0.0.0.0/0 --target-tags ssh --allow=tcp:22 --network=griffin-dev-vpc

gcloud compute firewall-rules create fw-ssh-prod --source-ranges=0.0.0.0/0 --target-tags ssh --allow=tcp:22 --network=griffin-prod-vpc
```

## Task 4: Create and configure Cloud SQL Instance

```yaml
gcloud sql instances create griffin-dev-db --root-password password --region=us-east1

gcloud sql connect griffin-dev-db

# Copy paste the following from the lab mannual
CREATE DATABASE wordpress;
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
FLUSH PRIVILEGES;

# Use the following to get out of the SQL terminal
exit;
```

## Task 5: Create Kubernetes cluster

```yaml
gcloud container clusters create griffin-dev \
  --network griffin-dev-vpc \
  --subnetwork griffin-dev-wp \
  --machine-type n1-standard-4 \
  --num-nodes 2  \
  --zone us-east1-b
  
gcloud container clusters get-credentials griffin-dev --zone us-east1-b
```

## Task 6: Prepare the Kubernetes cluster

```yaml
gsutil cp -r gs://cloud-training/gsp321/wp-k8s .

cd wp-k8s

sed -i s/username_goes_here/wp_user/g wp-env.yaml

sed -i s/password_goes_here/stormwind_rules/g wp-env.yaml

kubectl create -f wp-env.yaml

gcloud iam service-accounts keys create key.json \
    --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com

kubectl create secret generic cloudsql-instance-credentials \
    --from-file key.json
```

## Task 7: Create a WordPress deployment

```yaml
# Use the following for replace YOUR_SQL_INSTANCE with "griffin-dev-db"
I=$(gcloud sql instances describe griffin-dev-db --format="value(connectionName)")

sed -i s/YOUR_SQL_INSTANCE/$I/g wp-deployment.yaml

kubectl create -f wp-deployment.yaml

kubectl create -f wp-service.yaml
```

## Task 8: Enable monitoring

1. Go to **OPERATIONS** > **Monitoring**
2. Wait for the workspace creation to complete
3. Go to **Uptime checks** > **CREATE UPTIME CHECKS**
4. Now enter the info as below:

<p align="center">
  <img width=500 src="https://github.com/DSC-IIIT-Kalyani/qwiklabs_challenges/raw/master/screenshots/uptime.png" alt="Uptime check" />
</p>

## Task 9: Provide access for an additional engineer

1. Go to **IAM & Admin** > **ADD**
2. Enter the second username and give him **Project** > **Editor** access in **Role**

<p align="center">
  <img width=500 src="https://github.com/DSC-IIIT-Kalyani/qwiklabs_challenges/raw/master/screenshots/IAM.png" alt="IAM" />
</p>














