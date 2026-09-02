What is Instrumentation?

Instrumentation means adding monitoring capabilities to an application so we can understand what is happening inside it.

It can collect:

Metrics
Logs
Traces

For example, a developer can add code to a Node.js application to count HTTP requests and measure request duration.

> check sample metric nodejs code in folder Application

Application
    ↓
Instrumentation
    ↓
Metrics / Logs / Traces
    ↓
Monitoring tools
    ↓
Prometheus / Grafana / ELK / Jaeger
---------------------------------------------------------------------------
Why Do We Need Instrumentation?

Instrumentation gives us visibility into the application.

It helps us:

Monitor application health
Measure performance
Detect errors
Troubleshoot problems
Understand request traffic
Find slow requests
Monitor business/application-specific behavior
Application Instrumentation

There are two common approaches.

1. Code-level instrumentation

Developers add monitoring code directly into the application.

For a Node.js application, the prom-client library can be used to create Prometheus metrics.

Example:

const httpRequestCounter = new promClient.Counter({
    name: 'http_requests_total',
    help: 'Total number of HTTP requests',
    labelNames: ['method', 'path', 'status_code'],
});


The application can then expose the metrics through:

/metrics


Example output:

http_requests_total{method="GET",path="/",status_code="200"} 10

2. Exporters

Sometimes we don't modify the application.

Instead, we use an exporter that collects metrics from an existing system and exposes them for Prometheus.

Examples:

Node Exporter       → Linux server metrics
MySQL Exporter      → MySQL metrics
PostgreSQL Exporter → PostgreSQL metrics

Prometheus Metric Types
---------------------------------------------------------------------------
Prometheus has four important metric types.

1. Counter

A Counter is a value that normally only increases.

Used for:

HTTP requests
Errors
Container restarts
Completed jobs

Example:

http_requests_total


Kubernetes example:

kube_pod_container_status_restarts_total


Typical query:

rate(http_requests_total[5m])


This helps calculate the request rate.

2. Gauge

A Gauge can increase or decrease.

Used for values that represent the current state.

Examples:

Memory usage
CPU usage
Active users
Queue size

Example:

container_memory_usage_bytes


Think:

10 → 20 → 15 → 30


A gauge can move in both directions.

3. Histogram

A Histogram measures observations and puts them into configurable buckets.

It is commonly used for:

Request duration
Response size
Latency

Example:

http_request_duration_seconds


A histogram produces metrics such as:

http_request_duration_seconds_bucket
http_request_duration_seconds_sum
http_request_duration_seconds_count


It is useful for calculating latency percentiles such as P95 and P99.

Example:

histogram_quantile(
  0.95,
  rate(http_request_duration_seconds_bucket[5m])
)

4. Summary

A Summary also measures observations such as request duration.

It can provide quantiles/percentiles.

Example:

http_request_duration_summary_seconds


It can expose:

quantile="0.5"   → P50
quantile="0.9"   → P90
quantile="0.99"  → P99
-------------------------------------------------------------------------
Labels

Metrics can have labels.

Example:

http_requests_total{
    method="GET",
    path="/",
    status_code="200"
}


Here:

method      → GET
path        → /
status_code → 200


Labels allow us to filter and separate metrics by different dimensions.

For example:

http_requests_total{status_code="500"}


This shows HTTP 500 requests.

Day-4 Practical Application

In the Day-4 project, Service A has custom Prometheus instrumentation.

The application creates:

http_requests_total
http_request_duration_seconds
http_request_duration_summary_seconds
node_gauge_example


The application also provides endpoints such as:

/
/healthy
/serverError
/notFound
/logs
/example
/crash
/metrics
/call-service-b
-------------------------------------------------------------------------------

The important endpoint for Prometheus is:

/metrics


We tested it and received output such as:

# HELP http_requests_total Total number of HTTP requests
# TYPE http_requests_total counter

http_requests_total{
    method="GET",
    path="/",
    status_code="200"
} 1


This proves that the application is generating and exposing custom metrics.

How Prometheus Gets Application Metrics

> Prometheus does not automatically know about every application.

We need to configure discovery/scraping.

In Kubernetes, we can use a ServiceMonitor with the Prometheus Operator.

The flow is:

Node.js Application
        ↓
/metrics
        ↓
Kubernetes Service
        ↓
ServiceMonitor
        ↓
Prometheus
        ↓
PromQL
        ↓
Grafana


The ServiceMonitor tells Prometheus:

"Find this Kubernetes Service and scrape its metrics endpoint."

After the ServiceMonitor is configured, we can verify the target in:

Prometheus
→ Status
→ Targets

> see folder for the deplyment of service monitor

# kubectl apply -k kubernetes-manifest/
# kubectl apply -k kubernetes-manifest/

-------------------------------------------------------

EMAIl ALERT

# configure app password in gmail and update the email-secret.yml with base64 format
# configure SMTP server in alertmanager-service.yml 
> kubectl apply  