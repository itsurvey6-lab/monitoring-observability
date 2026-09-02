> Logging overview
Logging is crucial in any distributed system, especially in Kubernetes, to monitor application behavior, detect issues, and ensure the smooth functioning of microservices.

> Importance:
Debugging: Logs provide critical information when debugging issues in applications.
Auditing: Logs serve as an audit trail, showing what actions were taken and by whom.
Performance Monitoring: Analyzing logs can help identify performance bottlenecks.
Security: Logs help in detecting unauthorized access or malicious activities.

> Tools Available for Logging in Kubernetes
🗂️ EFK Stack (Elasticsearch, Fluentbit, Kibana)
🗂️ EFK Stack (Elasticsearch, FluentD, Kibana)
🗂️ ELK Stack (Elasticsearch, Logstash, Kibana)
📊 Promtail + Loki + Grafana

> EFK Stack (Elasticsearch, Fluentbit, Kibana)
EFK is a popular logging stack used to collect, store, and analyze logs in Kubernetes.
Elasticsearch: Stores and indexes log data for easy retrieval.
Fluentbit: A lightweight log forwarder that collects logs from different sources and sends them to Elasticsearch.
Kibana: A visualization tool that allows users to explore and analyze logs stored in Elasticsearch.
-----------------------------------------------------------------------------------

> EKS Storage + EFK — Quick Notes

- EBS CSI Driver and its IAM setup are normally installed/configured at the EKS level
                    AWS / EKS
                        │
                        │
              ┌─────────▼─────────┐
              │   EBS CSI Driver  │
              │                   │
              │ ServiceAccount    │
              │        │          │
              │        ▼          │
              │    IAM Role       │
              └─────────┬─────────┘
                        │
                        │ AWS API
                        ▼
                   Amazon EBS
                        │
                        │
                ┌───────▼───────┐
                │       PV      │
                │ Persistent    │
                │   Volume      │
                └───────┬───────┘
                        │
                        │ bound to
                        ▼
                ┌───────────────┐
                │      PVC      │
                │ Persistent    │
                │ Volume Claim  │
                └───────┬───────┘
                        │
                        │ mounted by
                        ▼
                ┌───────────────┐
                │ Elasticsearch │
                │      Pod      │
                └───────────────┘
# Component	                                    # Simple meaning
-   Pod	                            -  Application/workload running in Kubernetes
-   Service	                        -  Provides stable network access to Pods
-   ServiceAccount	                -  Kubernetes identity for a Pod
-   IAM Role	                    -  AWS permissions/identity
-   EBS CSI Driver	                -  Connects Kubernetes storage operations with AWS EBS
-   StorageClass	                -  Defines how storage should be dynamically provisioned
-   PVC	                            -  Pod/workload asks for storage
-   PV	                            -  Kubernetes representation of persistent storage
-   EBS	                            -  Actual AWS block storage


> EFK storage flow

Application Pods
       │
       │ logs
       ▼
  Fluent Bit
       │
       │ sends logs
       ▼
 Elasticsearch
       │
       │ persistent data
       ▼
      PVC
       │
       ▼
      PV
       │
       ▼
   EBS Volume
