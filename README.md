# My Home Lab Logging Setup

This repository contains the Docker Compose configurations and related configuration files for a ClickHouse and Vector logging pipeline.

## Components:

* **ClickHouse:** An analytical database for storing logs.
* **Vector:** A high-performance observability data router for collecting and sending logs to ClickHouse.

---

## **Instructions for AI Coding Agent**

*(This section is for generating the files. For manual setup, skip to "User Setup Instructions")*

Your task is to create a new Git repository containing the configuration files and Docker Compose setups for a ClickHouse and Vector logging pipeline.

**Repository Structure:**
```
my_home_lab_config/
├── .gitignore
├── README.md
├── clickhouse-setup/
│   ├── docker-compose.yml
│   └── config/
│       └── listen.xml
└── vector-setup/
├── docker-compose.yml
└── vector_config/
└── vector.yaml
```

**Step 1: Create the Root Directory and Basic Git Files**

* Create a new directory named `my_home_lab_config`.
* Inside `my_home_lab_config`, create a file named `.gitignore` with the following content:
    ```gitignore
    # Docker
    *.env
    .docker/
    docker-compose.override.yml
    docker-compose.*.yml
    !docker-compose.yml

    # Logs
    *.log
    logs/
    *.gz

    # Data
    data/
    *.db
    *.sqlite
    *.sqlite3

    # Operating System
    .DS_Store
    Thumbs.db

    # IDEs
    .idea/
    .vscode/
    *.iml
    ```
* Inside `my_home_lab_config`, this `README.md` file itself will be created with the content you are currently reading.

**Step 2: Create ClickHouse Setup Files**

* Create a directory named `clickhouse-setup` inside `my_home_lab_config`.
* Inside `clickhouse-setup`, create a file named `docker-compose.yml` with the following content:
    ```yaml
    version: '3.8'

    services:
      clickhouse:
        image: clickhouse/clickhouse-server:latest
        container_name: clickhouse-server
        restart: unless-stopped
        
        ports:
          - "8123:8123" # HTTP interface
          - "9000:9000" # Native TCP client protocol

        volumes:
          # Bind mount for ClickHouse data persistence
          - ~/docker_container_storage/clickhouse/data:/var/lib/clickhouse
          # Bind mount for ClickHouse logs persistence
          - ~/docker_container_storage/clickhouse/logs:/var/log/clickhouse-server
          # Bind mount for the custom listen_host configuration
          - ~/docker_container_storage/clickhouse/config/listen.xml:/etc/clickhouse-server/config.d/listen.xml:ro # 'ro' for read-only

        environment:
          CLICKHOUSE_USER: default
          CLICKHOUSE_PASSWORD: "YourStrongSecurePasswordHere" # IMPORTANT: REPLACE THIS!
          CLICKHOUSE_DEFAULT_ACCESS_MANAGEMENT: 1

        ulimits:
          nofile:
            soft: 262144
            hard: 262144

        cap_add:
          - SYS_NICE
          - NET_ADMIN
          - IPC_LOCK

        networks:
          - server_default_network # Attach this service to the shared network

    # Define the shared network as external at the top level
    networks:
      server_default_network:
        external: true # This tells Docker Compose that the network already exists
    ```
* Create a directory named `config` inside `clickhouse-setup`.
* Inside `clickhouse-setup/config`, create a file named `listen.xml` with the following content:
    ```xml
    <clickhouse>
        <listen_host>0.0.0.0</listen_host>
        </clickhouse>
    ```

**Step 3: Create Vector Setup Files**

* Create a directory named `vector-setup` inside `my_home_lab_config`.
* Inside `vector-setup`, create a file named `docker-compose.yml` with the following content:
    ```yaml
    version: '3.8'

    services:
      vector:
        image: timberio/vector:0.36.0-debian
        container_name: vector-syslog-clickhouse
        restart: unless-stopped
        
        ports:
          - "514:514/udp" # Map host port 514 UDP to container port 514 UDP for Syslog

        volumes:
          - ./vector_config/vector.yaml:/etc/vector/vector.yaml:ro # Mount your config file

        networks:
          - server_default_network # Attach this service to the same shared network

    # Define the shared network as external at the top level
    networks:
      server_default_network:
        external: true # This tells Docker Compose that the network already exists
    ```
* Create a directory named `vector_config` inside `vector-setup`.
* Inside `vector-setup/vector_config`, create a file named `vector.yaml` with the following content:
    ```yaml
    # vector-setup/vector_config/vector.yaml

    # --- Sources ---
    sources:
      routeros_syslog_input:
        type: syslog
        address: 0.0.0.0:514 # Listen for Syslog on all interfaces, UDP port 514
        mode: udp

    # --- Transforms ---
    transforms:
      parse_routeros_logs:
        inputs: ["routeros_syslog_input"]
        type: remap
        source: |
          . = parse_syslog!(.message)

          .timestamp_ch = parse_timestamp!(.timestamp, "%+")
          .hostname_ch = to_string(.host)
          .facility_ch = to_string(.facility)
          .severity_ch = to_string(.severity)
          .tag_ch = to_string(.program || .appname || "unknown_routeros")
          .message_ch = to_string(.message)
          .raw_log_ch = to_string(.raw_log)

          del(.message)
          del(.host)
          del(.facility)
          del(.severity)
          del(.program)
          del(.appname)
          del(.procid)
          del(.msgid)
          del(.version)
          del(.transport)
          del(.raw_log)

          .timestamp = .timestamp_ch
          .hostname = .hostname_ch
          .facility = .facility_ch
          .severity = .severity_ch
          .tag = .tag_ch
          .message = .message_ch
          .raw_log = .raw_log_ch

    # --- Sinks ---
    sinks:
      clickhouse_output:
        inputs: ["parse_routeros_logs"]
        type: clickhouse
        endpoint: "http://clickhouse:8123" # Use the service name 'clickhouse' for inter-container communication
        database: routeros_logs
        table: syslog
        compression: none # Start with none, then try lz4 for performance
        
        auth:
          strategy: basic
          user: default
          password: "YourStrongSecurePasswordHere" # IMPORTANT: REPLACE THIS!
    ```

**Step 4: Initialize Git Repository**

* Navigate to the root directory `my_home_lab_config`.
* Initialize a Git repository:
    ```bash
    git init
    ```
* Add all generated files to the staging area:
    ```bash
    git add .
    ```
* Make the initial commit:
    ```bash
    git commit -m "Initial setup for ClickHouse and Vector logging pipeline"
    ```

---

## **User Setup Instructions**

To get your logging pipeline running after the files are generated:

1.  **Navigate to the repository root:**
    ```bash
    cd my_home_lab_config
    ```

2.  **Create the shared Docker network:**
    This network allows ClickHouse and Vector (and other services) to communicate.
    ```bash
    docker network create server_default_network
    ```

3.  **Create host storage directories:**
    These directories will store ClickHouse data, logs, and custom configuration.
    ```bash
    mkdir -p ~/docker_container_storage/clickhouse/data
    mkdir -p ~/docker_container_storage/clickhouse/logs
    mkdir -p ~/docker_container_storage/clickhouse/config
    ```

4.  **Place ClickHouse `listen.xml`:**
    Ensure the `listen.xml` file is in `~/docker_container_storage/clickhouse/config/`. Its content is provided in `clickhouse-setup/config/listen.xml` in this repository.

5.  **Set your ClickHouse password:**
    **IMPORTANT:** Open `clickhouse-setup/docker-compose.yml` and `vector-setup/vector_config/vector.yaml`. Replace `"YourStrongSecurePasswordHere"` with a strong, unique password.

6.  **Deploy ClickHouse:**
    Navigate to the `clickhouse-setup` directory and start the service.
    ```bash
    cd clickhouse-setup
    docker compose up -d
    cd .. # Go back to repo root
    ```

7.  **Create ClickHouse Database and Table:**
    Connect to ClickHouse and run the SQL commands to create the `routeros_logs` database and `syslog` table.
    ```bash
    clickhouse-client --host 127.0.0.1 --port 9000 --user default --ask-password
    # (Enter your password)
    ```
    Then, in the ClickHouse client:
    ```sql
    CREATE DATABASE IF NOT EXISTS routeros_logs;

    CREATE TABLE routeros_logs.syslog
    (
        `timestamp` DateTime64(3) CODEC(DoubleDelta, ZSTD),
        `hostname` LowCardinality(String) CODEC(ZSTD),
        `facility` LowCardinality(String) CODEC(ZSTD),
        `severity` LowCardinality(String) CODEC(ZSTD),
        `tag` LowCardinality(String) CODEC(ZSTD),
        `message` String CODEC(ZSTD),
        `raw_log` String CODEC(ZSTD),
        `_timestamp_ingested` DateTime DEFAULT now() CODEC(Delta, ZSTD)
    )
    ENGINE = MergeTree()
    ORDER BY (timestamp, hostname)
    PARTITION BY toYYYYMM(timestamp)
    TTL toDateTime(`timestamp`) + INTERVAL 30 DAY
    SETTINGS index_granularity = 8192;
    ```

8.  **Deploy Vector:**
    Navigate to the `vector-setup` directory and start the service.
    ```bash
    cd vector-setup
    docker compose up -d
    cd .. # Go back to repo root
    ```

9.  **Send a test log:**
    From your Ubuntu host:
    ```bash
    logger -p local0.info -t ROUTEROS-TEST -n 127.0.0.1 -P 514 -u /dev/udp "May 23 21:50:00 RouterOS_Test_Device firewall: input: in:ether1 out:(none), src-mac 00:11:22:33:44:55, proto TCP (SYN), 192.168.1.1:1234->8.8.8.8:53, NAT (to 10.0.0.1:53)"
    ```

10. **Verify logs in ClickHouse:**
    ```bash
    clickhouse-client --host 127.0.0.1 --port 9000 --user default --ask-password
    # (Enter your password)
    SELECT * FROM routeros_logs.syslog WHERE tag = 'ROUTEROS-TEST' LIMIT 1;
    ```

## Management Commands:

* **Stop ClickHouse:** `cd clickhouse-setup && docker compose down`
* **Stop Vector:** `cd vector-setup && docker compose down`
* **Remove shared network:** `docker network rm server_default_network` (only if no containers are using it)