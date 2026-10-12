# Cloud Services

Scale applications across distributed infrastructure: caching strategies, cloud platforms (AWS, Azure, GCP), serverless computing, and data replication.

---

## Topics

### [Caching](caching.md)
**Cache-Aside - Write-Through - Write-Behind - Redis - Memcached - Cache Invalidation - Cache Stampede**  
Caching patterns: when to use each strategy, cache hit ratio optimization, TTL strategies, cache stampede prevention, distributed cache (Redis, Memcached), cache consistency.

### [AWS](aws.md)
**EC2 - RDS - S3 - Lambda - DynamoDB - CloudFront - Auto-Scaling - IAM - VPC**  
Amazon Web Services: compute (EC2, Lambda), storage (S3, EBS), databases (RDS, DynamoDB), networking (VPC, CloudFront), monitoring (CloudWatch), cost optimization.

---

## Quick Interview Prep

**Caching strategies:**
- **Cache-Aside (Lazy Loading)**: App checks cache → miss → fetch from DB → store in cache (most common)
- **Write-Through**: Write to cache + DB synchronously (consistent, slower, higher latency)
- **Write-Behind (Write-Back)**: Write to cache only → async write to DB (fast, eventual consistency, risk of data loss)

**Cache invalidation:**
- **TTL (Time-To-Live)**: Automatic expiration; trade-off between freshness and hits
- **Event-based**: Invalidate on data change (explicit management)
- **Cache Stampede**: Multiple requests miss simultaneously, overload DB; use locks or probabilistic early expiration

**AWS core services:**
- **EC2**: Virtual machines; autoscaling groups; elastic IPs; security groups (firewall rules)
- **RDS**: Managed relational databases; automated backups; read replicas for scalability
- **S3**: Object storage; buckets; versioning; CDN integration with CloudFront
- **Lambda**: Serverless functions; auto-scaling; pay per invocation; ideal for event-driven workloads
- **DynamoDB**: NoSQL key-value store; single-digit millisecond latency; auto-scaling
- **CloudFront**: CDN for content distribution; edge locations worldwide; cache control

**Scaling patterns:**
- **Horizontal Scaling**: Add more instances; distribute load; stateless services
- **Vertical Scaling**: Increase instance size; simpler, has limits
- **Auto-Scaling**: Trigger on CPU/memory/custom metrics; launch/terminate instances dynamically
