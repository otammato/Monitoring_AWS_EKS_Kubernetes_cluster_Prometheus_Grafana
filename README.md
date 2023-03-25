# Monitoring_AWS_EKS_with_Prometheus_Grafana


<img width="700" alt="Screenshot 2023-03-25 at 22 14 36" src="https://user-images.githubusercontent.com/104728608/227744857-0be2db6f-e4df-411d-8ba8-26b1dfa1dfcf.png">

---

## Result

<img width="700" alt="Screenshot 2023-03-25 at 11 48 47" src="https://user-images.githubusercontent.com/104728608/227743686-d79aab7f-18da-4ca9-b38c-3c35d77b5c1f.png">

<img width="700" alt="Screenshot 2023-03-25 at 11 43 14" src="https://user-images.githubusercontent.com/104728608/227743988-165a412b-f084-478b-9326-152526a791c0.png">

<img width="700" alt="Screenshot 2023-03-25 at 11 38 57" src="https://user-images.githubusercontent.com/104728608/227743819-38ae0727-1770-4388-849b-0b3363d3d8a0.png">


<img width="700" alt="Screenshot 2023-03-25 at 11 56 00" src="https://user-images.githubusercontent.com/104728608/227744095-9e3ae8dc-ac09-4783-a4fb-ac5d0d3c6daf.png">

<img width="700" alt="Screenshot 2023-03-25 at 12 05 37" src="https://user-images.githubusercontent.com/104728608/227744097-ec3e2bfe-c8ea-4456-945e-c449bbd85c4a.png">


## Setup an AWS EC2 Instance



## Install AWS CLI and Configure

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" 
sudo apt install unzip
unzip awscliv2.zip 
sudo ./aws/install
```
```
aws configure
```
## Install and Setup Kubectl

```
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version
```

## Install and Setup eksctl

```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin

eksctl version
```
## Install Helm chart

```
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```
## Creating an Amazon EKS cluster using eksctl

```
eksctl create cluster --name eks2 --version 1.24 --region us-east-1 --nodegroup-name worker-nodes --node-type t2.large --nodes 2 --nodes-min 2 --nodes-max 3

aws eks update-kubeconfig --name eks2 # verify cluster is instaled

```
## Installing the Kubernetes Metrics Server

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

kubectl get deployment metrics-server -n kube-system # verify

```
## Install Prometheus

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo update

helm repo list

kubectl create namespace prometheus

```
```
helm install prometheus prometheus-community/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2"
```

## Create IAM OIDC Provider

```
oidc_id=$(aws eks describe-cluster --name eks2 --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4

eksctl utils associate-iam-oidc-provider --cluster eks2 --approve
```
```
eksctl create iamserviceaccount \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster eks2 \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve \
  --role-only \
  --role-name AmazonEKS_EBS_CSI_DriverRole
```
```
eksctl create addon --name aws-ebs-csi-driver --cluster eks2 --service-account-role-arn arn:aws:iam::xxxxxxxxx:role/AmazonEKS_EBS_CSI_DriverRole --force

```
```
curl localhost:9090/graph # check the prometheus was launched

```
## Install Grafana

```
helm repo add grafana https://grafana.github.io/helm-charts 
helm repo update
```
```
vi prometheus-datasource.yaml
```
```
datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      url: http://prometheus-server.prometheus.svc.cluster.local
      access: proxy
      isDefault: true
```
```
kubectl create namespace grafana
```
```
helm install grafana grafana/grafana \
    --namespace grafana \
    --set persistence.storageClassName="gp2" \
    --set persistence.enabled=true \
    --set adminPassword='EKS!sAWSome' \
    --values prometheus-datasource.yaml \
    --set service.type=LoadBalancer
```

Copy External IP address and open it in the browser -

Password is EKS!sAWSome as set up while creating Grafana

<img width="810" alt="Screenshot 2023-03-25 at 22 39 33" src="https://user-images.githubusercontent.com/104728608/227745655-bf1325b0-ce5f-4508-9f93-9fc936d52225.png">

## Import Grafana dashboard from Grafana Labs

```
```
## Deploy a Node.js application and monitor it on Grafana
## Clean Up
