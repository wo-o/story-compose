# Docker Compose Setup for Story

This `docker-compose.yml` file defines the setup for the Story Project, which
consists of multiple utilities for managing blockchain nodes, monitoring, and
data visualization. Below are detailed explanations of each component, their
purpose, and how to use this setup.

## Prerequisites

- Docker and Docker Compose must be installed on your system.
- Set environment variables before running the services (you can create a `.env`
  file or export them directly).
  - `NETWORK` to specify the blockchain network configuration.
  - `IMAGE_TAG__STORY_GETH` for the `story-geth` service image version.
  - `IMAGE_TAG__STORY` for the `story` service image version.
  - `IMAGE_TAG__PROMETHEUS` for the `story-prometheus` image version.
  - `IMAGE_TAG__GRAFANA` for the `story-grafana` image version.

## Services Overview

### 1. story-geth

- **Container Name**: `story-geth`
- **Image**: `piplabs/story-geth:${IMAGE_TAG__STORY_GETH}`
- **Purpose**: This service runs a Geth (Go Ethereum) node for the Story
  blockchain.
- **Ports**:
  - HTTP-RPC: `8545`
  - WebSocket-RPC: `8546`
  - Metrics: `6060`
  - Authenticated RPC: `8551`
  - Discovery Ports: `30304/tcp`, `30303/udp`
- **Volumes**: Persists blockchain data at `db-story-geth`.
- **Logging**: Uses JSON file logging with max size of 10MB and max of 3 log
  files.

### 2. story

- **Container Name**: `story`
- **Image**: `piplabs/story:${IMAGE_TAG__STORY}`
- **Depends On**: `story-geth`
- **Purpose**: The main service for Story blockchain, interacting with the
  `story-geth` service.
- **Ports**:
  - HTTP-API: `1317`
  - P2P: `26656`
  - HTTP-RPC: `26657`
  - Metrics: `26660`
- **Volumes**:
  - Uses `db-story-geth` for blockchain data.
  - Stores story data in `db-story-data`.
  - Uses a custom `config.toml` configuration file.
- **Environment Variables**: Uses `NETWORK` to define the network settings.

### 3. story-prometheus

- **Container Name**: `story-prometheus`
- **Image**: `prom/prometheus:${IMAGE_TAG__PROMETHEUS}`
- **Purpose**: Collects metrics from the blockchain nodes for monitoring.
- **Ports**:
  - Prometheus Web UI: `9090`
- **Volumes**:
  - Uses a custom Prometheus configuration located at
    `./monitoring/prometheus.yml`.
  - Data is persisted at `db-story-prometheus`.
- **Profiles**: `monitoring`

### 4. story-grafana

- **Container Name**: `story-grafana`
- **Image**: `grafana/grafana:${IMAGE_TAG__GRAFANA}`
- **Depends On**: `story-prometheus`
- **Purpose**: Visualizes metrics collected by Prometheus for easy monitoring.
- **Ports**:
  - Grafana Web UI: `3000`
- **Volumes**:
  - Uses `db-story-grafana` to persist Grafana data.
  - Loads Grafana provisioning settings from `./monitoring/provisioning` and
    configuration from `./monitoring/grafana.ini`.
- **Profiles**: `monitoring`

## Usage Instructions

1. **Set Up Environment Variables**: Ensure you have defined the required
   environment variables (`IMAGE_TAG__STORY_GETH`, `IMAGE_TAG__STORY`,
   `IMAGE_TAG__PROMETHEUS`, `IMAGE_TAG__GRAFANA`, `NETWORK`). This can be done
   using a `.env` file.

2. **Run Services**: To start all services, use the following command:

   ```bash
   docker-compose up -d
   ```

   The `-d` flag runs the containers in detached mode.

3. **Access Services**:

   - Story Geth API: `http://localhost:8545`
   - Story API: `http://localhost:1317`
   - Prometheus Web UI: `http://localhost:9090`
   - Grafana Web UI: `http://localhost:3000`

4. **Monitoring Setup**: By default, `story-prometheus` and `story-grafana` are
   enabled under the `monitoring` profile. You can activate these services by
   specifying the profile:

   ```bash
   docker-compose --profile monitoring up -d
   ```

5. **Stopping the Services**: To stop all services, run:

   ```bash
   docker-compose down
   ```

   This will stop and remove all containers.

6. **Removing All Volumes**: To stop all services and delete all associated
   Docker volumes, use:
   ```bash
   docker-compose down -v
   ```
   Adding the -v option will remove all volumes, including db-story-geth,
   db-story-data, and any other volumes created by the services.

## Logging

All services are configured to use JSON file logging, with a maximum size of
10MB per log file and up to 3 files. Logs are stored within the container and
can be accessed with `docker logs <container_name>`.

## Volumes

Persistent data is stored in named volumes to ensure that data such as
blockchain state, Prometheus metrics, and Grafana configurations are retained
even if containers are removed. The following volumes are used:

- `db-story-geth`: Blockchain data for `story-geth`.
- `db-story-data`: Story blockchain data for `story`.
- `db-story-prometheus`: Metric data for Prometheus.
- `db-story-grafana`: Grafana configurations and dashboards.

## Notes

- Make sure to modify the `.env` file and configuration files (`prometheus.yml`,
  `grafana.ini`, etc.) according to your requirements before running the
  services.
- To update the images, pull the latest versions and recreate the containers:
  ```bash
  docker-compose pull
  docker-compose up -d --force-recreate
  ```

## Troubleshooting

- **Unable to Connect to Grafana or Prometheus**: Ensure that the `monitoring`
  profile is activated during service startup.
