# NGINX Load Balancer Setup

This directory contains a Docker Compose setup that implements an NGINX load balancer pattern for distributing telemetry data across multiple OpenTelemetry Collector instances.

                    ┌─────────────────────────────┐
                    │     Telemetry Generator     │
                    └─────────────┬───────────────┘
                                  │
                                  │
                                  │
                    ┌─────────────▼─────────────┐
                    │     NGINX Load Balancer   │
                    │        (otlp-lb)          │
                    │   Ports: 4317 (gRPC)      │
                    │        4318 (HTTP)        │
                    └─────────────┬─────────────┘
                                  │
          ┌───────────────────────┼───────────────────────┐
          │                       │                       │
    ┌─────▼─────┐           ┌─────▼─────┐           ┌─────▼─────┐
    │   gw1     │           │   gw2     │           │   gw3     │
    │ Collector │           │ Collector │           │ Collector │
    └─────┬─────┘           └─────┬─────┘           └─────┬─────┘
          │                       │                       │
          └───────────────────────┼───────────────────────┘
                                  │
                    ┌─────────────▼─────────────┐
                    │    External Gateway       │
                    │    (external-gw)          │
                    └─────────────┬─────────────┘
                                  │
                    ┌─────────────▼─────────────┐
                    │      Backend Systems      │
                    │   (Observability Stack)   │
                    └───────────────────────────┘

## Quick Start

[Install Bindplane CLI](https://bindplane.com/docs/advanced-setup/cli/installation).

Apply resources:

```bash
bindplane apply ./bindplane-resources/*.yaml
```

## Architecture

The setup uses a two-tier architecture:

### First Tier: NGINX Load Balancer
- **Service**: `otlp-lb`
- **Purpose**: Receives OTLP traffic and distributes it across collector instances
- **Configuration**: Uses NGINX round-robin load balancing
- **Endpoints**:
  - OTLP gRPC: `0.0.0.0:4317`

### Second Tier: External Gateway Collectors
- **Services**: `gw1`, `gw2`, `gw3`
- **Purpose**: Process telemetry data and forward to configured destinations
- **Configuration**: Uses Bindplane external gateway configuration with OTLP receivers
- **Load Balancing**: Traffic distributed evenly via NGINX round-robin

## Key Features

1. **Simple Load Distribution**: Round-robin distribution of telemetry data
2. **High Availability**: Multiple collector instances provide redundancy
3. **Scalability**: Easy to add more collectors behind NGINX
4. **Standard Protocol**: Uses OTLP gRPC for all communication

## Configuration Files

- `config/config.yaml`: Configuration for 3 gateway collector instances
- `config/external-gw-config.yaml`: External gateway configuration
- `nginx-otlp.conf`: NGINX load balancer configuration
- `docker-compose.yaml`: Docker Compose services definition

## Usage

Start the services:

```bash
docker-compose up -d
```
