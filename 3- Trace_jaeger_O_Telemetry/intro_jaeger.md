# 🕵️‍♂️ What is Jaeger?
Jaeger is an open-source, end-to-end distributed tracing system used for monitoring and troubleshooting microservices-based architectures. It helps developers understand how requests flow through a complex system, by tracing the path a request takes and measuring how long each step in that path takes.

🐢 Identifying bottlenecks: See where your application spends most of its time.

🔍 Finding root causes of errors: Trace errors back to their source.

⚡ Optimizing performance: Understand and improve the latency of services.

Service A / Service B
        │
        │ OpenTelemetry tracing
        ▼
      Jaeger
        │
        ▼
 Elasticsearch
        │
        ▼
    Jaeger UI



# Components of Jaeger

Jaeger consists of several components:
Agent: Collects traces from your application.
Collector: Receives traces from the agent and processes them.
Query: Provides a UI to view traces.
Storage: Stores traces for later retrieval (often a database like Elasticsearch).

---------------------------------------------------------------

# ⚙️ Setting Up Jaeger

> Step 1: Instrumenting Your Code  # very importnat #
To start tracing, you need to instrument your services. This means adding tracing capabilities to your code.

#                instrumented code is availble in application/service-a/

> step 2: configure or install the elastic search to store the traces and get the username and password in namespace logging
- follow the the steps availble in 2-Logging_EFF

- Then export the CA certificate:
cmd: kubectl get secret elasticsearch-master-certs -n logging -o jsonpath='{.data.ca\.crt}' | base64 --decode > ca-cert.pem

> step 3: Create Tracing namespace tracing
-   Creates a ConfigMap in the tracing namespace, containing the CA certificate to be used by Jaeger for TLS.
cmd:    kubectl create configmap jaeger-tls --from-file=ca-cert.pem -n tracing

> step 4:  Create Secret for Elasticsearch TLS
-   Creates a Kubernetes Secret in the tracing namespace, containing the CA certificate for Elasticsearch TLS communication.
cmd: kubectl create secret generic es-tls-secret --from-file=ca-cert.pem -n tracing

> Step 5: Add Jaeger Helm Repository
adds the official Jaeger Helm chart repository to your Helm setup, making it available for installations.
cmd:   helm repo add jaegertracing https://jaegertracing.github.io/helm-charts

cmd:    helm repo update

>   Step 6: Install Jaeger with Custom Values
👉 Note: Update the Elasticsearch password in jaeger-values.yaml
using the current credential from the logging namespace.


-   Command installs Jaeger into the tracing namespace using a custom jaeger-values.yaml configuration file. Ensure the password is updated in the file before installation.
cmd: helm install jaeger jaegertracing/jaeger -n tracing --values jaeger-values.yaml

> step 7:   Port Forward Jaeger Query Service
-   Command forwards port 8080 on your local machine to the Jaeger Query service, allowing you to access the Jaeger UI locally.
cmd: kubectl port-forward svc/jaeger-query 8080:80 -n tracing
-------------------------------------------------------------------------------

🧼 Clean Up

helm uninstall jaeger -n tracing

helm uninstall elasticsearch -n logging

# Also delete PVC created for elasticsearch

helm uninstall monitoring -n monitoring

cd day-4

kubectl delete -k kubernetes-manifest/

kubectl delete -k alerts-alertmanager-servicemonitor-manifest/

# Delete cluster
eksctl delete cluster --name observability