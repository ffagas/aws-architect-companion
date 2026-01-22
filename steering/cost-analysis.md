# Cost Analysis - Steering File

Estimate infrastructure costs with interactive assumption collection.

## Prerequisites

**1. Architecture must be analyzed:** Services must be identified (from design or code)

**If not done yet:**
- Check for design.md or infrastructure code
- If found, inform user: "To estimate costs, I need to identify your services first. Let me analyze your [design/code]..."
- Run `diagram-from-design.md` or `diagram-from-code.md` workflow first
- Then proceed with cost analysis

**2. AWS credentials configured:** Required for aws-pricing MCP server

**3. Region specified:** Default to us-east-1 if not specified

**Important:** Handle prerequisites automatically. Don't ask user to run other workflows manually.

## Step 1: Verify AWS Credentials

Check if AWS credentials are configured:
- Look for AWS_PROFILE or AWS_ACCESS_KEY_ID environment variables
- If not configured, inform user that Cost Analysis requires AWS credentials
- Provide instructions: `aws configure` or set environment variables

**If credentials not available:**
- Explain that pricing data requires AWS API access
- Offer to continue with other capabilities
- Document this limitation in the report

## Step 2: Identify Services for Pricing

From the architecture analysis, extract:

**Compute services:**
- Lambda (functions count, estimated invocations)
- EC2 (instance types, count)
- ECS/Fargate (tasks, vCPU, memory)

**Storage services:**
- S3 (buckets, estimated storage GB)
- EBS (volumes, size, type)

**Database services:**
- DynamoDB (tables, read/write capacity or on-demand)
- RDS (instance type, storage, multi-AZ)
- Aurora (instance type, storage)

**Networking services:**
- API Gateway (requests)
- CloudFront (requests, data transfer)
- ALB/NLB (hours, LCUs)
- VPC (NAT Gateway hours)

**Integration services:**
- SQS (requests)
- SNS (requests, notifications)
- EventBridge (events)
- Step Functions (state transitions)

**Other services:**
- CloudWatch (logs GB, metrics, alarms)
- Cognito (MAUs)
- Secrets Manager (secrets count)

## Step 3: Collect Usage Assumptions

For each service, ask user for usage assumptions. If user doesn't know, provide reasonable defaults.

### 3.1: General Application Assumptions

Ask user:
```
To estimate costs accurately, I need some information about your application:

1. Expected number of users per month: [___]
   (Default: 1,000 users)

2. Expected requests per second: [___]
   (Default: 10 req/s = ~2.6M requests/month)

3. Operating hours: [___]
   - 24/7 (730 hours/month)
   - Business hours only (8x5 = 160 hours/month)
   (Default: 24/7)

4. Environment: [___]
   - Production
   - Development/Test
   (Default: Production)
```

### 3.2: Service-Specific Assumptions

**For Lambda:**
```
Lambda Function: [function-name]
- Estimated invocations per month: [___]
  (Default: Based on requests/sec × 2.6M)
- Average duration: [___] ms
  (Default: 200ms)
- Memory allocation: [___] MB
  (Default: From code or 512MB)
```

**For DynamoDB:**
```
DynamoDB Table: [table-name]
- Billing mode: [Provisioned/On-Demand]
  (Default: From code or On-Demand)
- Estimated reads per month: [___]
  (Default: requests × 2)
- Estimated writes per month: [___]
  (Default: requests × 0.5)
- Average item size: [___] KB
  (Default: 1KB)
- Storage: [___] GB
  (Default: 10GB)
```

**For S3:**
```
S3 Bucket: [bucket-name]
- Storage: [___] GB
  (Default: 100GB)
- Storage class: [Standard/IA/Glacier]
  (Default: Standard)
- GET requests per month: [___]
  (Default: requests × 1)
- PUT requests per month: [___]
  (Default: requests × 0.1)
```

**For RDS:**
```
RDS Instance: [instance-id]
- Instance type: [___]
  (Default: From code or db.t3.micro)
- Storage: [___] GB
  (Default: 20GB)
- Multi-AZ: [Yes/No]
  (Default: From code or No)
- Backup storage: [___] GB
  (Default: Storage × 1)
```

**For API Gateway:**
```
API Gateway: [api-name]
- Requests per month: [___]
  (Default: requests × 2.6M)
- Average request size: [___] KB
  (Default: 4KB)
- Average response size: [___] KB
  (Default: 4KB)
```

### 3.3: Document All Assumptions

Create a list of all assumptions used:
```
Assumptions Used:
1. Users per month: 1,000 (default)
2. Requests per second: 10 (default)
3. Operating hours: 24/7 (default)
4. Lambda invocations: 2.6M/month (calculated from requests)
5. Lambda duration: 200ms (default)
6. Lambda memory: 512MB (from code)
7. DynamoDB billing: On-Demand (default)
8. S3 storage: 100GB (default)
... etc
```

## Step 4: Query Pricing Data

Use `aws-pricing` MCP server to get pricing information.

### 4.1: Get Service Codes

```
Use get_pricing_service_codes() to find service codes:
- AWSLambda
- AmazonDynamoDB
- AmazonS3
- AmazonRDS
- AmazonAPIGateway
- etc.
```

### 4.2: Get Pricing for Each Service

**For Lambda:**
```
Use get_pricing(
  service_code="AWSLambda",
  region="us-east-1",
  filters=[
    {"Field": "group", "Value": "AWS-Lambda-Requests"},
    {"Field": "group", "Value": "AWS-Lambda-Duration"}
  ]
)
```

**For DynamoDB:**
```
Use get_pricing(
  service_code="AmazonDynamoDB",
  region="us-east-1",
  filters=[
    {"Field": "group", "Value": "DDB-ReadUnits"},
    {"Field": "group", "Value": "DDB-WriteUnits"},
    {"Field": "group", "Value": "DDB-Storage"}
  ]
)
```

**For S3:**
```
Use get_pricing(
  service_code="AmazonS3",
  region="us-east-1",
  filters=[
    {"Field": "storageClass", "Value": "General Purpose"},
    {"Field": "volumeType", "Value": "Standard"}
  ]
)
```

### 4.3: Alternative - Use generate_cost_report

The `aws-pricing` MCP server has a `generate_cost_report` function that can create a complete cost report:

```
Use generate_cost_report(
  pricing_data={...},
  service_name="[Project Name]",
  assumptions=[list of assumptions],
  detailed_cost_data={
    "services": {
      "Lambda": {
        "usage": "2.6M invocations, 200ms avg, 512MB",
        "estimated_cost": "$X.XX",
        "unit_pricing": {...},
        "calculation_details": "..."
      },
      ...
    }
  }
)
```

## Step 5: Calculate Costs

For each service, calculate monthly cost:

### Lambda Cost Calculation
```
Requests cost = (invocations / 1M) × $0.20
Duration cost = (invocations × duration_ms × memory_MB / 1024) × $0.0000166667

Total Lambda cost = Requests cost + Duration cost
```

### DynamoDB Cost Calculation (On-Demand)
```
Read cost = (reads / 1M) × $0.25
Write cost = (writes / 1M) × $1.25
Storage cost = storage_GB × $0.25

Total DynamoDB cost = Read + Write + Storage
```

### S3 Cost Calculation
```
Storage cost = storage_GB × $0.023 (Standard)
GET requests cost = (GET_requests / 1000) × $0.0004
PUT requests cost = (PUT_requests / 1000) × $0.005

Total S3 cost = Storage + GET + PUT
```

### API Gateway Cost Calculation
```
Requests cost = (requests / 1M) × $3.50
Data transfer cost = (data_GB) × $0.09

Total API Gateway cost = Requests + Data transfer
```

## Step 6: Generate Cost Report

Create `architecture-docs/CostAnalysis.md`:

**Filename strategy:**
- First analysis: `CostAnalysis.md`
- Additional scenarios: `CostAnalysis-[scenario].md`
  - Example: `CostAnalysis-20k-users.md`
  - Example: `CostAnalysis-2k-tps.md`
  - Example: `CostAnalysis-200-images-day.md`

**Important:** Users may request multiple cost analyses with different assumptions. Each should be saved with a descriptive filename based on the scenario.

```markdown
# AWS Cost Analysis

**Project:** [Project Name]
**Date:** [Date]
**Region:** [Region]
**Analysis Type:** [Design-based / Code-based]

## Executive Summary

**Estimated Monthly Cost:** $XXX.XX
**Estimated Annual Cost:** $X,XXX.XX

**Cost Breakdown:**
- Compute: $XX.XX (XX%)
- Storage: $XX.XX (XX%)
- Database: $XX.XX (XX%)
- Networking: $XX.XX (XX%)
- Other: $XX.XX (XX%)

## Cost Breakdown by Service

| Service | Monthly Cost | Annual Cost | % of Total |
|---------|--------------|-------------|------------|
| Lambda | $XX.XX | $XXX.XX | XX% |
| DynamoDB | $XX.XX | $XXX.XX | XX% |
| S3 | $XX.XX | $XXX.XX | XX% |
| API Gateway | $XX.XX | $XXX.XX | XX% |
| ... | ... | ... | ... |
| **Total** | **$XXX.XX** | **$X,XXX.XX** | **100%** |

## Detailed Cost Analysis

### Lambda Functions

**Usage:**
- Invocations: 2,600,000 per month
- Average duration: 200ms
- Memory allocation: 512MB

**Unit Pricing:**
- Requests: $0.20 per 1M requests
- Duration: $0.0000166667 per GB-second

**Calculation:**
- Requests: (2.6M / 1M) × $0.20 = $0.52
- Duration: (2.6M × 0.2s × 0.5GB) × $0.0000166667 = $4.33
- **Total: $4.85/month**

---

[Repeat for each service]

---

## Assumptions

The following assumptions were used for this cost analysis:

### Application Assumptions
1. **Users:** 1,000 concurrent users per month (default)
2. **Requests:** 10 requests/second = 2.6M requests/month (default)
3. **Operating Hours:** 24/7 (730 hours/month) (default)
4. **Environment:** Production (default)

### Service-Specific Assumptions
5. **Lambda invocations:** 2.6M/month (calculated from requests)
6. **Lambda duration:** 200ms average (default)
7. **Lambda memory:** 512MB (from code)
8. **DynamoDB reads:** 5.2M/month (requests × 2, default)
9. **DynamoDB writes:** 1.3M/month (requests × 0.5, default)
10. **S3 storage:** 100GB (default)
11. **S3 GET requests:** 2.6M/month (requests × 1, default)

**Note:** Costs are estimates based on these assumptions. Actual costs may vary based on real usage patterns.

## Cost Optimization Recommendations

### High Impact (Potential savings: $XX/month)
1. **Use Lambda Provisioned Concurrency selectively**
   - Current: All functions use on-demand
   - Recommendation: Use provisioned only for critical functions
   - Estimated savings: $XX/month

2. **Implement DynamoDB Reserved Capacity**
   - Current: On-Demand billing
   - Recommendation: Switch to provisioned for predictable workloads
   - Estimated savings: $XX/month (up to 50%)

### Medium Impact (Potential savings: $XX/month)
3. **Enable S3 Intelligent-Tiering**
   - Current: All data in Standard storage
   - Recommendation: Use Intelligent-Tiering for unknown access patterns
   - Estimated savings: $XX/month (up to 40%)

4. **Right-size Lambda memory**
   - Current: 512MB for all functions
   - Recommendation: Use Lambda Power Tuning to optimize
   - Estimated savings: $XX/month

### Low Impact (Potential savings: $XX/month)
5. **Implement CloudFront caching**
   - Reduces API Gateway requests
   - Estimated savings: $XX/month

## Alternative Architectures

### Option 1: Serverless with Caching
- Add ElastiCache for DynamoDB
- Estimated cost: $XXX/month (+ $XX for cache)
- Performance improvement: 50% faster reads
- Net cost change: +$XX/month

### Option 2: Reserved Capacity
- Use Reserved Instances for RDS
- Use DynamoDB Reserved Capacity
- Estimated cost: $XXX/month
- Savings: $XX/month (XX% reduction)

## Next Steps

1. **Validate assumptions** with actual usage data
2. **Implement high-impact optimizations** first
3. **Set up AWS Cost Explorer** for ongoing monitoring
4. **Configure AWS Budgets** with alerts
5. **Re-run analysis** after 1 month of production usage

## Additional Resources

- AWS Pricing Calculator: https://calculator.aws
- AWS Cost Explorer: https://aws.amazon.com/aws-cost-management/aws-cost-explorer/
- AWS Cost Optimization: https://aws.amazon.com/pricing/cost-optimization/
```

## Step 7: Report to User

Present summary:

```
Cost Analysis Complete!

💰 Estimated Monthly Cost: $XXX.XX
📅 Estimated Annual Cost: $X,XXX.XX

📊 Top Cost Drivers:
1. [Service]: $XX.XX (XX%)
2. [Service]: $XX.XX (XX%)
3. [Service]: $XX.XX (XX%)

💡 Top 3 Optimization Opportunities:
1. [Recommendation] - Save $XX/month
2. [Recommendation] - Save $XX/month
3. [Recommendation] - Save $XX/month

📄 Full report: architecture-docs/CostAnalysis.md

**CRITICAL:** Only generate `CostAnalysis.md` (or `CostAnalysis-[scenario].md` for additional scenarios). Do NOT create:
- README.md
- INDICE.md
- RESUMEN_EJECUTIVO.md (this is generated separately in complete report)
- Any other documentation files

⚠️  Note: Costs are estimates based on assumptions. Validate with AWS Pricing Calculator for production planning.

Would you like me to:
1. Explain any cost calculation
2. Explore alternative architectures
3. Generate Well-Architected Framework Review (WAFR)
4. Adjust assumptions and recalculate
```

## Error Handling

**If AWS credentials not configured:**
- Inform user that Cost Analysis requires AWS credentials
- Provide setup instructions
- Offer to continue with other capabilities

**If pricing API fails:**
- Use cached pricing data if available
- Inform user that prices may not be current
- Suggest manual validation with AWS Pricing Calculator

**If service not found in pricing API:**
- Use approximate pricing from similar services
- Document which services used approximations
- Suggest manual lookup

**If assumptions unclear:**
- Ask user for clarification
- Provide examples of typical values
- Use conservative defaults

## Tips for Accurate Cost Estimates

- **Be conservative:** Overestimate usage rather than underestimate
- **Include all services:** Don't forget CloudWatch, data transfer, etc.
- **Consider growth:** Factor in 20-30% growth over first year
- **Validate assumptions:** Use actual data when available
- **Include free tier:** Subtract free tier allowances for new accounts
- **Consider Reserved Instances:** For predictable workloads
- **Add buffer:** Add 10-15% buffer for unexpected costs
- **Document everything:** Clear assumptions enable better validation
