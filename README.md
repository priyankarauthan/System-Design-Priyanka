# System-Design-Priyanka


# 🌐 Swiggy-like Nationwide Scalable System Architecture

## 🏛️ Tiered Architecture View

```plaintext
+---------------------------+
|   End Users (All Over India) |
+---------------------------+
             ↓
+---------------------------+
|     CDN (CloudFront / Akamai)   | ← Static content delivery (images, JS, CSS)
+---------------------------+
             ↓
+----------------------------+
|   Global Load Balancer (GSLB)   | ← Routes to nearest region (Mumbai, Delhi, etc.)
+----------------------------+
             ↓
        Region A (e.g., Mumbai)                        Region B (e.g., Delhi)
+---------------------------+                +---------------------------+
|     Regional Load Balancer  |              |     Regional Load Balancer  |
+---------------------------+                +---------------------------+
             ↓                                         ↓
     +-----------------+                          +------------------+
     | API Gateway (K8s) |                          | API Gateway (K8s) |
     +-----------------+                          +------------------+
             ↓                                         ↓
+------------------------------------+      +------------------------------------+
| Microservices (User, Order, etc.) |      | Microservices (User, Order, etc.) |
| Each region has own service copy  |      | Each region has own service copy  |
+------------------------------------+      +------------------------------------+
             ↓                                         ↓
    Regional Database (Sharded)               Regional Database (Sharded)
   (e.g., MySQL + Vitess or Yugabyte)       (e.g., MySQL + Vitess or Yugabyte)
             ↓                                         ↓
    Redis Cache + Kafka (Regional)         Redis Cache + Kafka (Regional)
             ↓                                         ↓
     Delivery Agent App + Maps API        Delivery Agent App + Maps API




# 🌍 Key Components for Country-Wide Scale

## 1. Global Load Balancer (GSLB)
- Routes user requests based on location (via IP) to the nearest data center.
- E.g., Mumbai users go to Mumbai region, Delhi to Delhi region, etc.

## 2. Regional Clusters
- Each major metro has a region: Mumbai, Bangalore, Delhi, Hyderabad, etc.
- Each region runs duplicate microservices, its own Redis cache, Kafka, and database shards.

## 3. Data Partitioning
- Use Geo-sharding:
  - Orders and users from Mumbai → Mumbai DB
  - Delhi → Delhi DB
- Use consistent hashing or location-based routing.

## 4. Global Services
- Some services like Search, Analytics, Reporting, and Admin dashboards may be centralized.
- These can consume Kafka events from all regions.

## 5. Replication and Backup
- Periodic cross-region DB replication for DR (Disaster Recovery).
- Kafka topics may be mirrored between regions using MirrorMaker or Confluent Replicator.

## 6. Real-time Tracking & Location Services
- Integrated with Google Maps API, updated every few seconds via WebSocket/Push.

## 7. Monitoring & Observability
- Use centralized systems like:
  - Prometheus + Grafana
  - ELK stack / Splunk
  - Jaeger/Zipkin for tracing.

## 🚦 Deployment Infrastructure
- Kubernetes (GKE/EKS/AKS) per region.
- Terraform or Pulumi for infrastructure as code.
- Helm for service deployment.
- Istio or Linkerd for service mesh (optional).

## 🔐 Security & Compliance
- Encrypted data in transit and at rest (TLS + DB encryption).
- OAuth2/JWT for authentication.
- Rate limiting and abuse protection at API gateway.

