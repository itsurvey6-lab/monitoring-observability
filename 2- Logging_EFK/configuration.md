# WHOLE SETUP

AWS EKS Cluster
       │
       ├── EBS CSI Driver
       │       │
       │       ├── ServiceAccount
       │       └── IAM Role
       │
       └── logging namespace
               │
               ├── Elasticsearch ← Helm
               │       │
               │       └── PVC → PV → EBS
               │
               ├── Kibana ← Helm
               │
               └── Fluent Bit ← Helm

# 📝 Step-by-Step Setup

> PHASE-1

1> Create IAM Role for Service Account
# Give the EBS CSI Driver permission to manage EBS volumes in AWS.

eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster observability \
    --role-name AmazonEKS_EBS_CSI_DriverRole \
    --role-only \
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
    --approve

2> Retrieve IAM Role ARN
# Command retrieves the ARN of the IAM role created for the EBS CSI controller service account.

ARN=$(aws iam get-role \
  --role-name AmazonEKS_EBS_CSI_DriverRole \
  --query 'Role.Arn' \
  --output text)

3> Install EBS CSI Driver

eksctl create addon \
    --cluster observability \
    --name aws-ebs-csi-driver \
    --version latest \
    --service-account-role-arn $ARN \
    --force
------------------------------------------------------------------

> PHASE-2

4> Create Namespace for Logging
cmd:  kubectl create namespace logging

5> Install Elasticsearch on K8s

cmd: helm repo add elastic https://helm.elastic.co

helm install elasticsearch \
 --set replicas=1 \
 --set volumeClaimTemplate.storageClassName=gp2 \
 --set persistence.labels.enabled=true elastic/elasticsearch -n logging

- Installs Elasticsearch in the logging namespace.
- It sets the number of replicas, specifies the storage class, and enables persistence labels to ensure data is stored on persistent volumes.
# Elasticsearch requests persistent storage
-   volumeClaimTemplate.storageClassName=gp2

?   gp2 is an AWS EBS-backed StorageClass commonly found in older EKS setups.

-   Today, gp3 is generally preferred for new workloads because it is more flexible/cost-effective for many use cases.

6> Retrieve Elasticsearch Username & Password

# for username
cmd: kubectl get secrets --namespace=logging elasticsearch-master-credentials -ojsonpath='{.data.username}' | base64 -d
# for password
cmd: kubectl get secrets --namespace=logging elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d

-   Retrieves the password for the Elasticsearch cluster's master credentials from the Kubernetes secret.
-   The password is base64 encoded, so it needs to be decoded before use.
👉 Note: Please write down the password for future reference

7> Install Kibana
cmd: helm install kibana --set service.type=LoadBalancer elastic/kibana -n logging

-   Kibana provides a user-friendly interface for exploring and visualizing data stored in Elasticsearch.
-   It is exposed as a LoadBalancer service, making it accessible from outside the cluster

8> Install Fluentbit with Custom Values/Configurations
👉 Note: Please update the HTTP_Passwd field in the fluentbit-values.yml file with the password retrieved earlier in step 6: (i.e NJyO47UqeYBsoaEU)"

cmd: helm repo add fluent https://fluent.github.io/helm-charts
cmd: helm install fluent-bit fluent/fluent-bit -f fluentbit-values.yaml -n logging

------------------------------------------------------------------

🧼 Clean Up

helm uninstall monitoring -n monitoring

helm uninstall fluent-bit -n logging

helm uninstall elasticsearch -n logging

helm uninstall kibana -n logging

cd day-4

kubectl delete -k kubernetes-manifest/

kubectl delete -k alerts-alertmanager-servicemonitor-manifest/


eksctl delete cluster --name observability