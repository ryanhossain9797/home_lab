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