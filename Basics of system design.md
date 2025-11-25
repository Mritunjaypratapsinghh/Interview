# System Design – Detailed Notes

## 1. Introduction to System Design

System Design is the art of building **scalable**, **highly available**, and **fault-tolerant** systems.  
The goal is to understand how different components work together inside a real-world production environment.

### Topics covered:
- Load Balancers  
- API Gateways  
- Caching Layers  
- Queue Systems & Asynchronous Processing  
- Microservice Architecture  
- Cloud Concepts  
- Horizontal & Vertical Scaling  
- Rate Limiting  
- CDN (Content Delivery Networks)

---

## 2. Basic Architecture — Client and Server

At the highest level, any system consists of two major components:

### 1) Client
- Devices: mobile phones, laptops, tablets, IoT devices  
- Sends HTTP requests to the server  

### 2) Server
- A machine capable of running **24/7** with a **public IP address**  
- Cloud servers (AWS EC2, DigitalOcean, etc.) = remote always-on machines  
- Client must know the server’s IP to communicate  

---

## 3. DNS (Domain Name System)

Since IP addresses are hard to remember, DNS maps **domain names → IP addresses**.

**Example:**  
`amazon.com → 13.32.64.0`

### Process:
1. User types domain  
2. Browser sends request to DNS server  
3. DNS resolves domain → IP address (**DNS Resolution**)  
4. Browser uses IP to contact server  

DNS is a **global, decentralized directory**.

---

## 4. Problem: Server Overload

Every server has physical limits:
- CPU  
- RAM  

If traffic increases beyond capacity, it causes:
- Server overwhelmed  
- Out-of-memory crashes  
- High latency  
- Complete system failure  

This is why websites crash during:
- Exam result days  
- Big sales  
- Ticket launches  

---

## 5. Vertical Scaling (Scale Up)

Increase resources of a **single machine**:
- Add more CPU  
- Add more RAM  
- Add more Disk  

**Example:**  
`2 CPU / 4GB RAM → 64 CPU / 128GB RAM → 6000 CPU / 128TB RAM`

### Pros:
- Easy to implement  
- No application changes  

### Cons:
- Very expensive  
- Hardware limit exists  
- Requires downtime (restart)  
- Still a single point of failure  

**Not suitable** for companies like Amazon that cannot tolerate even **1 minute of downtime**.

---

## 6. Horizontal Scaling (Scale Out)

Add **multiple servers** instead of upgrading one.

**Example:**  
Add 3, 10, or 100 servers in parallel.

### Advantages:
- Zero downtime  
- Near-infinite scaling  
- High fault tolerance  
- Cheaper than one big machine  

### Problem:
Multiple servers = multiple IPs → **How to distribute traffic?**

### Solution: Load Balancer

---

## 7. Load Balancer

A load balancer sits between the client and servers.

### Responsibilities:
- Distribute traffic across servers  
- Perform health checks  
- Send traffic only to healthy instances  
- Provide a **single public IP**  

### Algorithms:
- Round Robin  
- Least Connections  
- Weighted Distribution  

On AWS: **Elastic Load Balancer (ELB)**

When new servers boot:
- LB checks health  
- Only then routes traffic  

---

## 8. Microservices Architecture

Break the system into smaller independent services.

### Example Services:
- Authentication  
- Orders  
- Payments  
- API Service  

Each service:
- Runs on multiple EC2 machines  
- Has its own load balancer  
- Scales independently  

---

## 9. API Gateway

API Gateway is the **single entry point** for all users.

### Responsibilities:
- Path-based routing  
  - `/auth → Auth LB`  
  - `/orders → Orders LB`  
  - `/payment → Payments LB`  
- Authentication  
- Rate limiting  
- Logging  
- Request/response transformation  

API Gateway → forwards request → correct microservice.

---

## 10. Synchronous vs Asynchronous Processing

Slow operations cause bottlenecks:

Examples:
- Sending 1M emails  
- Processing huge CSV files  

### Synchronous:
Caller waits → slow performance  

### Asynchronous:
Use **Queues + Workers**  
→ no waiting → high throughput  

---

## 11. Queues (SQS / RabbitMQ / Kafka)

A queue:
- Stores messages in FIFO  
- Decouples producer & consumer  
- Enables async processing  

### Flow:
1. Payment service publishes `payment_success`  
2. Message goes to queue  
3. Email worker pulls the message  
4. Sends email  
5. Deletes message  

### Queue Polling:
- **Short Polling:** frequent → costly  
- **Long Polling:** waits up to 10s → efficient  

### Queue-based Rate Limiting:
Limit worker to process **N messages/sec**  
→ avoids exceeding external API limits (e.g., Gmail).

---

## 12. Pub/Sub Model (SNS)

Queue: one-to-one  
Pub/Sub: **one-to-many**

### Example:
A payment event triggers:
- Email  
- SMS  
- WhatsApp  
- Analytics  

Used for:
- Notification fan-out  
- Event-driven systems  

---

## 13. Fan-Out Architecture

Combine **SNS + SQS**:

1. SNS publishes event  
2. Event delivered to multiple SQS queues  
3. Each queue has its own worker  

### Advantages:
- Reliable  
- Independent retries  
- DLQ support  

**Use case:** YouTube upload →  
transcoding, thumbnail generation, indexing, notifications.

---

## 14. Dead Letter Queue (DLQ)

If a message fails repeatedly:
- Move to DLQ  
- Avoid loss  
- Investigate or retry later  

**Queues = reliable, guaranteed delivery**  
**SNS alone = NOT guaranteed**  

---

## 15. Rate Limiting

Used to:
- Protect system  
- Prevent DDOS  
- Prevent abusive users  
- Protect downstream APIs  

### Strategies:
- Token Bucket  
- Leaky Bucket  
- Sliding Window  

When exceeded → **HTTP 429 (“Too Many Requests”)**

### Applied at:
- API Gateway  
- Load Balancer  
- Individual microservices  

---

## 16. Caching Layer (Redis)

Caching improves speed & reduces DB load.

### Flow:
1. Check Redis  
2. If available → return  
3. Else → query DB  
4. Save result to Redis  
5. Return response  

---

## 17. Database Scaling

Databases need scaling too.

### 1) Read Replicas
- Offload reads  
- Eventual consistency  

### 2) Primary DB
- Handles all writes  
- Strong consistency  

### 3) Sharding
Distribute database horizontally.

Shard keys:
- User ID  
- Tenant ID  
- Region  

---

## 18. Content Delivery Network (CDN)

CDNs like **CloudFront** cache static content globally.

### Functions:
- Serve content from **nearest location**  
- Reduce latency  
- Reduce bandwidth cost  
- Reduce server load  

### Flow:
1. User requests image  
2. CDN checks local cache  
3. If miss → fetch from origin  
4. Cache it  
5. Serve user  

---

## 19. Complete System Summary

A scalable system includes:

- DNS (domain resolution)  
- CDN (static caching)  
- API Gateway (routing + auth)  
- Load Balancer (traffic distribution)  
- Microservices (modular logic)  
- Autoscaling  
- Queues (async tasks)  
- Pub/Sub (event distribution)  
- Database scaling (replicas + sharding)  
- Cache (fast data access)  
- Monitoring (logs & metrics)  
- Rate limiting (abuse control)  

All components work together to form a **fault-tolerant, globally scalable** system.

---

## 20. Final Understanding

By mastering:
- LB  
- API Gateway  
- Cache  
- Queue  
- Pub/Sub  
- CDN  
- Scaling  
- Microservices  

You can design systems like:
**Amazon, Flipkart, Netflix, Uber**, etc.


# Interactive System Design Quiz

This repo includes:
- `system_design_notes.md` — detailed notes (Markdown)
- `docs/system_design_quiz.html` — **interactive quiz** (hosted via GitHub Pages)

👉 **Open the interactive quiz:** https://mritunjaypratapsinghh.github.io/Interview/system_design_quiz.html



