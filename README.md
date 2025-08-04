# SOC Zeek Lab

This repository contains a hands‑on Security Operations Center (SOC) lab built around [Zeek](https://zeek.org/) for network traffic analysis, log collection, and visualization with the Elastic Stack.

## Table of Contents

* [Overview](#overview)
* [Prerequisites](#prerequisites)
* [Lab Setup](#lab-setup)

  * [Zeek Installation](#zeek-installation)
  * [Running Zeek](#running-zeek)
* [Log Files](#log-files)
* [Elastic Stack Integration](#elastic-stack-integration)

  * [Filebeat](#filebeat)
  * [Elasticsearch](#elasticsearch)
  * [Kibana](#kibana)
  * [Sample Visualizations](#sample-visualizations)
* [Repository Structure](#repository-structure)
* [Usage](#usage)
* [Contributing](#contributing)
* [License](#license)

## Overview

This lab demonstrates:

1. Deploying Zeek on a Linux host to monitor network interface traffic.
2. Generating Zeek log files (`conn.log`, `dns.log`, `http.log`, etc.).
3. Shipping logs to Elasticsearch via Filebeat.
4. Building visualizations in Kibana using Lens and dashboards.
5. Basic incident detection use cases (e.g., identifying rejected connections, DNS anomalies).

## Prerequisites

* Ubuntu 20.04+ or compatible Linux for Zeek
* Zeek 4.x installed (from source or packages)
* Elasticsearch 7.x or 8.x
* Kibana 7.x or 8.x
* Filebeat 7.x or 8.x
* Git and GitHub account

## Lab Setup

### Zeek Installation

```bash
# Clone and build Zeek (if not installed via package)
cd /usr/local/src
sudo git clone --recursive https://github.com/zeek/zeek.git
cd zeek
sudo ./configure --prefix=/usr/local/zeek
sudo make
sudo make install
```

### Running Zeek

```bash
# Define log output directory
export ZEEK_OUT=/home/ubuntu/zeek-logs
mkdir -p "$ZEEK_OUT"

# Start Zeek on interface enp0s1
sudo /usr/local/zeek/bin/zeek -i enp0s1 -C local.zeek
```

Logs will appear under `$ZEEK_OUT/logs/current/`.

## Log Files

* `conn.log` – Connection summaries (TCP, UDP)
* `dns.log` – DNS queries and responses
* `http.log` – HTTP requests
* `ssl.log` – SSL/TLS handshakes
* `weird.log` – Anomalous traffic
* `packet_filter.log` – Raw packet filtering info

## Elastic Stack Integration

### Filebeat

Configure Filebeat to monitor your Zeek log directory:

```yaml
filebeat.inputs:
- type: log
  paths:
    - "/home/ubuntu/zeek-logs/logs/current/*.log"

setup.kibana:
  host: "http://localhost:5601"

output.elasticsearch:
  hosts: ["localhost:9200"]
```

```bash
sudo filebeat setup
sudo systemctl enable --now filebeat
```

### Elasticsearch

Data is indexed into `filebeat-*` indices.

### Kibana

* Use Lens to create visualizations.
* Build dashboards for SOC metrics (e.g., top source IPs, connection states).

### Sample Visualizations

* **Rejected vs Established Connections**: Stacked bar of `conn_state` counts by destination IP.
* **Top DNS Query Names**: Pie chart of `dns.qry_name` frequencies.
* **HTTP Status Codes**: Bar chart of `http.status_code` counts.

## Repository Structure

```
SOC-Zeek-Lab/
├── conn.log
├── dns.log
├── dhcp.log
├── http.log
├── ntp.log
├── packet_filter.log
├── quic.log
├── reporter.log
├── ssl.log
├── weird.log
├── local.zeek            # Custom scripts & site policies
├── filebeat.yml          # Filebeat configuration
└── README.md             # This file
```

## Usage

1. Clone this repo and navigate into it:

   ```bash
   git clone https://github.com/Kishpud/SOC-Zeek-Lab.git
   cd SOC-Zeek-Lab
   ```
2. Install prerequisites and start Zeek.
3. Run Filebeat to ship logs into Elasticsearch.
4. Open Kibana at `http://localhost:5601` and load `filebeat-*` index.
5. Build or import dashboards.

## Contributing

Contributions welcome! Feel free to submit issues or pull requests.

## License

This lab is released under the MIT License. See [LICENSE](LICENSE) for details.
