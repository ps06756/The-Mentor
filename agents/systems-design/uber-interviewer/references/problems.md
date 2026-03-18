# Uber/Ride-Sharing — Problem Bank, Walkthroughs & Calculations

---

## 📝 Complete System Design Walkthrough

### 1. Requirements & Back-of-the-Envelope
- **Scale**: 1M active drivers, 10M active riders.
- **Throughput**: 1M drivers * 1 ping / 4 sec = 250,000 writes/sec.
- **Latency**: Location update < 100ms. Dispatch matching < 2-3 seconds.
- **Storage**: Real-time locations in memory. Historical trips in persistent DB.

### 2. Location Ingestion Service
- Drivers connect via WebSockets or MQTT.
- Gateway publishes to Kafka (partitions by driver_id).
- Location Consumer updates Redis (TTL 30s to auto-expire offline drivers).
- Background worker updates Quadtree/Geohash index.

### 3. Geospatial Index (The "Map" Service)
- **Data Structure**: Redis Geospatial commands (GEOADD, GEORADIUS) or an in-memory Quadtree cluster.
- **Sharding**: Shard the Quadtree by City or Region. New York doesn't need to know about drivers in San Francisco.

### 4. Matching & Dispatch Service
1. Rider requests ride.
2. Matching Service queries Map Service for drivers within 2km.
3. Ranks drivers (ETA, direction of travel).
4. Sends proposal to Driver 1 via WebSocket.
5. If Driver 1 accepts, lock driver state. If reject/timeout, try Driver 2.

### 5. Trip State Machine
- Database: Cassandra or DynamoDB (highly available).
- States: REQUESTED -> DRIVER_ASSIGNED -> ARRIVED -> EN_ROUTE -> COMPLETED.
