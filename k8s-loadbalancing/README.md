# Kubernetes Load Balancing

This directory contains a Kubernetes setup of the OpenTelemetry Collector that implements K8s-native load balancing with horizontal pod autoscaling.

                    ┌─────────────────────────────┐
                    │     Telemetry Generator     │
                    │          Node Agent         │
                    └─────────────┬───────────────┘
                                  ▼
        ┌────────────────────────────────────────────────────────┐
        │                   Telemetry Generator                  │
        │                       Node Agent                       │
        │                 `node-agent-kind-1.yaml`               │
        └─────────────┬─────────────┬──────────────────────┬─────┘
                      │             │                      │
        ┌─────────────▼─────────────▼──────────────────────▼─────────────┐
        │    Gateway Collector Deployment w/ horiz autoscaler            │
        │    `gateway-collector-kind-1.yaml`                             │
        │  ┌──────────────┐     ┌─────────────┐       ┌───────────────┐  │
        │  │     Pod 1    │     │    Pod 2    │       │     Pod 10    │  │
        │  └──────────────┘     └─────────────┘  ...  └───────────────┘  │
        └────────────────────────────────────────────────────────────────┘
                 └───────────────────┼────────────────────┘
                                     ▼
                    ┌──────────────────────────────────────────┐
                    │           External Gateway               │
                    │       `gateway-collector-kind-2.yaml`    │
                    └───────────────┬──────────────────────────┘
                                    ▼
                    ┌──────────────────────────────┐
                    │     Backend Systems          │
                    │  (Observability Stack etc.)  │
                    └──────────────────────────────┘

## Quick Start

[Install kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl) and ensure you have access to a Kubernetes cluster. Install [kind](https://kind.sigs.k8s.io/docs/user/quick-start) for easy access to local clusters.

Apply resources:

```bash
bindplane apply ./bindplane-resources/*.yaml
```

## Architecture

The setup uses a single-tier architecture:

### Single Tier: Kubernetes-native Deployment w/ `HorizontalPodAutoscaler`
- **Deployment**: `bindplane-gateway-agent`
- **Purpose**: Receives OTLP traffic from applications, load-balances traffic, and distributes it to backends
- **Configuration**: Uses the native `HorizontalPodAutoscaler` to scale Deployment and a `ClusterIP` service for DNS access
- **Endpoints**: 
  - OTLP gRPC: `bindplane-gateway-agent.bindplane-agent.svc.cluster.local:4317`
  - OTLP HTTP: `bindplane-gateway-agent.bindplane-agent.svc.cluster.local:4318`

## Key Features

1. **Simple Load Distribution**: Round-robin distribution of telemetry data
2. **High Availability**: Multiple collector instances provide redundancy
3. **Scalability**: Easy to add more collectors behind with the `HorizontalPodAutoscaler`
4. **Standard Protocol**: Uses OTLP gRPC & HTTP for all communication

## Configuration Files

- `node-agent-kind-1.yaml`: Configuration for an Agent-mode collector acting as a telemetry generator
- `gateway-collector-kind-1.yaml`: Internal gateway configuration w/ load balancer
- `gateway-collector-kind-2.yaml`: External gateway configuration (external cluster)
- `gateway-nodeport-service.yaml`: Exposing the NodePort of the external gateway

## Usage

Create two clusters:

```bash
kind create cluster --name kind-1
kind create cluster --name kind-2
```

In all YAML files, replace the env vars for the ones provided in your Bindplane account Agent installation section.

Start the services:

```bash
kubectl config use-context kind-kind-1
kubectl apply -f node-agent-kind-1.yaml
kubectl apply -f gateway-collector-kind-1.yaml

kubectl config use-context kind-kind-2
kubectl apply -f gateway-collector-kind-2.yaml
kubectl apply -f gateway-nodeport-service.yaml
```
