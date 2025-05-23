# Install Fluentd un kubernetes

- Fluentd is an open-source data collector designed for processing logs and events. It's highly flexible, allowing you to collect data from multiple sources, transform it as needed, and send it to various destinations.

## Deploying Fluentd as a DaemonSet

- A DaemonSet ensures that a copy of the pod runs on each node in the cluster. This is perfect for log collection, as we want Fluentd to collect logs from every node.

- At first, we create a Fluentd configuration file specifies how logs should be collected and forwarded.

```conf
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>
<match **>
  @type elasticsearch
  host elasticsearch.default.svc.cluster.local
  port 9200
  logstash_format true
</match>
```

- `source` and `match` are directives. Following is a list of all directives and the respective aspects of logging they relate to:

```console
source: Input sources.
match: Output destinations.
filter: Event processing pipelines.
system: System-wide configuration.
label: Group the output and filter for internal routing.
worker: Limit to the specific workers.
@include: Include other files.
```
- Fluentd needs to be configured to collect logs from both Kubernetes nodes and pods. The configuration we defined earlier sets Fluentd to listen for logs and forward them to Elasticsearch.

- This has been made possible because of the use of output plugin `fluent-plugin-elasticsearch` which is used to forward data to Elasticsearch. The match section displays the parameters it uses.

### Collecting Node Logs

- The DaemonSet configuration mounts the `/var/log` directory from the host into the Fluentd container, allowing Fluentd to access and collect system and application logs from the nodes.

### Collecting Pod Logs

- Similarly, mounting `/var/lib/docker/containers` enables Fluentd to collect logs from Docker containers, including those managed by Kubernetes pods.

### Access the ES

- Access elastic search for viewing the logs with this path `http://localhost:nodeport-num/_search?q=*:*&size=10000&pretty`

