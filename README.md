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
```



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


## 🔹 What is Sharding?

Sharding is a database partitioning technique that splits a large dataset into smaller, more manageable pieces called shards.
Each shard holds a subset of the data and is often stored on a separate server or node.

Think of it as slicing a large pizza into slices so that many people can eat at once—similarly, sharding allows a database to scale by dividing its load.


## 📦 Why Sharding Is Needed

### Without sharding:

One database server handles everything (single point of failure, bottleneck)

Not scalable beyond a certain point

Performance degrades as data grows

### With sharding:

Load is distributed across multiple servers

Each shard can be optimized individually

Horizontal scaling becomes possible

### 🎯 How Sharding Works

You choose a sharding key (usually a unique ID or a user’s location), and then:

Shard 1 → Handles data for user_id % 3 == 0

Shard 2 → Handles data for user_id % 3 == 1

Shard 3 → Handles data for user_id % 3 == 2

### OR for a food delivery system:

Shard 1 (Mumbai) → Handles orders from Mumbai

Shard 2 (Delhi) → Handles orders from Delhi

Shard 3 (Bangalore) → Handles orders from Bangalore

### 🗺️ Types of Sharding

### Horizontal Sharding (most common)

→ Rows are divided based on a key (e.g., location, ID)

### Vertical Sharding

→ Tables are split based on features (e.g., one DB for orders, one for payments)

### Geo-Sharding

→ Based on geographical regions (used by Swiggy, Zomato, etc.)

### ✅ Benefits

Handles huge volumes of data

Reduces load on individual servers

Improves performance and availability

Enables regional scaling

### ⚠️ Challenges

More complex to implement and maintain

Cross-shard joins are difficult

Requires shard-aware application logic

Data movement between shards during scaling can be tricky

### 📌 Real-World Example (Swiggy)

Imagine Swiggy serves all of India:

All users and orders from Mumbai go to Shard 1 (Mumbai DB)

All from Delhi go to Shard 2 (Delhi DB)

All from Bangalore go to Shard 3

If one region gets heavy traffic (like Mumbai during monsoon), the other shards remain unaffected.


