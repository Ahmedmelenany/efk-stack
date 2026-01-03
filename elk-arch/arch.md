# ELK Stack Architecture

## Recommended Approach

This document describes a **production-ready ELK Stack (Elasticsearch, Logstash, Kibana)** architecture and provides step-by-step guidance for **installing, configuring, and validating** the cluster.

The recommended and supported deployment for stability, scalability, and high availability is:

* **Elasticsearch:** 3 nodes (master-eligible + data)
* **Logstash:** 2 nodes (for high availability and load distribution)
* **Kibana:** 2 nodes (behind a load balancer)

This architecture ensures:

* Fault tolerance (node failure without downtime)
* Horizontal scalability
* Separation of responsibilities
* Production-grade performance

---

## Architecture Overview

### Component Roles

| Component     | Nodes | Purpose                                              |
| ------------- | ----- | ---------------------------------------------------- |
| Elasticsearch | 3     | Data storage, indexing, search, cluster coordination |
| Logstash      | 2     | Ingest, parse, enrich logs                           |
| Kibana        | 2     | Visualization and dashboards                         |

### High-Level Flow

```
Applications / Systems
        |
        v
   Logstash (2 nodes)
        |
        v
Elasticsearch Cluster (3 nodes)
        |
        v
 Kibana (2 nodes) -> Load Balancer
```

---

## Prerequisites

### Operating System

* Linux (Ubuntu 20.04/22.04 or RHEL/CentOS 8/9 recommended)

### Hardware (Minimum)

**Elasticsearch Nodes (per node)**

* CPU: 4 cores
* RAM: 16 GB (8 GB heap)
* Disk: SSD, minimum 100 GB

**Logstash Nodes**

* CPU: 2–4 cores
* RAM: 8 GB

**Kibana Nodes**

* CPU: 2 cores
* RAM: 4–8 GB

### System Settings (All Nodes)

```bash
# Disable swap
sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab

# Increase virtual memory
sudo sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

---

## Version Compatibility

Use the **same Elastic Stack version** for all components.

Example:

* Elasticsearch: 8.x
* Logstash: 8.x
* Kibana: 8.x

---

## Elasticsearch Installation & Configuration

### Install Elasticsearch (All ES Nodes)

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt update && sudo apt install elasticsearch -y
```

### Elasticsearch Node Roles

All 3 nodes will be:

* master-eligible
* data nodes

### Example `elasticsearch.yml`

```yaml
cluster.name: elk-prod-cluster
node.name: es-node-1

network.host: 0.0.0.0
http.port: 9200

# Discovery
discovery.seed_hosts:
  - es-node-1-ip
  - es-node-2-ip
  - es-node-3-ip

cluster.initial_master_nodes:
  - es-node-1
  - es-node-2
  - es-node-3

# Paths
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch

# Security
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
```

> Update `node.name` and IPs on each node.

### JVM Heap Configuration

Edit `/etc/elasticsearch/jvm.options.d/heap.options`:

```
-Xms8g
-Xmx8g
```

Rule: **Heap = 50% of RAM, max 32GB**

### Start Elasticsearch

```bash
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
```

### Validate Cluster

```bash
curl -k -u elastic https://es-node-1-ip:9200/_cluster/health?pretty
```

Status should be `green` or `yellow`.

---

## Logstash Installation & Configuration

### Install Logstash (Both Nodes)

```bash
sudo apt install logstash -y
```

### Logstash Pipeline Example

Create `/etc/logstash/conf.d/logstash.conf`

```conf
input {
  beats {
    port => 5044
  }
}

filter {
  if [log][level] {
    mutate { lowercase => ["[log][level]"] }
  }
}

output {
  elasticsearch {
    hosts => ["https://es-node-1-ip:9200","https://es-node-2-ip:9200","https://es-node-3-ip:9200"]
    index => "logs-%{+YYYY.MM.dd}"
    user => "logstash_writer"
    password => "<password>"
    ssl => true
    cacert => "/etc/logstash/certs/ca.crt"
  }
}
```

### Logstash Settings (`logstash.yml`)

```yaml
node.name: logstash-1
http.host: "0.0.0.0"
```

### Start Logstash

```bash
sudo systemctl enable logstash
sudo systemctl start logstash
```

---

## Kibana Installation & Configuration

### Install Kibana (Both Nodes)

```bash
sudo apt install kibana -y
```

### Kibana Configuration (`kibana.yml`)

```yaml
server.host: "0.0.0.0"
server.port: 5601

elasticsearch.hosts:
  - "https://es-node-1-ip:9200"
  - "https://es-node-2-ip:9200"
  - "https://es-node-3-ip:9200"

elasticsearch.username: "kibana_system"
elasticsearch.password: "<password>"

xpack.security.enabled: true
```

### Start Kibana

```bash
sudo systemctl enable kibana
sudo systemctl start kibana
```

---

## Load Balancer Configuration (Recommended)

* Place a load balancer in front of Kibana nodes
* Use round-robin or least-connections
* Health check on `/api/status`

Example:

```
Kibana URL: https://kibana.example.com
```

---

## Security Best Practices

* Enable TLS for all components
* Use built-in Elastic users and roles
* Rotate credentials regularly
* Restrict network access using firewalls/security groups

---

## Monitoring & Maintenance

* Monitor Elasticsearch disk usage
* Set index lifecycle policies (ILM)
* Regular snapshots to object storage
* Monitor JVM heap and GC

---


## Notes

* Always scale Elasticsearch before Logstash
* Avoid single-node clusters in production
* Test configuration changes in non-prod first

---

**This README can be used as the baseline document to provision and configure a full production ELK cluster.**
