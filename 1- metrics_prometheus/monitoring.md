
📊 Metrics in Prometheus:
Metrics in Prometheus are the core data objects that represent measurements collected from monitored systems.
These metrics provide insights into various aspects of system performance, health, and behavior.

🏷️ Labels:
Metrics are paired with Labels.
Labels are key-value pairs that allow you to differentiate between dimensions of a metric, such as different services, instances, or endpoints.

## container_cpu_usage_seconds_total{namespace="kube-system", endpoint="https-metrics"}

> http://kube-state-metrics-IP:8080/metrics
> http://prometheus-node-exporter-Ip:9100/metrics

------------------
Prometheus UI:
> kubectl port-forward service/prometheus-operated -n monitoring 9090:9090

Here you can PromQL:
-   go through the complete metrics get from above 2 url, and try different metrics in promql quries based on name spaces.

#	Metric	                                     What it tells you	                        Typical use
1	container_cpu_usage_seconds_total	        Container CPU usage	                    Find CPU-hungry containers
2	node_memory_MemAvailable_bytes	            Available node memory	                Detect memory pressure
3	node_filesystem_avail_bytes	                Free disk space	Detect                  disk-full problems
4	node_filesystem_size_bytes	                Total filesystem size	                Calculate disk utilization
5	node_network_receive_bytes_total	        Network traffic received	            Monitor incoming traffic
6	node_network_transmit_bytes_total	        Network traffic sent	                Monitor outgoing traffic
7	kube_pod_container_status_restarts_total	Container restart count	                Detect crashing/unhealthy containers

⚙️ Aggregation & Functions in PromQL
Aggregation in PromQL allows you to combine multiple time series into a single one, based on certain labels.

Sum Up All CPU Usage:

sum(rate(node_cpu_seconds_total[5m]))
This query aggregates the CPU usage across all nodes.
Average Memory Usage per Namespace:

avg(container_memory_usage_bytes) by (namespace)
This query provides the average memory usage grouped by namespace.
rate() Function:

The rate() function calculates the per-second average rate of increase of the time series in a specified range.
rate(container_cpu_usage_seconds_total[5m])
This calculates the rate of CPU usage over 5 minutes.
increase() Function:

The increase() function returns the increase in a counter over a specified time range.
increase(kube_pod_container_status_restarts_total[1h])
This gives the total increase in container restarts over the last hour.
-----------------------------------------------------


Grafana UI:

>kubectl port-forward service/monitoring-grafana -n monitoring 8080:80

-   We can use data source as prometheus for metrics visualization.
    queries can be same as promQL

    