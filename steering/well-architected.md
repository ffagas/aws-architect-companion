# Well-Architected Framework Review (WAFR) - Steering File

Evaluate architecture against AWS Well-Architected Framework's 6 pillars.

**Output:** `architecture-docs/WellArchitectedFrameworkReview.md`

## Prerequisites

**Architecture must be analyzed first** (from design or code):
- Services identified
- Relationships mapped
- Configurations extracted

**If not done yet:**
- If user has design.md → Run `diagram-from-design.md` workflow first
- If user has infrastructure code → Run `diagram-from-code.md` workflow first
- Inform user: "To run a Well-Architected Review, I need to understand your architecture first. Let me analyze it..."

**Important:** Don't ask the user to run another workflow manually. Do it automatically as a prerequisite.

## Step 1: Prepare Architecture Summary

Create a summary of the architecture:

**Services detected:**
- [List of AWS services with counts]

**Architecture patterns identified:**
- Serverless (Lambda, API Gateway, DynamoDB)
- Container-based (ECS, Fargate)
- Traditional (EC2, RDS)
- Event-driven (EventBridge, SQS, SNS)
- etc.

**Key components:**
- Entry points (API Gateway, CloudFront, ALB)
- Compute (Lambda, EC2, ECS)
- Data stores (DynamoDB, RDS, S3)
- Integration (SQS, SNS, EventBridge)

## Step 2: Evaluate Against 6 Pillars

For each pillar, use `aws-documentation` MCP server to search for best practices.

### Pillar 1: Operational Excellence

**Focus:** Run and monitor systems, continually improve processes and procedures.

**Use MCP to search:**
```
search_documentation("operational excellence best practices")
search_documentation("monitoring and observability AWS")
search_documentation("infrastructure as code best practices")
```

**Evaluate:**

✅ **Strengths:**
- Infrastructure as Code detected (CDK/Terraform/CloudFormation)
- [Check for CloudWatch, X-Ray, CloudTrail]
- [Check for automated deployments]

⚠️ **Concerns:**
- No monitoring configuration found
- No logging setup detected
- No alerting configured

❌ **Missing:**
- CloudWatch dashboards
- X-Ray tracing
- Automated deployment pipeline

**Recommendations:**
1. Implement CloudWatch Logs for all Lambda functions
2. Add X-Ray tracing for distributed tracing
3. Set up CloudWatch alarms for critical metrics
4. Implement CI/CD pipeline (CodePipeline, GitHub Actions)

**Score:** [0-100 based on findings]

---

### Pillar 2: Security

**Focus:** Protect information, systems, and assets.

**Use MCP to search:**
```
search_documentation("security best practices AWS")
search_documentation("IAM least privilege")
search_documentation("encryption at rest and in transit")
```

**Evaluate:**

✅ **Strengths:**
- [Check for IAM roles with least privilege]
- [Check for encryption settings]
- [Check for VPC, Security Groups]

⚠️ **Concerns:**
- IAM policies may be too permissive
- No encryption at rest configured for [service]
- Public access not blocked on S3 buckets

❌ **Missing:**
- AWS Secrets Manager for credentials
- VPC for network isolation
- WAF for API protection
- GuardDuty for threat detection

**Recommendations:**
1. Review IAM policies for least privilege
2. Enable encryption at rest for DynamoDB/RDS/S3
3. Enable encryption in transit (HTTPS/TLS)
4. Implement VPC with private subnets
5. Use Secrets Manager for sensitive data
6. Enable MFA for critical operations

**Score:** [0-100 based on findings]

---

### Pillar 3: Reliability

**Focus:** Recover from failures, meet demand, mitigate disruptions.

**Use MCP to search:**
```
search_documentation("reliability best practices AWS")
search_documentation("multi-az deployment")
search_documentation("backup and disaster recovery")
```

**Evaluate:**

✅ **Strengths:**
- [Check for multi-AZ deployments]
- [Check for auto-scaling]
- [Check for backup configurations]

⚠️ **Concerns:**
- Single-AZ deployment detected
- No backup strategy configured
- No auto-scaling configured

❌ **Missing:**
- Multi-AZ for RDS/DynamoDB
- Backup and restore procedures
- Auto-scaling for compute resources
- Health checks and automatic recovery

**Recommendations:**
1. Enable multi-AZ for RDS instances
2. Configure DynamoDB global tables for DR
3. Implement backup strategy with retention policies
4. Add auto-scaling for Lambda/ECS/EC2
5. Configure health checks and automatic failover
6. Implement circuit breakers for external dependencies

**Score:** [0-100 based on findings]

---

### Pillar 4: Performance Efficiency

**Focus:** Use computing resources efficiently.

**Use MCP to search:**
```
search_documentation("performance efficiency best practices")
search_documentation("caching strategies AWS")
search_documentation("right-sizing instances")
```

**Evaluate:**

✅ **Strengths:**
- [Check for caching (CloudFront, ElastiCache)]
- [Check for appropriate instance types]
- [Check for CDN usage]

⚠️ **Concerns:**
- No caching layer detected
- Lambda memory may not be optimized
- No CDN for static content

❌ **Missing:**
- CloudFront for content delivery
- ElastiCache for database caching
- DynamoDB DAX for caching
- Lambda Power Tuning for optimization

**Recommendations:**
1. Add CloudFront for static content delivery
2. Implement ElastiCache for frequently accessed data
3. Use DynamoDB DAX for read-heavy workloads
4. Optimize Lambda memory allocation
5. Use appropriate instance types (Graviton2 for cost/performance)
6. Implement database query optimization

**Score:** [0-100 based on findings]

---

### Pillar 5: Cost Optimization

**Focus:** Avoid unnecessary costs.

**Use MCP to search:**
```
search_documentation("cost optimization best practices")
search_documentation("reserved instances savings")
search_documentation("right-sizing recommendations")
```

**Evaluate:**

✅ **Strengths:**
- [Check for serverless usage (pay-per-use)]
- [Check for auto-scaling]
- [Check for S3 lifecycle policies]

⚠️ **Concerns:**
- No Reserved Instances for predictable workloads
- No S3 lifecycle policies configured
- Lambda memory may be over-provisioned

❌ **Missing:**
- Reserved Instances for RDS/EC2
- S3 Intelligent-Tiering
- DynamoDB on-demand vs provisioned analysis
- Cost allocation tags

**Recommendations:**
1. Consider Reserved Instances for RDS (save up to 72%)
2. Implement S3 lifecycle policies (move to IA/Glacier)
3. Use S3 Intelligent-Tiering for unknown access patterns
4. Right-size Lambda memory allocation
5. Use DynamoDB on-demand for unpredictable workloads
6. Implement cost allocation tags for tracking
7. Set up AWS Budgets and alerts

**Score:** [0-100 based on findings]

---

### Pillar 6: Sustainability

**Focus:** Minimize environmental impact.

**Use MCP to search:**
```
search_documentation("sustainability best practices AWS")
search_documentation("carbon footprint reduction")
search_documentation("efficient resource usage")
```

**Evaluate:**

✅ **Strengths:**
- [Check for serverless usage (efficient)]
- [Check for auto-scaling (no idle resources)]
- [Check for regions with renewable energy]

⚠️ **Concerns:**
- Resources may be over-provisioned
- No auto-shutdown for non-production environments
- Region not optimized for sustainability

❌ **Missing:**
- Auto-shutdown schedules for dev/test
- Right-sizing of resources
- Use of Graviton processors (more efficient)

**Recommendations:**
1. Use AWS regions with renewable energy (us-west-2, eu-west-1)
2. Implement auto-shutdown for non-production environments
3. Right-size all resources to avoid waste
4. Use Graviton2/3 instances (better performance per watt)
5. Optimize Lambda memory to reduce execution time
6. Use S3 Intelligent-Tiering to reduce storage footprint

**Score:** [0-100 based on findings]

---

## Step 3: Calculate Overall Score

**Scoring methodology:**
- Each pillar: 0-100 points
- Overall score: Average of 6 pillars

**Weighting (optional):**
- Security: 1.5x weight (most critical)
- Reliability: 1.2x weight
- Others: 1.0x weight

**Overall Score:** [X]/100

**Rating:**
- 90-100: Excellent
- 75-89: Good
- 60-74: Fair
- Below 60: Needs Improvement

## Step 4: Generate Report

Create `architecture-docs/WellArchitectedFrameworkReview.md`:

```markdown
# AWS Well-Architected Framework Review (WAFR)

**Project:** [Project Name]
**Date:** [Date]
**Reviewer:** AWS Architect Companion
**Overall Score:** [X]/100

## Executive Summary

This architecture has been evaluated against the AWS Well-Architected Framework's 6 pillars. 

**Key Findings:**
- [Highlight 2-3 major strengths]
- [Highlight 2-3 critical concerns]

**Priority Recommendations:**
1. [Top recommendation]
2. [Second recommendation]
3. [Third recommendation]

## Architecture Overview

**Services Detected:**
- [List of services]

**Architecture Pattern:**
- [Serverless/Container/Traditional/Hybrid]

**Entry Points:**
- [API Gateway, CloudFront, ALB, etc.]

## Pillar Analysis

### 1. Operational Excellence (Score: X/100)

**Strengths:**
- [List strengths]

**Concerns:**
- [List concerns]

**Recommendations:**
- [List recommendations with priority]

**Resources:**
- [Links to AWS documentation]

---

[Repeat for all 6 pillars]

---

## Summary of Recommendations

### High Priority (Critical)
- [ ] [Recommendation 1]
- [ ] [Recommendation 2]

### Medium Priority (Important)
- [ ] [Recommendation 3]
- [ ] [Recommendation 4]

### Low Priority (Nice to Have)
- [ ] [Recommendation 5]
- [ ] [Recommendation 6]

## Next Steps

1. Review high-priority recommendations with team
2. Create implementation plan with timeline
3. Re-run review after implementing changes
4. Consider AWS Well-Architected Tool for deeper analysis

## Additional Resources

- AWS Well-Architected Framework: https://aws.amazon.com/architecture/well-architected/
- AWS Well-Architected Tool: https://aws.amazon.com/well-architected-tool/
- AWS Architecture Center: https://aws.amazon.com/architecture/
```

## Step 5: Report to User

Present summary:

```
Well-Architected Framework Review Complete!

📊 Overall Score: [X]/10

📈 Pillar Scores:
- Operational Excellence: [X]/10
- Security: [X]/10
- Reliability: [X]/10
- Performance Efficiency: [X]/10
- Cost Optimization: [X]/10
- Sustainability: [X]/10

🎯 Top 3 Recommendations:
1. [Recommendation]
2. [Recommendation]
3. [Recommendation]

📄 Full report: architecture-docs/WellArchitectedFrameworkReview.md

**CRITICAL:** Only generate WellArchitectedFrameworkReview.md. Do NOT create:
- README.md
- INDICE.md
- RESUMEN_EJECUTIVO.md (this is generated separately in complete report)
- Any other documentation files

Would you like me to:
1. Explain any specific finding
2. Generate cost analysis
3. Suggest implementation steps
4. Create updated architecture diagram
```

## Error Handling

**If architecture not analyzed:**
- Run diagram-from-design or diagram-from-code first
- Extract services and configurations

**If MCP documentation search fails:**
- Use cached best practices knowledge
- Document that search was unavailable

**If service not recognized:**
- Skip specific recommendations for that service
- Provide general best practices

**If no issues found:**
- Still generate report with all strengths
- Suggest advanced optimizations

## Tips for Accurate Reviews

- **Be specific:** Reference actual services and configurations
- **Provide context:** Explain why a recommendation matters
- **Prioritize:** Not all recommendations are equally important
- **Link to docs:** Include AWS documentation links
- **Be constructive:** Focus on improvements, not just problems
- **Consider trade-offs:** Some recommendations may conflict (cost vs reliability)
- **Ask questions:** If configuration is unclear, ask user for clarification
