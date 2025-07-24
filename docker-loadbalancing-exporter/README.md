# Load-Balancing Exporter Setup

This directory contains a Docker Compose setup that implements the OpenTelemetry Collector load-balancing exporter pattern as described in the [OpenTelemetry documentation](https://opentelemetry.io/docs/collector/deployment/gateway/#load-balancing-exporter).

                    ┌─────────────────────────────┐
                    │     Telemetry Generator     │
                    └─────────────┬───────────────┘
                                  ▼
        ┌────────────────────────────────────────────────────────┐
        │           Collector with Loadbalancing Exporter        │
        │                           (lb)                         │
        │                  4317 gRPC / 4318 HTTP                 │
        └─────────────┬─────────────┬──────────────────────┬─────┘
                      │             │                      │
          ┌───────────▼──┐     ┌────▼────────┐     ┌───────▼───────┐
          │     gw1      │     │     gw2     │     │      gw3      │
          │   Collector  │     │  Collector  │     │   Collector   │
          └──────┬───────┘     └─────┬───────┘     └──────┬────────┘
                 │                   │                    │
                 └───────────────────┼────────────────────┘
                                     ▼
                    ┌──────────────────────────────┐
                    │        External Gateway      │
                    │          (external-gw)       │
                    └───────────────┬──────────────┘
                                    ▼
                    ┌──────────────────────────────┐
                    │     Backend Systems          │
                    │  (Observability Stack etc.)  │
                    └──────────────────────────────┘

## Quick Start

[Install Bindplane CLI](https://bindplane.com/docs/advanced-setup/cli/installation).

Apply resources:

```bash
bindplane apply ./bindplane-resources/*.yaml
```

## Architecture

The setup uses a two-tier architecture:

### First Tier: Load-Balancing Gateway
- **Service**: `lb`
- **Purpose**: Receives OTLP traffic from applications and distributes it to second-tier collectors
- **Configuration**: Uses the `loadbalancing` exporter with static resolver
- **Endpoints**: 
  - OTLP gRPC: `0.0.0.0:4317`
  - OTLP HTTP: `0.0.0.0:4318`

### Second Tier: Processing Gateways
- **Services**: `gw1`, `gw2`, `gw3`
- **Purpose**: Process telemetry data and send to backends
- **Configuration**: Standard Bindplane agent configuration with OTLP receivers
- **Load Balancing**: Traffic distributed by trace ID (default routing key)

## Key Features

1. **Trace-Aware Load Balancing**: Uses trace ID as the routing key to ensure all spans for a given trace reach the same collector instance
2. **High Availability**: Multiple collector instances provide redundancy
3. **Scalability**: Easy to add more second-tier collectors by updating the static resolver configuration
4. **Monitoring**: Load-balancing exporter emits metrics for health and performance monitoring

## Configuration Files

- `config/lb-config.yaml`: Configuration for the first-tier load-balancing gateway
- `config/gw-config.yaml`: Configuration for second-tier collectors (gw1, gw2, gw3)
- `docker-compose.yaml`: Docker Compose services definition

## Usage

Start the services:

```bash
docker-compose up -d
```
