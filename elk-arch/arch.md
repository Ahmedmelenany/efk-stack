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

**This README can be used as the baseline document to provision and configure a full production ELK cluster.**
