# Design vs Code Comparison - Steering File

Compare design documentation against actual infrastructure code implementation.

## Prerequisites

**This steering file requires BOTH:**
1. Design document analyzed (services extracted)
2. Infrastructure code analyzed (services detected)

**If not done yet:**
- Check for design.md → If found, run `diagram-from-design.md` workflow
- Check for infrastructure code → If found, run `diagram-from-code.md` workflow
- Inform user: "To compare design vs code, I need to analyze both. Let me do that first..."

**If only one is available:**
- If only design.md: Inform user that code is needed for comparison
- If only code: Inform user that design.md is needed for comparison
- Offer to generate diagram from what's available instead

**Important:** Handle prerequisites automatically. Run both analysis workflows before comparison.

## Step 1: Prepare Service Lists

### From Design Document
Extract list of AWS services mentioned in design.md:
- Service names (e.g., "Lambda", "DynamoDB", "S3")
- Approximate counts if mentioned
- Intended purposes/roles

### From Infrastructure Code
Extract list of AWS services from IaC:
- Service types (e.g., "AWS::Lambda::Function")
- Actual resource counts
- Resource names and configurations

## Step 2: Normalize Service Names

Normalize both lists to common service names for comparison:

**Design variations → Standard name:**
- "Lambda", "Lambda Function", "AWS Lambda" → "Lambda"
- "S3", "S3 Bucket", "Simple Storage Service" → "S3"
- "DynamoDB", "DynamoDB Table", "Dynamo" → "DynamoDB"
- "API Gateway", "Amazon API Gateway", "APIGW" → "API Gateway"
- etc.

**Code variations → Standard name:**
- `AWS::Lambda::Function` → "Lambda"
- `aws_lambda_function` → "Lambda"
- `lambda.Function` → "Lambda"
- etc.

## Step 3: Compare Services

### 3.1: Services in Design but NOT in Code

Identify services mentioned in design that are missing from implementation:

```
Missing from Implementation:
- [Service Name]: Mentioned in design but not found in code
  - Design context: [where/how it was mentioned]
  - Possible reasons: Not yet implemented, replaced by alternative, or named differently
```

**Analysis:**
- Is this a critical service?
- Was it replaced by an alternative?
- Is it planned for future implementation?

### 3.2: Services in Code but NOT in Design

Identify services in code that weren't documented in design:

```
Not Documented in Design:
- [Service Name]: Found in code but not mentioned in design
  - Code location: [file/resource name]
  - Possible reasons: Added during implementation, infrastructure service, or design doc outdated
```

**Analysis:**
- Is this an infrastructure service (VPC, IAM roles)?
- Was it added as an implementation detail?
- Should the design doc be updated?

### 3.3: Services in Both (Matching)

Identify services that exist in both design and code:

```
Matching Services:
- [Service Name]: ✅ Present in both design and code
  - Design count: [X] (if mentioned)
  - Actual count: [Y]
  - Match: [Yes/No - if counts differ]
```

**For each matching service, check:**
- Do counts match (if specified in design)?
- Do configurations align with design intent?
- Are relationships as designed?

## Step 4: Analyze Relationships

Compare how services connect:

### Design Relationships
From design document:
- "API Gateway → Lambda → DynamoDB"
- "S3 triggers Lambda"
- etc.

### Code Relationships
From infrastructure code:
- API Gateway integration with Lambda
- Lambda event source from S3
- Lambda permissions to DynamoDB
- etc.

### Comparison
```
Relationship Comparison:
✅ API Gateway → Lambda: Matches design
✅ Lambda → DynamoDB: Matches design
⚠️  Lambda → S3: Found in code but not in design
❌ CloudFront → S3: In design but not implemented
```

## Step 5: Analyze Configurations

For matching services, compare key configurations:

### Lambda Functions
- **Design:** "Serverless function with 512MB memory"
- **Code:** `memory_size = 256`
- **Status:** ⚠️ Configuration mismatch (256MB vs 512MB)

### DynamoDB Tables
- **Design:** "NoSQL database with on-demand billing"
- **Code:** `billing_mode = "PROVISIONED"`
- **Status:** ⚠️ Billing mode mismatch

### S3 Buckets
- **Design:** "Private bucket with versioning"
- **Code:** `versioning = { enabled = true }, public_access_block = { block_public_acls = true }`
- **Status:** ✅ Matches design intent

## Step 6: Generate Comparison Report

Create `architecture-docs/differences.md` with the following structure:

```markdown
# Design vs Implementation Comparison

**Generated:** [Date]
**Design Document:** design.md
**Infrastructure Code:** [CDK/Terraform/CloudFormation]

## Executive Summary

- **Total services in design:** [X]
- **Total services in code:** [Y]
- **Matching services:** [Z]
- **Missing from code:** [A]
- **Not in design:** [B]
- **Configuration mismatches:** [C]

## Detailed Comparison

### ✅ Matching Services

[List of services that match]

### ❌ Missing from Implementation

[Services in design but not in code]

**Recommendations:**
- Implement missing services
- Or update design if no longer needed

### ⚠️ Not Documented in Design

[Services in code but not in design]

**Recommendations:**
- Update design documentation
- Or remove if not needed

### ⚠️ Configuration Mismatches

[Services with different configurations]

**Recommendations:**
- Align code with design
- Or update design if intentional change

### 🔗 Relationship Comparison

[Comparison of service connections]

## Recommendations

1. **High Priority:**
   - [Critical mismatches or missing services]

2. **Medium Priority:**
   - [Configuration differences]

3. **Low Priority:**
   - [Documentation updates]

## Next Steps

- [ ] Review missing services and decide: implement or update design
- [ ] Review undocumented services and update design.md
- [ ] Align configurations where mismatches exist
- [ ] Update architecture diagrams if needed
```

## Step 7: Report to User

Present summary to user:

```
Comparison Complete!

📊 Summary:
- Matching services: [X]
- Missing from code: [Y]
- Not in design: [Z]
- Configuration mismatches: [W]

📄 Report generated: architecture-docs/differences.md

**CRITICAL:** Only generate `differences.md`. Do NOT create:
- README.md
- INDICE.md
- RESUMEN_EJECUTIVO.md
- Any other documentation files

Would you like me to:
1. Generate updated architecture diagrams
2. Suggest fixes for mismatches
3. Update design.md with missing services
4. Run Well-Architected Framework Review (WAFR)
```

## Error Handling

**If design document not analyzed:**
- Run `diagram-from-design.md` workflow first
- Extract services from design

**If code not analyzed:**
- Run `diagram-from-code.md` workflow first
- Detect services from IaC

**If service names don't normalize well:**
- Ask user to confirm mappings
- Document ambiguous cases

**If no differences found:**
- Report perfect match
- Still generate report for documentation

## Example Workflow

**User request:** "Compare my design.md with the CDK implementation"

**Agent actions:**
1. Check if design already analyzed → Yes, 5 services found
2. Check if code already analyzed → Yes, 7 services found
3. Normalize service names
4. Compare:
   - Matching: API Gateway, Lambda, DynamoDB
   - Missing from code: CloudFront
   - Not in design: S3, CloudWatch, IAM Role
5. Analyze relationships
6. Check configurations for matching services
7. Generate `differences.md`
8. Report: "Found 3 matching services, 1 missing from code, 3 not documented. Report saved to differences.md"

## Tips for Accurate Comparison

- **Normalize carefully:** Different naming conventions can cause false mismatches
- **Consider infrastructure services:** VPC, IAM, CloudWatch are often implementation details
- **Check context:** A service might be mentioned implicitly in design
- **Ask for clarification:** When unsure about a mismatch, ask the user
- **Focus on intent:** Configuration differences might be intentional optimizations
