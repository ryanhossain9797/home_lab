# My Home Lab Logging Setup

This repository provides a ready-to-use logging pipeline using Docker Compose, ClickHouse, and Vector. It is designed for collecting, transforming, and storing syslog data (such as from MikroTik RouterOS devices) in an analytical database for efficient querying and analysis.

## Repository Structure

```
clickhouse-setup/
├── docker-compose.yml    # ClickHouse service definition
└── config/
    └── listen.xml        # Custom ClickHouse listen configuration
vector-setup/
├── docker-compose.yml    # Vector service definition
└── vector_config/
    └── vector.yaml       # Vector pipeline configuration
.gitignore               # Standard ignores for Docker, logs, data, and IDE files
README.md                # (This file) Overview and codebase explanation
```

## Components

### ClickHouse
- **Purpose:** High-performance analytical database for log storage and querying.
- **Configuration:**
  - `clickhouse-setup/docker-compose.yml` defines the ClickHouse container, persistent storage, environment variables, and network settings.
  - `clickhouse-setup/config/listen.xml` allows ClickHouse to listen on all interfaces (0.0.0.0).

### Vector
- **Purpose:** Collects syslog data, transforms it, and sends it to ClickHouse.
- **Configuration:**
  - `vector-setup/docker-compose.yml` defines the Vector container and its network.
  - `vector-setup/vector_config/vector.yaml` sets up a syslog source, remap transform for parsing and normalizing logs, and a ClickHouse sink for storage.

## How It Works

1. **Syslog Collection:**
   - Vector listens for syslog messages on UDP port 514.
2. **Log Transformation:**
   - Vector parses and normalizes incoming logs using a remap transform.
3. **Log Storage:**
   - Transformed logs are sent to ClickHouse, stored in the `routeros_logs.syslog` table (schema must be created manually).

## Usage Notes
- You must create a shared Docker network (`server_default_network`) and host storage directories as described in the original setup instructions.
- Set a strong password in both `clickhouse-setup/docker-compose.yml` and `vector-setup/vector_config/vector.yaml` before deploying.
- The ClickHouse database and table schema must be created manually after the containers are running.

## Management
- Use `docker compose up -d` and `docker compose down` in each service directory to start/stop services.
- Logs are persisted on the host for durability.

For more details on extending or customizing the pipeline, refer to the configuration files in each service directory.
