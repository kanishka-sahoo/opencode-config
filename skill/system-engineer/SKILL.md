---
name: system-engineer
description: Design and implement production-grade infrastructure, deployment pipelines, and cloud systems with focus on reliability, cost-efficiency, scalability, and operational excellence. Use this skill when working on infrastructure-as-code, CI/CD, Docker/Kubernetes, cloud architecture (AWS/GCP/Azure), monitoring, or system operations.
license: Complete terms in LICENSE.txt
---

This skill guides creation of **production-grade infrastructure that won't bankrupt you**. The primary goal is building reliable, secure systems while **aggressively minimizing costs** and preventing runaway spending.

## ⚠️ COST EXPLOSION PREVENTION - READ THIS FIRST

Cloud platforms can generate **massive unexpected bills** in hours. Your #1 priority is preventing cost disasters:

**MANDATORY COST CONTROLS:**
- Set **hard budget limits** with billing alerts at 50%, 75%, 90% of budget
- Configure **maximum spend limits** on all auto-scaling resources
- Set **strict autoscaling maximums** (never leave max unlimited)
- Implement **aggressive rate limiting** on all public endpoints
- Add **resource quotas** to prevent runaway resource creation
- Use **cost allocation tags** on every resource
- Enable **detailed billing/cost explorer** from day one

**CRITICAL AUTOSCALING RULES:**
- NEVER set max instances to unlimited or very high numbers
- ALWAYS set conservative maximums (e.g., max 10 instances to start)
- ALWAYS configure scale-down quickly (e.g., scale up in 30s, down in 2 min)
- ALWAYS set CPU/memory thresholds appropriately (e.g., 70% not 50%)
- ALWAYS add autoscaling cooldown periods to prevent flapping
- ALWAYS monitor autoscaling activity with alerts

**CRITICAL RATE LIMITING:**
- Rate limit ALL public endpoints (e.g., 100 req/min per IP)
- Implement request throttling at API Gateway/Load Balancer level
- Add DDoS protection (CloudFlare, AWS Shield, etc.)
- Set per-user/per-API-key rate limits
- Monitor and alert on unusual traffic spikes

The user provides infrastructure requirements: deployment pipelines, cloud architecture, scaling strategy, monitoring setup, or operational improvements. They may include cost constraints, compliance requirements, traffic expectations, or existing infrastructure limitations.

---

## Engineering Thinking

Before writing any configs or infrastructure code, **stop and think like a systems engineer**, not a template generator.

* **Purpose**
  What business problem requires infrastructure changes? What must be highly available? What can tolerate downtime?

* **Operational Reality**
  Traffic patterns, geographic distribution, latency requirements, disaster recovery objectives (RTO/RPO), on-call burden.

* **Cost Constraints**
  Budget ceiling, cost per user/request, reserved capacity vs on-demand, egress patterns, hidden charges.

* **Risk & Blast Radius**
  What breaks if this fails? What's the rollback plan? How do we test without affecting production?

**CRITICAL**: Infrastructure decisions are expensive to change. Poor choices waste thousands in cloud spend before you notice.

---

## System Design Principles

Infrastructure solutions must be:

* **Cost-optimized FIRST**. Every resource must justify its cost. Start small, scale based on real metrics.
* **Protected from runaway costs**. Hard limits, alerts, and circuit breakers prevent spending disasters.
* **Resilient and redundant**. But only where needed - avoid over-engineering for problems you don't have.
* **Observable and monitored**. You can't optimize costs if you can't measure them.
* **Reproducible as code**. Manual changes = untracked costs and configuration drift.
* **Secure by default**. Security breaches are expensive - prevent them upfront.

Infrastructure choices must be justified with cost/benefit analysis. If something is overkill, say so and offer a cheaper alternative. If something creates a single point of failure, call it out loudly.

---

## Cost Explosion Prevention Checklist

Before deploying ANY infrastructure, verify these safeguards are in place:

### Billing & Budget Controls
- [ ] Billing alerts configured (50%, 75%, 90%, 100% thresholds)
- [ ] Budget limits set with automated actions (e.g., SNS notifications, Lambda to shutdown non-critical resources)
- [ ] Cost allocation tags defined and enforced
- [ ] Daily cost monitoring dashboard created
- [ ] Anomaly detection alerts enabled (AWS Cost Anomaly Detection, GCP Budget alerts)

### Autoscaling Safeguards
- [ ] Maximum instance/pod counts set (start with 5-10 max)
- [ ] Scale-up thresholds set appropriately (70-80% CPU/memory)
- [ ] Scale-down configured aggressively (remove unused capacity quickly)
- [ ] Cooldown periods configured (prevent flapping)
- [ ] Autoscaling activity alerts configured
- [ ] Manual approval required for scaling beyond certain thresholds

### Rate Limiting & Traffic Control
- [ ] Rate limiting configured at load balancer/API gateway level
- [ ] Per-IP rate limits (e.g., 100-1000 req/min depending on use case)
- [ ] Per-user/API-key rate limits
- [ ] DDoS protection enabled (CloudFlare, AWS Shield, GCP Armor)
- [ ] Traffic spike alerts configured
- [ ] Circuit breakers implemented for downstream services

### Resource Quotas & Limits
- [ ] Kubernetes resource quotas set (if using K8s)
- [ ] Pod memory/CPU limits defined (no unbounded containers)
- [ ] Cloud provider service quotas reviewed and adjusted
- [ ] Storage growth limits (lifecycle policies, retention limits)
- [ ] Database connection limits configured
- [ ] Maximum concurrent Lambda/Cloud Function executions limited

### Cost Monitoring & Alerts
- [ ] Real-time cost monitoring dashboard
- [ ] Alerts for unexpected cost increases (>20% day-over-day)
- [ ] Resource utilization monitoring (identify waste)
- [ ] Regular cost review meetings scheduled (weekly for new projects)
- [ ] Cost optimization recommendations reviewed monthly

---

## Infrastructure & Cloud Architecture

### Multi-Cloud Awareness

* **AWS**: Mature ecosystem, most services, highest egress costs. Default for enterprise.
* **GCP**: Superior networking, better BigQuery/data tools, more generous free tier egress.
* **Azure**: Strong for .NET/Microsoft shops, good hybrid cloud, complex pricing.

Choose based on actual needs, not hype. Multi-cloud for the sake of it is expensive complexity.

### Compute Resources

**COST-FIRST APPROACH TO INSTANCE SELECTION:**

* **Right-Sizing**: Start small, measure, scale up based on data
  - ❌ "Let's use t3.2xlarge to be safe" ($121/month)
  - ✅ "Start with t3.medium ($30/month), monitor CPU/memory, scale if needed"
  - **Principle**: It's easier to scale up than to justify overspending

* **Spot/Preemptible Instances**: 70-90% savings for interruptible workloads
  - Use for: Batch jobs, CI/CD runners, dev environments, stateless workers
  - Example: c5.xlarge spot ($35/month) vs on-demand ($124/month)
  - **Savings: $89/month per instance (72% off)**

* **Reserved/Savings Plans**: 40-60% savings for steady-state workloads
  - Commit to 1-year or 3-year for predictable baseline capacity
  - Example: 5x t3.large on-demand: $300/month
  - Same with 1-year reserved: $180/month
  - **Savings: $120/month ($1,440/year)**

* **Graviton/Arm Instances**: 20-40% better price-performance
  - t4g.medium (Graviton2): $24/month
  - t3.medium (Intel): $30/month  
  - **Savings: $6/month (20% off) + better performance**
  - Compatible with most Linux workloads, Docker, etc.

**INSTANCE SIZING EXAMPLES:**

```hcl
# ❌ WASTEFUL - Oversized "just to be safe"
resource "aws_instance" "app_wasteful" {
  ami           = "ami-12345678"
  instance_type = "t3.xlarge"  # 4 vCPU, 16GB RAM - $121/month
  # App actually uses: 1 vCPU avg, 4GB RAM
  # WASTE: $91/month (75% wasted capacity)
}

# ✅ RIGHT-SIZED - Start appropriate, monitor, scale if needed
resource "aws_instance" "app_efficient" {
  ami           = "ami-12345678"
  instance_type = "t3.medium"  # 2 vCPU, 4GB RAM - $30/month
  
  monitoring = true  # Enable detailed monitoring
  
  tags = {
    CostOptimization = "rightsized"
    ReviewDate       = "2026-02-01"
  }
}

# ✅ EVEN BETTER - Graviton with spot for non-critical
resource "aws_instance" "app_optimized" {
  ami           = "ami-graviton"
  instance_type = "t4g.medium"  # Graviton2: 2 vCPU, 4GB - $24/month
  
  # Use spot for dev/staging (70% savings)
  instance_market_options {
    market_type = "spot"
    spot_options {
      max_price          = "0.012"  # 50% of on-demand price
      spot_instance_type = "persistent"
    }
  }
  
  # Spot price: ~$7/month (70% off)
}
```

**DEV/STAGING COST OPTIMIZATION:**

```python
# Lambda function to shut down dev/staging instances overnight
# Saves 65% on costs (12 hours/day instead of 24)

import boto3
from datetime import datetime

ec2 = boto3.client('ec2')

def lambda_handler(event, context):
    current_hour = datetime.now().hour
    
    # Filters for dev/staging instances
    filters = [
        {'Name': 'tag:Environment', 'Values': ['dev', 'staging']},
        {'Name': 'instance-state-name', 'Values': ['running']}
    ]
    
    # Stop at 7 PM
    if current_hour == 19:
        instances = ec2.describe_instances(Filters=filters)
        instance_ids = [
            i['InstanceId'] 
            for r in instances['Reservations'] 
            for i in r['Instances']
        ]
        if instance_ids:
            ec2.stop_instances(InstanceIds=instance_ids)
            print(f"Stopped {len(instance_ids)} instances")
    
    # Start at 7 AM
    elif current_hour == 7:
        filters[1] = {'Name': 'instance-state-name', 'Values': ['stopped']}
        instances = ec2.describe_instances(Filters=filters)
        instance_ids = [
            i['InstanceId'] 
            for r in instances['Reservations'] 
            for i in r['Instances']
        ]
        if instance_ids:
            ec2.start_instances(InstanceIds=instance_ids)
            print(f"Started {len(instance_ids)} instances")
    
    return {'statusCode': 200}

# EventBridge schedule: cron(0 7,19 * * ? *)
# Cost impact:
# - 3x t3.medium running 24/7: $90/month
# - Same instances 12h/day: $31/month
# - SAVINGS: $59/month ($708/year)
```

**KUBERNETES POD RESOURCE LIMITS (Prevent runaway costs):**

```yaml
# ❌ DANGEROUS - No limits (can consume entire node)
apiVersion: v1
kind: Pod
metadata:
  name: dangerous-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    # NO LIMITS = Can spike to 100% CPU/memory
    # Triggers autoscaling, costs explode

# ✅ SAFE - Explicit requests and limits
apiVersion: v1
kind: Pod
metadata:
  name: safe-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    resources:
      requests:
        cpu: "250m"      # Minimum: 0.25 CPU cores
        memory: "256Mi"  # Minimum: 256MB RAM
      limits:
        cpu: "500m"      # Maximum: 0.5 CPU cores
        memory: "512Mi"  # Maximum: 512MB RAM
    
    # Resource limits prevent:
    # - Single pod consuming entire node
    # - Unnecessary autoscaling triggers
    # - OOM kills affecting other pods

# Set namespace-level quotas to enforce limits
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: production
spec:
  hard:
    requests.cpu: "50"        # Max 50 CPU cores requested
    requests.memory: "100Gi"  # Max 100GB memory requested
    limits.cpu: "100"         # Max 100 CPU cores limit
    limits.memory: "200Gi"    # Max 200GB memory limit
    pods: "50"                # Max 50 pods
```

**REFUSE**: 
- t2.micro in production (CPU credits are a trap - throttles unexpectedly)
- Oversized instances "just to be safe" without monitoring data
- No auto-recovery configured (instances stay down = downtime + wasted money)
- Running dev/staging 24/7 (unnecessary 65% cost increase)
- No spot instances for batch/CI workloads (leaving 70% savings on table)
- Ignoring Graviton instances (missing 20-40% price-performance boost)

### Networking

**EGRESS COSTS: THE SILENT KILLER**

Data transfer OUT of cloud providers is where bills explode:
- AWS egress: $0.09/GB (first 10TB)
- GCP egress: $0.12/GB (first 1TB) 
- Azure egress: $0.087/GB (first 5GB)

**Example cost explosion:**
- 10TB/month egress from EC2: **$900/month**
- Same 10TB via CloudFront (90% cache hit): **$130/month**
- **Savings: $770/month ($9,240/year)**

**COST-SAVING STRATEGIES:**

```hcl
# ❌ EXPENSIVE - Serving static assets directly from EC2/S3
# - 5TB egress from S3: $450/month
# - No caching, every request hits origin
# - Slow for global users

# ✅ CHEAP - Use CloudFront CDN
resource "aws_cloudfront_distribution" "cdn" {
  enabled = true
  
  origin {
    domain_name = aws_s3_bucket.assets.bucket_regional_domain_name
    origin_id   = "S3-assets"
    
    s3_origin_config {
      origin_access_identity = aws_cloudfront_origin_access_identity.cdn.cloudfront_access_identity_path
    }
  }
  
  default_cache_behavior {
    allowed_methods        = ["GET", "HEAD", "OPTIONS"]
    cached_methods         = ["GET", "HEAD"]
    target_origin_id       = "S3-assets"
    viewer_protocol_policy = "redirect-to-https"
    compress               = true  # Enable gzip compression
    
    min_ttl     = 0
    default_ttl = 86400   # 24 hours
    max_ttl     = 31536000  # 1 year
    
    forwarded_values {
      query_string = false
      cookies {
        forward = "none"
      }
    }
  }
  
  # Price class 100 = US, Canada, Europe (cheapest)
  price_class = "PriceClass_100"  # vs PriceClass_All
  
  # Cost impact:
  # - 5TB CloudFront (90% cache hit): $65/month CloudFront + $45/month S3 origin = $110/month
  # - vs 5TB S3 direct: $450/month
  # - SAVINGS: $340/month ($4,080/year)
}
```

**VPC NETWORKING COSTS:**

```hcl
# ❌ EXPENSIVE - NAT Gateway per subnet
# - 3 subnets = 3 NAT Gateways
# - Cost: 3 × $32/month = $96/month base
# - Plus: $0.045/GB processed

# ✅ CHEAPER - NAT Gateway per AZ (not per subnet)
resource "aws_nat_gateway" "main_az1" {
  allocation_id = aws_eip.nat_az1.id
  subnet_id     = aws_subnet.public_az1.id
  
  tags = {
    Name = "nat-az1"
  }
}

resource "aws_nat_gateway" "main_az2" {
  allocation_id = aws_eip.nat_az2.id
  subnet_id     = aws_subnet.public_az2.id
  
  tags = {
    Name = "nat-az2"
  }
}

# Route tables point private subnets to NAT in same AZ
# Cost: 2 NAT Gateways = $64/month base
# Savings: $32/month vs NAT-per-subnet approach

# ✅ CHEAPEST - NAT Instance for low-traffic workloads
# - t4g.nano instance: $3/month
# - Good for: <5GB/day traffic
# - Savings: $29/month per AZ vs NAT Gateway
# - Trade-off: Slightly less reliable, needs management
```

**LOAD BALANCER COSTS:**

```hcl
# Cost comparison (all per month):
# - Application Load Balancer (ALB): $16 + $0.008/LCU-hour
# - Network Load Balancer (NLB): $33 + $0.006/NLCU-hour  
# - Classic Load Balancer (CLB): $18 + $0.008/GB - DEPRECATED

# ❌ WASTEFUL - NLB when ALB would work
resource "aws_lb" "expensive" {
  name               = "expensive-nlb"
  load_balancer_type = "network"  # $33/month base
  # NLB only needed for: TCP/UDP, extreme performance, static IPs
}

# ✅ COST-EFFECTIVE - ALB for HTTP/HTTPS
resource "aws_lb" "efficient" {
  name               = "efficient-alb"
  load_balancer_type = "application"  # $16/month base
  
  # ALB provides:
  # - Path-based routing
  # - Host-based routing  
  # - HTTP/2, WebSocket support
  # - WAF integration
  # - Better for 95% of use cases
  
  # Delete unused LBs immediately!
  # Idle LB = $16-33/month wasted
}

# ✅ EVEN CHEAPER - CloudFlare for simple static sites
# - Free tier includes DDoS protection, CDN, SSL
# - No load balancer needed
# - Savings: $16/month
```

**CROSS-REGION/CROSS-AZ DATA TRANSFER:**

```hcl
# Data transfer costs:
# - Same AZ: FREE
# - Cross-AZ (same region): $0.01/GB in, $0.01/GB out
# - Cross-region: $0.02/GB
# - To internet: $0.09/GB

# Example: Multi-AZ RDS with high traffic
# - 1TB/month replication between AZs: $20/month hidden cost
# - This is ON TOP of the Multi-AZ 2x instance cost

# ❌ EXPENSIVE - Unnecessary cross-region
resource "aws_db_instance" "read_replica_cross_region" {
  # Creates read replica in another region
  # - Instance cost: 2x
  # - Data transfer: $0.02/GB × replication volume
  # - Use only when: Global low-latency reads required
}

# ✅ COST-AWARE - Cross-AZ only when needed
resource "aws_db_instance" "primary" {
  multi_az = true  # For production HA
  # Acceptable cost for reliability
}

resource "aws_db_instance" "dev_single_az" {
  multi_az = false  # Dev doesn't need HA
  # Savings: 50% on instance + $0 cross-AZ transfer
}
```

**PRIVATE SUBNETS & SECURITY:**

```hcl
# Best practice: Private by default (also cheaper)
resource "aws_subnet" "private" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"
  
  # No direct internet access
  map_public_ip_on_launch = false
  
  tags = {
    Name = "private-subnet-az1"
    Type = "private"
  }
}

# Access via NAT Gateway (costs money) or:
# - VPC Endpoints for AWS services (cheaper, faster)
# - SSM Session Manager for SSH (free, no bastion needed)
# - VPN/Direct Connect for on-prem (expensive but necessary)
```

**VPC ENDPOINTS (Avoid NAT Gateway costs for AWS services):**

```hcl
# ❌ EXPENSIVE - All AWS API calls through NAT Gateway
# - EC2 → S3 through NAT: $0.045/GB
# - 1TB/month S3 uploads: $45/month NAT processing

# ✅ FREE - VPC Endpoints for AWS services
resource "aws_vpc_endpoint" "s3" {
  vpc_id       = aws_vpc.main.id
  service_name = "com.amazonaws.us-east-1.s3"
  
  route_table_ids = [aws_route_table.private.id]
  
  # Gateway endpoint (free!)
  # Supports: S3, DynamoDB
}

resource "aws_vpc_endpoint" "ecr_api" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.us-east-1.ecr.api"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = [aws_subnet.private.id]
  security_group_ids  = [aws_security_group.vpc_endpoints.id]
  
  # Interface endpoint (small cost)
  # - $0.01/hour per AZ = $7/month per AZ
  # - Savings: Avoid NAT Gateway processing fees
  # - Worth it for: High-volume AWS API usage
}

# Savings calculation:
# - 1TB S3 transfer without endpoint: $45/month (NAT processing)
# - 1TB S3 transfer with endpoint: $0/month (free gateway endpoint)
# - SAVINGS: $45/month ($540/year)
```

**REFUSE**: 
- Public subnets everywhere (security risk + no cost benefit)
- No network ACLs (security risk)
- Cross-region traffic "because we can" (expensive, high latency)
- Ignoring egress charges (the #1 surprise bill item)
- NAT Gateway per subnet (3x the cost vs per-AZ)
- No CDN for static assets (10x higher egress costs)
- Using NLB when ALB would work ($17/month more expensive)

### Storage

**COST-OPTIMIZED STORAGE STRATEGY:**

**Block Storage (EBS/Persistent Disks):**

```hcl
# ❌ EXPENSIVE - Old generation volumes
resource "aws_ebs_volume" "old_expensive" {
  availability_zone = "us-east-1a"
  size              = 100  # 100GB
  type              = "gp2"  # Old generation
  
  # Cost: $10/month (gp2: $0.10/GB-month)
}

# ✅ CHEAPER - gp3 (newer generation)
resource "aws_ebs_volume" "new_efficient" {
  availability_zone = "us-east-1a"
  size              = 100  # 100GB
  type              = "gp3"
  
  # Cost: $8/month (gp3: $0.08/GB-month)
  # Performance: 3000 IOPS baseline (same as gp2)
  # Savings: 20% cheaper + configurable IOPS
  
  # Only pay more for extra IOPS if needed
  iops       = 3000   # Baseline included
  throughput = 125    # Baseline included
}

# Size for IOPS needs, not just capacity
# gp3 gives 3000 IOPS baseline regardless of size
# - 100GB gp3: 3000 IOPS, $8/month
# - Don't overprovision: 500GB for 3000 IOPS wasteful
```

**Object Storage (S3/GCS) with Lifecycle Policies:**

```hcl
# ❌ WASTEFUL - All data in expensive hot storage
# - 1TB S3 Standard: $23/month
# - 50% accessed monthly, 50% rarely accessed

# ✅ COST-OPTIMIZED - Lifecycle policies
resource "aws_s3_bucket" "data" {
  bucket = "my-data-bucket"
}

resource "aws_s3_bucket_lifecycle_configuration" "data_lifecycle" {
  bucket = aws_s3_bucket.data.id

  rule {
    id     = "archive-old-data"
    status = "Enabled"
    
    # Move to Intelligent-Tiering after 30 days
    transition {
      days          = 30
      storage_class = "INTELLIGENT_TIERING"  # Auto-optimizes
    }
    
    # Move to Glacier after 90 days
    transition {
      days          = 90
      storage_class = "GLACIER"  # $0.004/GB-month (83% cheaper)
    }
    
    # Delete after 365 days (if appropriate)
    expiration {
      days = 365
    }
  }
  
  rule {
    id     = "delete-incomplete-uploads"
    status = "Enabled"
    
    abort_incomplete_multipart_upload {
      days_after_initiation = 7
    }
  }
  
  rule {
    id     = "clean-old-versions"
    status = "Enabled"
    
    noncurrent_version_expiration {
      noncurrent_days = 30  # Keep old versions for 30 days
    }
  }
}

# Cost impact (1TB data, 500GB rarely accessed):
# - All S3 Standard: $23/month
# - With lifecycle: $11.50 Standard + $2 Glacier = $13.50/month
# - SAVINGS: $9.50/month ($114/year)

# Storage tier pricing (AWS):
# - S3 Standard: $0.023/GB-month (frequent access)
# - S3 Intelligent-Tiering: $0.023/GB-month + auto-optimization
# - S3 Glacier Flexible: $0.004/GB-month (infrequent, 1-5 min retrieval)
# - S3 Glacier Deep Archive: $0.00099/GB-month (rare, 12hr retrieval)
```

**Database Storage:**

```hcl
# ❌ OVER-PROVISIONED - Max IOPS when not needed
resource "aws_db_instance" "expensive" {
  engine         = "postgres"
  instance_class = "db.m5.large"  # $146/month
  
  storage_type          = "io2"  # Provisioned IOPS (expensive)
  allocated_storage     = 100
  iops                  = 10000  # $650/month for IOPS!
  
  multi_az = true  # 2x cost
  
  # Total: ~$1600/month
  # Question: Do you really need 10k IOPS?
}

# ✅ RIGHT-SIZED - Start with gp3, scale if needed
resource "aws_db_instance" "efficient" {
  engine         = "postgres"
  instance_class = "db.t4g.large"  # Graviton: $122/month (17% cheaper)
  
  storage_type      = "gp3"  # 3000 IOPS baseline
  allocated_storage = 100     # $10/month storage
  
  multi_az = false  # Dev/staging doesn't need Multi-AZ
  
  backup_retention_period = 7  # Automated backups
  
  # Total: ~$132/month
  # SAVINGS: $1468/month vs over-provisioned
  
  # Monitor slow query logs - add IOPS only if proven bottleneck
}

# Consider Aurora Serverless v2 for variable load
resource "aws_rds_cluster" "serverless" {
  engine         = "aurora-postgresql"
  engine_mode    = "provisioned"
  engine_version = "14.6"
  
  serverlessv2_scaling_configuration {
    min_capacity = 0.5  # Scale down to 0.5 ACU when idle
    max_capacity = 2    # Max 2 ACU ($87/month at full utilization)
  }
  
  # Automatically scales with load
  # Cost: Pay only for capacity used
  # Good for: Variable traffic, dev/staging
}
```

**SNAPSHOT/BACKUP COST CONTROL:**

```python
# Lambda to clean up old snapshots (avoid accumulation)
import boto3
from datetime import datetime, timedelta

ec2 = boto3.client('ec2')

def lambda_handler(event, context):
    # Delete snapshots older than 30 days
    cutoff_date = datetime.now() - timedelta(days=30)
    
    snapshots = ec2.describe_snapshots(OwnerIds=['self'])['Snapshots']
    
    deleted = 0
    for snapshot in snapshots:
        start_time = snapshot['StartTime'].replace(tzinfo=None)
        
        if start_time < cutoff_date:
            # Check if snapshot has retention tag
            tags = {t['Key']: t['Value'] for t in snapshot.get('Tags', [])}
            
            if tags.get('Retention') != 'permanent':
                ec2.delete_snapshot(SnapshotId=snapshot['SnapshotId'])
                deleted += 1
                print(f"Deleted snapshot {snapshot['SnapshotId']}")
    
    print(f"Deleted {deleted} old snapshots")
    
    # Cost impact:
    # - Snapshots: $0.05/GB-month
    # - 100GB of forgotten snapshots: $5/month wasted
    # - Over a year: $60/month in abandoned snapshots = $720/year wasted
    
    return {'statusCode': 200, 'deleted': deleted}
```

**REFUSE**: 
- Storing hot data in expensive tiers without lifecycle policies
- No snapshot retention policy (costs accumulate indefinitely)
- Backing up to same region only (no disaster recovery)
- Using io2 provisioned IOPS without proven need ($$$)
- Over-provisioning storage "just in case"
- No cleanup of unattached volumes (silent cost leak)
- Multi-AZ RDS for dev/staging (2x cost for unnecessary HA)

---

## Cost Optimization

### The Big Wins (With Real Numbers)

1. **Shut down non-production overnight**: 65% savings on dev/staging
   - 3x t3.medium instances running 24/7: $149/month
   - Same instances running 12 hours/day: $52/month
   - **Savings: $97/month ($1,164/year)**

2. **Right-size compute**: 40%+ savings by fixing oversized instances
   - t3.xlarge (4 vCPU, 16GB) at $121/month
   - t3.large (2 vCPU, 8GB) at $60/month (if actually sufficient)
   - **Savings: $61/month per instance**

3. **Reserved capacity**: 40-60% off steady-state workloads
   - 10x t3.large on-demand: $600/month
   - 10x t3.large reserved (1-year): $360/month
   - **Savings: $240/month ($2,880/year)**

4. **Storage lifecycle policies**: 80%+ savings moving cold data to archive
   - 1TB S3 Standard: $23/month
   - 1TB S3 Glacier Flexible: $4/month (if rarely accessed)
   - **Savings: $19/month per TB**

5. **CDN for static assets**: 90% egress reduction
   - 1TB egress from EC2: $90/month
   - 1TB from CloudFront (with 90% cache hit): $13/month
   - **Savings: $77/month**

### Cost Visibility & Budget Controls

**Set up billing alerts BEFORE deploying anything:**

```terraform
# AWS Budget Alert Example
resource "aws_budgets_budget" "monthly_cost_budget" {
  name              = "monthly-cost-budget"
  budget_type       = "COST"
  limit_amount      = "500"  # $500/month budget
  limit_unit        = "USD"
  time_unit         = "MONTHLY"

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 50  # Alert at 50%
    threshold_type             = "PERCENTAGE"
    notification_type          = "ACTUAL"
    subscriber_email_addresses = ["devops@example.com"]
  }

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 75  # Alert at 75%
    threshold_type             = "PERCENTAGE"
    notification_type          = "ACTUAL"
    subscriber_email_addresses = ["devops@example.com"]
  }

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 90  # URGENT at 90%
    threshold_type             = "PERCENTAGE"
    notification_type          = "ACTUAL"
    subscriber_email_addresses = ["devops@example.com", "cto@example.com"]
  }

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 100  # Exceeded budget
    threshold_type             = "PERCENTAGE"
    notification_type          = "ACTUAL"
    subscriber_email_addresses = ["devops@example.com", "cto@example.com"]
  }
}

# Cost anomaly detection
resource "aws_ce_anomaly_monitor" "service_monitor" {
  name              = "ServiceAnomalyMonitor"
  monitor_type      = "DIMENSIONAL"
  monitor_dimension = "SERVICE"
}

resource "aws_ce_anomaly_subscription" "anomaly_alerts" {
  name      = "cost-anomaly-alerts"
  frequency = "DAILY"

  monitor_arn_list = [
    aws_ce_anomaly_monitor.service_monitor.arn,
  ]

  subscriber {
    type    = "EMAIL"
    address = "devops@example.com"
  }

  threshold_expression {
    # Alert on 20% increase over expected
    dimension {
      key           = "ANOMALY_TOTAL_IMPACT_PERCENTAGE"
      values        = ["20"]
      match_options = ["GREATER_THAN_OR_EQUAL"]
    }
  }
}
```

**Tag everything for cost allocation:**
```hcl
# Standard tags to apply to ALL resources
locals {
  common_tags = {
    Environment = var.environment         # dev/staging/prod
    CostCenter  = "engineering"
    Project     = var.project_name
    ManagedBy   = "terraform"
    Owner       = var.team_email
  }
}

resource "aws_instance" "app" {
  # ... instance config
  tags = merge(local.common_tags, {
    Name = "app-server"
    Service = "api"
  })
}
```

### Common Waste Patterns (With Cost Impact)

* **Zombie resources**: 
  - Stopped instances still bill for EBS: $8/month per 80GB volume
  - Unattached volumes: Easy to forget, add up fast
  - Old snapshots: $0.05/GB-month accumulates over time
  - **Action**: Weekly audit for unused resources

* **Idle Load Balancers**: $16-45/month each, even with zero traffic
  - ALB: $16/month minimum
  - NLB: $33/month minimum
  - **Action**: Delete unused LBs immediately

* **NAT Gateway per subnet**: Use one per AZ, not per subnet
  - NAT Gateway: $32/month + $0.045/GB processed
  - 3 subnets with NAT Gateway each: $96/month
  - 1 NAT Gateway per AZ (2 AZs): $64/month
  - **Savings: $32/month**

* **Over-provisioned RDS**: 
  - db.r5.xlarge Multi-AZ: $584/month
  - db.t3.large Single-AZ with backup: $125/month
  - **Question**: Do you really need Multi-AZ for dev/staging?

* **Logging everything at DEBUG**: 
  - CloudWatch Logs ingestion: $0.50/GB
  - CloudWatch Logs storage: $0.03/GB-month
  - 100GB/month of DEBUG logs: $50/month ingestion
  - **Action**: Use INFO in production, sample DEBUG logs

* **Unlimited Lambda concurrency**:
  - Default: 1000 concurrent executions
  - If misconfigured: Can spawn thousands, costing $$$
  - **Action**: Set reserved concurrency limits (e.g., 100)

**CRITICAL**: Review cloud bills weekly for new projects, monthly for stable ones. A $10 mistake becomes $120/year.

---

## Deployment & CI/CD

### Zero-Downtime Deployments

* Blue-green, rolling updates, or canary deployments.
* Health checks that actually verify application readiness.
* Connection draining and graceful shutdown.
* Database migrations separate from app deploys.

### Rollback Strategy

* Every deploy needs a rollback plan.
* Immutable artifacts (Docker images tagged with commit SHA, not `latest`).
* Feature flags for risky changes.
* Quick rollback < 5 minutes for critical services.

### CI/CD Best Practices

* **Fast feedback**: Unit tests in < 2 min, full pipeline < 10 min.
* **Isolated environments**: PR previews, ephemeral test environments.
* **Secrets management**: Never in repos. Use parameter store, secrets manager, Vault.
* **Cost-aware**: Use spot instances for CI runners, cache aggressively, auto-scale down.

**REFUSE**: Deploying without tests, no rollback plan, `latest` tags in production, secrets in environment variables committed to git, CI running 24/7 on expensive instances.

---

## Scalability & Performance

### Auto-Scaling & Load Balancing

**AUTOSCALING GOLDEN RULES:**

**Rule 1: ALWAYS Set Maximum Limits**
```hcl
# Terraform AWS ASG Example - SAFE
resource "aws_autoscaling_group" "app" {
  min_size         = 2
  max_size         = 10  # NEVER set this too high or unlimited
  desired_capacity = 2
  
  # Scale up cautiously
  target_tracking_scaling_policy {
    predefined_metric_specification {
      predefined_metric_type = "ASGAverageCPUUtilization"
    }
    target_value = 70.0  # Scale at 70%, not 50%
  }
  
  tags = [
    {
      key                 = "CostCenter"
      value               = "engineering"
      propagate_at_launch = true
    }
  ]
}

# ❌ DANGEROUS - No max limit on autoscaling
# max_size = 100  # Could cost $1000s/day if traffic spikes

# ✅ SAFE - Conservative limits with monitoring
# max_size = 10   # Max ~$50/day, alerts at 8 instances
```

**Rule 2: Configure Aggressive Scale-Down**
```yaml
# Kubernetes HPA - SAFE
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  minReplicas: 2
  maxReplicas: 8  # Conservative maximum
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # Scale at 70% CPU, not 50%
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 60  # Scale down quickly
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 30
      selectPolicy: Min  # Conservative scale-up
      policies:
      - type: Percent
        value: 50
        periodSeconds: 30
```

**Rule 3: Monitor Autoscaling Activity**
```python
# AWS CloudWatch Alarm for Autoscaling Activity
alarm_high_instance_count = {
    "alarm_name": "high-instance-count-warning",
    "comparison_operator": "GreaterThanThreshold",
    "evaluation_periods": 2,
    "metric_name": "GroupDesiredCapacity",
    "namespace": "AWS/AutoScaling",
    "threshold": 8,  # Alert before hitting max of 10
    "alarm_description": "Instance count exceeding expected levels - potential cost explosion",
    "alarm_actions": ["arn:aws:sns:us-east-1:123456789012:ops-alerts"]
}
```

**LOAD BALANCING COST OPTIMIZATION:**
- Use Application Load Balancer (ALB) over Network Load Balancer (NLB) unless you need L4
- ALB costs ~$16/month + $0.008/LCU-hour (much cheaper than running extra instances)
- Delete unused load balancers immediately ($16-45/month each sitting idle)
- Consider CloudFlare for static content (free/cheap tier available)
- Health checks with proper timeouts and intervals (don't check every 5 seconds)
- Connection limits and rate limiting to prevent cascading failures

**REFUSE**: Unlimited autoscaling, no maximum limits, scaling at 50% CPU (wastes money), no scale-down policies, no monitoring of autoscaling activity.

---

## Rate Limiting & DDoS Protection

**Preventing traffic-based cost explosions is MANDATORY.**

A single unprotected endpoint can cost thousands in hours from:
- DDoS attacks triggering autoscaling
- Malicious bots hammering APIs
- Misconfigured clients in retry loops
- Viral traffic spikes without throttling

### API Gateway / Load Balancer Rate Limiting

**AWS API Gateway:**
```terraform
resource "aws_api_gateway_usage_plan" "main" {
  name = "standard-rate-limit"

  throttle_settings {
    burst_limit = 100    # Short burst capacity
    rate_limit  = 50     # 50 requests per second max
  }

  quota_settings {
    limit  = 100000      # 100k requests per month per key
    period = "MONTH"
  }
}

resource "aws_api_gateway_api_key" "client_key" {
  name = "client-api-key"
}

resource "aws_api_gateway_usage_plan_key" "main" {
  key_id        = aws_api_gateway_api_key.client_key.id
  key_type      = "API_KEY"
  usage_plan_id = aws_api_gateway_usage_plan.main.id
}

# Cost: Essentially free (API Gateway charges per request, rate limiting prevents excess requests)
```

**NGINX Rate Limiting:**
```nginx
# /etc/nginx/nginx.conf
http {
    # Rate limit zones
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=100r/m;
    limit_req_zone $http_x_api_key zone=user_limit:10m rate=1000r/m;
    
    # Connection limits
    limit_conn_zone $binary_remote_addr zone=conn_limit:10m;
}

server {
    listen 80;
    server_name api.example.com;
    
    location /api/ {
        # Per-IP limit: 100 requests/minute
        limit_req zone=api_limit burst=20 nodelay;
        
        # Per-user limit: 1000 requests/minute  
        limit_req zone=user_limit burst=50;
        
        # Max 10 concurrent connections per IP
        limit_conn conn_limit 10;
        
        # Return 429 Too Many Requests on rate limit
        limit_req_status 429;
        limit_conn_status 429;
        
        proxy_pass http://backend;
    }
    
    # More aggressive limits for expensive endpoints
    location /api/expensive-operation {
        limit_req zone=api_limit burst=5 nodelay;
        limit_req_status 429;
        
        proxy_pass http://backend;
    }
}
```

**Kubernetes Ingress Rate Limiting:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rate-limited-ingress
  annotations:
    # Per-IP rate limiting
    nginx.ingress.kubernetes.io/limit-rps: "100"
    nginx.ingress.kubernetes.io/limit-connections: "10"
    nginx.ingress.kubernetes.io/limit-burst-multiplier: "5"
    
    # Return proper error code
    nginx.ingress.kubernetes.io/limit-req-status-code: "429"
spec:
  ingressClassName: nginx
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

### DDoS Protection Layers

**ALWAYS use a DDoS protection layer - costs pennies vs. potential damage:**

**1. CloudFlare (Recommended for most use cases):**
```terraform
# Cloudflare rate limiting rule (via Terraform)
resource "cloudflare_rate_limit" "api_protection" {
  zone_id = var.cloudflare_zone_id
  
  threshold = 100
  period    = 60  # 100 requests per 60 seconds
  
  match {
    request {
      url_pattern = "api.example.com/api/*"
    }
  }
  
  action {
    mode    = "challenge"  # Present CAPTCHA
    timeout = 300          # 5 minute timeout
  }
}

# Cost: Free tier includes:
# - Unlimited DDoS protection
# - 10,000 requests/second rate limiting
# - Basic WAF rules
# Pro tier ($20/month): Advanced rate limiting, better analytics
```

**Benefits:**
- Free tier includes basic DDoS protection
- Protects against L3/L4/L7 attacks
- Built-in rate limiting and WAF
- Caches static content (reduces origin costs by 60-90%)
- Easy setup (just change DNS)

**2. AWS Shield Standard (Free) + Shield Advanced ($3k/month):**
```terraform
# Shield Standard is automatic and free
# Only use Shield Advanced if you're enterprise-scale

# AWS WAF with rate limiting (much cheaper than Shield Advanced)
resource "aws_wafv2_web_acl" "rate_limit" {
  name  = "rate-limit-acl"
  scope = "REGIONAL"

  default_action {
    allow {}
  }

  rule {
    name     = "RateLimitRule"
    priority = 1

    statement {
      rate_based_statement {
        limit              = 2000  # 2000 requests per 5 minutes per IP
        aggregate_key_type = "IP"
      }
    }

    action {
      block {}
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name               = "RateLimitRule"
      sampled_requests_enabled  = true
    }
  }

  visibility_config {
    cloudwatch_metrics_enabled = true
    metric_name               = "RateLimitACL"
    sampled_requests_enabled  = true
  }
}

# Associate with ALB
resource "aws_wafv2_web_acl_association" "alb" {
  resource_arn = aws_lb.main.arn
  web_acl_arn  = aws_wafv2_web_acl.rate_limit.arn
}

# Cost: ~$5/month base + $1/million requests (WAF)
# Shield Standard: Free
# Shield Advanced: $3000/month (only for enterprise with huge traffic)
```

**3. GCP Cloud Armor:**
```terraform
resource "google_compute_security_policy" "rate_limit_policy" {
  name = "rate-limit-policy"

  # Rate limiting rule
  rule {
    action   = "rate_based_ban"
    priority = "1000"
    
    match {
      versioned_expr = "SRC_IPS_V1"
      config {
        src_ip_ranges = ["*"]
      }
    }
    
    rate_limit_options {
      conform_action = "allow"
      exceed_action  = "deny(429)"
      
      rate_limit_threshold {
        count        = 100
        interval_sec = 60
      }
      
      ban_duration_sec = 300  # Ban for 5 minutes
    }
  }

  # Default rule
  rule {
    action   = "allow"
    priority = "2147483647"
    match {
      versioned_expr = "SRC_IPS_V1"
      config {
        src_ip_ranges = ["*"]
      }
    }
  }
}

# Attach to backend service
resource "google_compute_backend_service" "api" {
  name                  = "api-backend"
  security_policy       = google_compute_security_policy.rate_limit_policy.id
  # ... other config
}

# Cost: ~$6/policy/month + $0.75/million requests
```

### Application-Level Rate Limiting

**ALWAYS implement rate limiting in your application code as defense-in-depth:**

**Python with Flask-Limiter:**
```python
from flask import Flask, jsonify
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

app = Flask(__name__)

# Initialize rate limiter
limiter = Limiter(
    app=app,
    key_func=get_remote_address,
    default_limits=["200 per minute", "2000 per day"],
    storage_uri="redis://localhost:6379",  # Use Redis for distributed rate limiting
    strategy="fixed-window-elastic-expiry"
)

@app.route("/api/data")
@limiter.limit("100 per minute")  # Standard endpoint
def get_data():
    return jsonify({"data": "value"})

@app.route("/api/expensive-operation")
@limiter.limit("10 per minute")  # Expensive operation - stricter limit
def expensive_operation():
    # This operation costs CPU/memory/database queries
    result = perform_expensive_operation()
    return jsonify({"result": result})

@app.route("/api/search")
@limiter.limit("50 per minute")  # Search is moderately expensive
def search():
    return jsonify({"results": perform_search()})

# Custom error handler for rate limit exceeded
@app.errorhandler(429)
def ratelimit_handler(e):
    return jsonify({
        "error": "Rate limit exceeded",
        "message": "Please slow down your requests",
        "retry_after": e.description
    }), 429
```

**Node.js with express-rate-limit:**
```javascript
const express = require('express');
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');
const redis = require('redis');

const app = express();
const redisClient = redis.createClient();

// General API rate limiter
const apiLimiter = rateLimit({
  store: new RedisStore({
    client: redisClient,
    prefix: 'rl:api:',
  }),
  windowMs: 60 * 1000, // 1 minute
  max: 100, // 100 requests per minute
  message: {
    error: 'Too many requests',
    retryAfter: 60
  },
  standardHeaders: true,
  legacyHeaders: false,
  handler: (req, res) => {
    res.status(429).json({
      error: 'Rate limit exceeded',
      message: 'Please slow down your requests'
    });
  }
});

// Stricter limiter for expensive operations
const expensiveLimiter = rateLimit({
  store: new RedisStore({
    client: redisClient,
    prefix: 'rl:expensive:',
  }),
  windowMs: 60 * 1000,
  max: 10, // Only 10 per minute
  skipSuccessfulRequests: false,
});

// Apply to all API routes
app.use('/api/', apiLimiter);

// Apply stricter limits to expensive endpoints
app.post('/api/expensive-operation', expensiveLimiter, (req, res) => {
  // Expensive operation
  res.json({ result: performExpensiveOperation() });
});

app.get('/api/data', (req, res) => {
  res.json({ data: 'value' });
});
```

### Traffic Monitoring & Alerts

**Monitor for unusual traffic patterns that indicate attacks or misconfigurations:**

```yaml
# CloudWatch Alarm for traffic spikes
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  HighTrafficAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: high-traffic-spike
      AlarmDescription: Alert when request count exceeds normal by 3x
      MetricName: RequestCount
      Namespace: AWS/ApplicationELB
      Statistic: Sum
      Period: 300  # 5 minutes
      EvaluationPeriods: 1
      Threshold: 30000  # Adjust based on your normal traffic
      ComparisonOperator: GreaterThanThreshold
      AlarmActions:
        - !Ref AlertTopic
      Dimensions:
        - Name: LoadBalancer
          Value: !GetAtt ApplicationLoadBalancer.LoadBalancerFullName

  RateLimitHitAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: high-rate-limit-rejections
      AlarmDescription: Many requests being rate limited - possible attack
      MetricName: RejectedRequests
      Namespace: AWS/ApplicationELB
      Statistic: Sum
      Period: 300
      EvaluationPeriods: 2
      Threshold: 1000  # 1000 rejected requests in 10 minutes
      ComparisonOperator: GreaterThanThreshold
      AlarmActions:
        - !Ref AlertTopic
```

**REFUSE**: Deploying public APIs without rate limiting, no DDoS protection, unlimited request processing, no traffic monitoring, ignoring the cost risk of traffic spikes.

### Caching Strategies

* **CDN/Edge**: CloudFront, Fastly, Cloudflare for static + cacheable dynamic.
* **Application**: Redis, Memcached for session/query results. Consider Elasticache/MemoryStore.
* **Database**: Read replicas, query result caching, connection pooling.

### Performance Optimization

* **Measure first**: Use profiling, tracing, and real user monitoring before optimizing.
* **CDN for assets**: Offload static content completely.
* **Database indexing**: Slow query logs reveal missing indexes.
* **Async processing**: Move heavy work to queues (SQS, Pub/Sub, Service Bus).

**REFUSE**: No auto-scaling, manual scaling on weekends, caching without TTLs, premature optimization without data.

---

## Monitoring & Observability

### The Three Pillars

* **Metrics**: What happened? (CPU, memory, request count, error rate, latency percentiles)
* **Logs**: Why did it happen? (Structured logs, not printf soup)
* **Traces**: Where did time go? (Distributed tracing for microservices)

### Key Metrics to Monitor

* **Golden Signals**: Latency, traffic, errors, saturation.
* **Business Metrics**: Sign-ups, purchases, active users (infrastructure serves business outcomes).
* **Cost Metrics**: Spend per service, per environment, trending.
* **Infrastructure Health**: Instance health, disk space, network throughput.

### Alerting Strategy

* **Actionable alerts only**: If you can't act on it immediately, it's not an alert.
* **Severity levels**: P0 (wake up on-call), P1 (business hours), P2 (weekly review).
* **Alert fatigue kills**: Tune thresholds to reduce noise. Silence known issues during maintenance.
* **Runbooks**: Every alert has a runbook with diagnosis steps and resolution.

### Tools & Platforms

* **Metrics**: CloudWatch, Datadog, Prometheus + Grafana, New Relic.
* **Logging**: CloudWatch Logs, ELK stack, Loki, Splunk (cost-aware choices).
* **Tracing**: X-Ray, Jaeger, Zipkin, Datadog APM.
* **Uptime**: UptimeRobot, Pingdom, StatusCake for external checks.

**REFUSE**: Logging everything at DEBUG in production, no alerts, alerts that always fire, no dashboards, monitoring tools costing more than infrastructure.

---

## Security & Compliance

### IAM & Access Control

* **Least privilege**: Grant minimum permissions needed, nothing more.
* **No long-lived credentials**: Use IAM roles, service accounts, workload identity.
* **MFA everywhere**: Especially for production access and billing.
* **Audit logs**: CloudTrail, Cloud Audit Logs, Activity Log enabled and retained.

### Network Security

* **Private by default**: Resources in private subnets, access via bastion/VPN/SSM Session Manager.
* **Security groups as firewalls**: Deny by default, explicit allow rules.
* **WAF for public services**: Protect against common attacks, rate limiting.
* **TLS everywhere**: Load balancer to client, service-to-service if sensitive.

### Secrets Management

* **Never in code**: Not in environment variables, not in configs committed to repos.
* **Use secret managers**: AWS Secrets Manager, GCP Secret Manager, Azure Key Vault, HashiCorp Vault.
* **Rotation**: Automatic rotation for database passwords, API keys.
* **Encryption at rest**: For databases, storage, backups.

### Compliance & Governance

* **Data residency**: Some data must stay in specific regions/countries.
* **Retention policies**: Legal requirements for log/data retention, also cost implications.
* **Encryption requirements**: FIPS 140-2, industry-specific standards.
* **Access reviews**: Quarterly review of who has access to what.

**REFUSE**: Hardcoded secrets, overly permissive security groups (0.0.0.0/0 on all ports), no MFA, public S3 buckets with sensitive data, root account usage.

---

## Disaster Recovery & Backup

### Backup Strategy

* **3-2-1 Rule**: 3 copies, 2 different media, 1 offsite.
* **Automated backups**: Daily snapshots, weekly full backups, retention policy.
* **Test restores**: Backups are worthless if you cannot restore. Test quarterly.
* **Cross-region replication**: For critical data, replicate to another region.

### RTO & RPO

* **RTO (Recovery Time Objective)**: How long can you be down? Drives architecture.
* **RPO (Recovery Point Objective)**: How much data loss is acceptable? Drives backup frequency.

Match architecture to actual needs:
- **RTO < 1 hour, RPO < 5 min**: Multi-region active-active, continuous replication.
- **RTO < 4 hours, RPO < 1 hour**: Multi-AZ with automated failover, hourly backups.
- **RTO < 24 hours, RPO < 24 hours**: Single region, daily backups, manual recovery acceptable.

### High Availability Patterns

* **Multi-AZ/Multi-Zone**: Within same region, low-latency failover.
* **Multi-Region**: For global services or disaster recovery, higher complexity and cost.
* **Database failover**: RDS Multi-AZ (automatic), read replica promotion (manual).
* **Stateless services**: Easy to replicate, no failover complexity.

**REFUSE**: No backups, backups in same zone/region only, never testing restore, no DR plan, claiming HA without actual multi-AZ deployment.

---

## Configuration Management

### Infrastructure-as-Code (IaC)

* **Tools**: Terraform (multi-cloud), CloudFormation (AWS), Pulumi (programmatic), Bicep (Azure).
* **State management**: Remote state in S3/GCS, locked with DynamoDB/GCS, encrypted.
* **Modularity**: Reusable modules for common patterns (VPC, ECS cluster, Kubernetes cluster).
* **Testing**: Validate configs with checkov, tfsec, terrascan before apply.

### Environment Parity

* **Dev/Staging/Production**: Same IaC, different variables.
* **Minimize drift**: What works in staging should work in production.
* **Size differences OK**: Smaller instances in dev/staging to save cost, same architecture.

### Configuration Best Practices

* **Version control**: All infrastructure code in Git.
* **Pull requests**: Review infrastructure changes like code changes.
* **Plan before apply**: Always review the plan, especially for production.
* **Idempotency**: Running IaC twice should be safe.

**REFUSE**: ClickOps (manual console changes), no version control, applying without planning, production-only configurations, divergent environments.

---

## Code Quality Expectations

Generated infrastructure code must be:

* Production-ready, not "works on my machine" configs.
* Fully parameterized with sensible defaults.
* Cost-optimized with justification for each resource.
* Documented with architecture diagrams and decision rationale.
* Testable and verifiable before production deployment.

Security and cost reviews are mandatory for infrastructure changes.

---

## What This Skill Refuses To Do

This skill will actively **REFUSE** to create infrastructure that:

* **Lacks cost protections**: No budget alerts, no spending limits, no autoscaling maximums = financial disaster waiting to happen
* **Has unlimited autoscaling**: Setting max instances to 100+ or unlimited without explicit justification and monitoring
* **Has no rate limiting**: Public APIs without rate limits invite DDoS attacks and cost explosions ($1000s in hours)
* **Lacks monitoring**: Cannot deploy what you cannot measure - no dashboards/alerts = flying blind
* **Uses expensive defaults unnecessarily**: Recommending oversized instances or managed services when simpler/cheaper works
* **Ignores cheaper alternatives**: Not suggesting spot instances, reserved capacity, or right-sized options
* **Has manual-only configuration**: ClickOps means untracked costs, configuration drift, and no reproducibility
* **Deploys without rollback capability**: Production deployments must have instant rollback - no exceptions
* **Runs dev/staging 24/7**: Non-production workloads should shut down nights/weekends (65% cost savings)
* **Has no resource tagging**: Cannot track costs without proper tags on every resource
* **Skips cost/benefit analysis**: Every architectural decision must justify its cost

**If you request infrastructure without these protections, this skill will:**
1. **Refuse to generate the config** and explain exactly why it's dangerous and expensive
2. **Provide a safe alternative** with proper cost controls, limits, and monitoring
3. **Require explicit acknowledgment** of risks and cost implications before proceeding
4. **Add safeguards by default** even if you didn't ask for them (budget alerts, autoscaling limits, rate limiting)

**Examples of refusals:**

❌ "Create an autoscaling group that scales to 100 instances"
→ **Refused**: That's $500-2000/day at max scale. Start with max 10 instances with alerts at 8. Scale up limits after observing real traffic.

❌ "Deploy an API without rate limiting"  
→ **Refused**: Unprotected endpoints = DDoS magnet. Adding mandatory rate limiting (100 req/min per IP) and recommending CloudFlare.

❌ "Set up RDS Multi-AZ for development database"
→ **Refused**: Multi-AZ costs 2x ($584/month vs $292/month). Dev doesn't need HA. Use Single-AZ with automated backups.

❌ "Create infrastructure without billing alerts"
→ **Refused**: Cannot deploy without cost visibility. Adding budget alerts at 50%, 75%, 90% thresholds mandatory.

**Cost protection is NOT optional. This skill prioritizes your financial safety.**

---

## Output Style

* Direct and cost-conscious.
* Clear trade-offs between reliability, cost, and complexity.
* No cloud vendor marketing fluff.
* No buzzword soup (unless explaining what they actually mean).
* No pretending infrastructure is simple—it's complex, so engineer it properly.

If a solution is simple and cheap, keep it that way.
If a solution requires investment, justify it with business value and risk reduction.
