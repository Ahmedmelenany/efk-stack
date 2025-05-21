# Install Elastic Search on Kubernetes

- Elasticsearch is a powerful open-source search and analytics engine designed for horizontal scalability, reliability, and real-time search. It's commonly used for log or event data analysis and full-text search.
Using Elasticsearch in conjunction with Kubernetes, allows it to reap benefits from Kubernetes' self-healing and scalability features.

## Using StatefulSet to deploy ES

- Stable Network Identity: Each pod gets a unique and stable hostname.
- Ordered, Graceful Deployment and Scaling: Pods are created and deleted in a predictable order.
- Persistent Storage: Each pod can be associated with its storage, which persists across pod rescheduling.

## Using PV and PVC to manage storage of ES

- Persistent Volume (PV): Represents a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes.
- Persistent Volume Claim (PVC): A request for storage by a user. It specifies size, and access modes, and can be linked to specific PVs.

## Using Services for network access

- Stable IP Addresses: Services provide stable IP addresses for pods.
- Load Balancing: Services can load-balance traffic to multiple pods.
- Service Discovery: Services allow other applications to discover and communicate with your Elasticsearch cluster through a stable endpoint.

