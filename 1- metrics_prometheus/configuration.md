
🛠️ Installation & Configurations

Prerequisites:

Download and Install AWS Cli
Setup and configure AWS CLI using the aws configure command.
Install and configure eksctl.
Install and configure kubectl.

----------------------------------
1. create eks cluster:

eksctl create cluster --name=observability \
                      --region=us-east-1 \
                      --zones=us-east-1a,us-east-1b \
                      --without-nodegroup

-------------------------------------
2. connect EKS cluster with an IAM OIDC (OpenID Connect) identity provider.

eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster observability \
    --approve


> so pods get acccess to aws resources:
-   CloudWatch
-   AWS Load Balancer Controller
-   S3
-   Secrets Manager

> EKS OIDC Provider: Associates an IAM OIDC identity provider with an EKS cluster. It enables Kubernetes service accounts to obtain AWS IAM permissions through mechanisms such as IRSA, allowing pods to access AWS services without relying on the worker node's IAM role.

Pod
 ↓
Kubernetes ServiceAccount  : This is called IAM Roles for Service Accounts (IRSA).
 ↓
OIDC Provider
 ↓
IAM Role
 ↓
CloudWatch

-------------------------------------

3. create node group

eksctl create nodegroup --cluster=observability \
                        --region=us-east-1 \
                        --name=observability-ng-private \
                        --node-type=t3.medium \
                        --nodes-min=2 \
                        --nodes-max=3 \
                        --node-volume-size=20 \
                        --managed \
                        --asg-access \
                        --external-dns-access \
                        --full-ecr-access \
                        --appmesh-access \
                        --alb-ingress-access \
                        --node-private-networking

# Update ./kube/config file
aws eks update-kubeconfig --name observability

---------------------------------------------------------

4. install prometheus

## 
cmd: helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

cmd: helm repo update

## Deploy the chart into a new namespace "monitoring"
cmd: kubectl create ns monitoring
cmd: helm install monitoring prometheus-community/kube-prometheus-stack \
-n monitoring \
-f ./custom_kube_prometheus_stack.yml

Prometheus UI:
> kubectl port-forward service/prometheus-operated -n monitoring 9090:9090

-------------------------------------------------------------
Grafana UI:

>kubectl port-forward service/monitoring-grafana -n monitoring 8080:80

Default Credentials:
Username: admin
Password: prom-operator (explicitly configured in custom_kube_prometheus_stack.yml)
Retrieving Auto-Generated/Actual Credentials: If you did not use the custom configuration file, or if the credentials don't work, retrieve them directly from the Kubernetes secret:

Get username:

> kubectl get secret --namespace monitoring monitoring-grafana -o jsonpath='{.data.admin-user}' | base64 -d
Get password:

> kubectl get secret --namespace monitoring monitoring-grafana -o jsonpath='{.data.admin-password}' | base64 -d
Port-forwarding to access Grafana UI:


Alertmanager UI:

> kubectl port-forward service/alertmanager-operated -n monitoring 9093:9093
---------------------------------------------------

In a real EKS environment
A common production architecture is:

                   Internet
                      │
                      ↓
              AWS Application
              Load Balancer (ALB)
                      │
                      ↓
             Kubernetes Ingress
                      │
             ┌────────┴────────┐
             ↓                 ↓
          Grafana           Other Apps
             │
             ↓
           Pods
-   In EKS, the AWS Load Balancer Controller can watch Kubernetes Ingress resources and provision/configure an AWS ALB.
-   In the AWS Load Balancer Controller setup, the Kubernetes Ingress resource is configuration (yml), and the controller uses that configuration to create/configure the AWS ALB.

> Run the ingress_prom_grafana.yml to setup ALB
