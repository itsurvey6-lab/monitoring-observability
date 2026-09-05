# OpenTelemetry (OTel) – Quick Notes
What is OpenTelemetry?

OpenTelemetry is an open-source framework used to collect Metrics, Logs, and Traces from applications and send them to monitoring tools.

Core Components
---------------
> API

The interface developers use in code.

Example:

Start Trace
Create Span
Add Attributes

API defines WHAT to do.

> SDK

The actual implementation.

SDK:

Creates Trace IDs
Creates Span IDs
Records timings
Collects telemetry
Sends data to exporters

> Instrumentation

Adding telemetry collection to an application.

Manual Instrumentation

Developer writes code.

Create Span
Add Attributes
End Span

> Exporter

Responsible for sending telemetry data out.

Examples:

OTLP Exporter
Jaeger Exporter
Prometheus Exporter

> OpenTelemetry Collector

Central component that receives telemetry from applications.

Collector can:

Receive
Buffer
Batch
Filter
Transform
Route

telemetry data before sending it to monitoring systems.

----------------------------------------------------------------------------
                                        End-to-End Flow

                                        User Request
                                            |
                                            v
                                        Application
                                            |
                                            v
                                        OpenTelemetry API
                                            |
                                            v
                                        OpenTelemetry SDK
                                            |
                                            | Creates Trace ID
                                            | Creates Span IDs
                                            | Collects Metrics/Logs/Traces
                                            |
                                            v
                                        OTLP Exporter
                                            |
                                            v
                                        OpenTelemetry Collector
                                            |
                                            | Buffer
                                            | Batch
                                            | Filter
                                            | Route
                                            |
                                            +----------------+
                                            |                |
                                            v                v
                                        Metrics          Logs
                                        Prometheus         Loki

                                            |
                                            v
                                            Traces
                                        Tempo / Jaeger

                                            |
                                            v
                                            Grafana

# One-Line Summary
--------------------

Developers instrument the application using OpenTelemetry.
The SDK generates telemetry (metrics, logs, traces),
the exporter sends it to the OpenTelemetry Collector,
and the Collector processes and forwards it to monitoring
tools like Prometheus, Loki, Tempo, Jaeger, and Grafana.