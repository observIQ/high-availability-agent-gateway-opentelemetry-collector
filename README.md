# High Availability Agent & Gateway Mode for the Opentelemetry Collector

This repository contains example code demonstrating how to implement high availability and gateway mode patterns with the OpenTelemetry Collector. The code shows how to:

- Set up redundant collectors for high availability
- Configure collectors in gateway mode to aggregate and forward telemetry data
- Handle failover between collectors
- Scale collectors horizontally

For a detailed walkthrough of the code and concepts, check out our blog post: [Building Resilient Observability Pipelines with OpenTelemetry Collector HA & Gateway Mode](https://bindplane.com/blog/opentelemetry-collector-ha-gateway-mode) (Note: Update with actual blog post URL)

## Quick Start

The examples are organized into the following directories:

- `/docker-nginx` - High Availability collector configuration using Nginx as a load balancer in Docker Compose. (Architecture compatible with VMs)
- `/docker-loadbalancing-exporter` - High Availability collector configuration using the OpenTelemetry Collectors `loadbalancing` exporter as a load balancer in Docker Compose. (Architecture compatible with VMs)
- `/k8s-loadbalancing` - Native Kubernetes load balancing for High Availability collector configuration. (Real-life use case)

Each directory contains a README with specific setup instructions.

## Prerequisites

- Docker and Docker Compose for running the local examples
- Two Kubernetes clusters for running the Kubernetes examples
- Basic familiarity with OpenTelemetry concepts

## How can we help?

If you need any additional help feel free to file a GitHub issue or reach out to us at support@bindplane.com.
