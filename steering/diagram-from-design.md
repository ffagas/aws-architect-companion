# Diagram from Design - Steering File

Generate architecture diagrams from design documents (design.md, architecture.md, or similar).

## Step 1: Locate Design Document

Search for design documentation in the workspace:

1. Look for common design document names:
   - `design.md`
   - `architecture.md`
   - `DESIGN.md`
   - `ARCHITECTURE.md`
   - Files in `docs/` or `documentation/` directories

2. Use `glob` tool to find candidates:
   - Pattern: `**/design.md` or `**/architecture.md`
   - Pattern: `**/DESIGN.md` or `**/ARCHITECTURE.md`

3. If multiple files found, ask user which one to use

4. If no design document found:
   - Ask user to specify the file path
   - Or ask user to describe the architecture verbally

## Step 2: Read and Analyze Design Document

Use `fs_read` to read the design document content.

### Extract AWS Services

**Strategy:** Use your knowledge of AWS services and the aws-documentation MCP server to identify services mentioned in the design.

**Process:**

1. **Read the entire document** and identify any mentions of AWS services or cloud infrastructure components

2. **Look for explicit service names:**
   - Direct mentions: "Lambda", "DynamoDB", "S3", "API Gateway"
   - AWS prefixes: "AWS Lambda", "Amazon S3", "Amazon DynamoDB"
   - Service categories: "serverless functions", "NoSQL database", "object storage"

3. **Look for implicit service references:**
   - "REST API" → likely API Gateway
   - "serverless functions" → Lambda
   - "NoSQL database" → DynamoDB
   - "object storage" → S3
   - "CDN" or "content delivery" → CloudFront
   - "message queue" → SQS
   - "pub/sub" → SNS
   - "container orchestration" → ECS/EKS
   - "relational database" → RDS/Aurora
   - "authentication" → Cognito
   - "monitoring" → CloudWatch

4. **Use aws-documentation MCP to verify services:**
   - For each potential service identified, use `search_documentation` to confirm it's a valid AWS service
   - Example: `search_documentation("AWS Lambda service overview")`
   - This helps distinguish between AWS services and general terms

5. **Resolve ambiguities:**
   - If design mentions "database" without specifics, check context:
     - Mentions of "NoSQL", "key-value", "document" → DynamoDB
     - Mentions of "SQL", "relational", "MySQL", "PostgreSQL" → RDS/Aurora
     - Mentions of "data warehouse", "analytics" → Redshift
   - Use `search_documentation` to understand service capabilities and match to requirements

6. **Create a list of identified services:**
   ```
   Identified AWS Services:
   - API Gateway (mentioned as "REST API endpoint")
   - Lambda (mentioned as "serverless compute")
   - DynamoDB (mentioned as "NoSQL database for user data")
   - S3 (mentioned as "file storage")
   - CloudFront (mentioned as "CDN for static assets")
   ```

**Important:** Don't rely on a hardcoded list. AWS has 200+ services and new ones are added regularly. Use your knowledge and the documentation MCP to identify services dynamically.

### Detect Relationships

**Strategy:** Identify how services connect and interact using context from the design document and AWS documentation.

**Process:**

1. **Look for explicit relationship indicators:**
   - Connection words: "connects to", "calls", "invokes", "triggers", "sends to", "publishes to", "subscribes to"
   - Flow indicators: "→", "->", "=>", "flows to", "passes data to"
   - Integration terms: "integrates with", "uses", "depends on"

2. **Identify relationship patterns from context:**
   - "API Gateway receives requests and forwards to Lambda" → API Gateway → Lambda
   - "Lambda stores data in DynamoDB" → Lambda → DynamoDB
   - "S3 bucket triggers Lambda on upload" → S3 → Lambda (event trigger)
   - "CloudFront serves content from S3" → CloudFront → S3 (origin)

3. **Use aws-documentation MCP to understand service relationships:**
   - For each service pair, search for integration patterns
   - Example: `search_documentation("API Gateway Lambda integration")`
   - Example: `search_documentation("S3 event notifications Lambda")`
   - This helps identify standard AWS integration patterns

4. **Infer implicit relationships:**
   - If design mentions "serverless API", likely pattern: API Gateway → Lambda → DynamoDB
   - If design mentions "static website", likely pattern: CloudFront → S3
   - If design mentions "event-driven processing", likely pattern: EventBridge/SQS → Lambda
   - Use `search_documentation` to validate these patterns

5. **Check for common AWS patterns:**
   - Use `search_documentation("AWS architecture patterns")` to find reference architectures
   - Match design requirements to known patterns
   - Example: "microservices" → likely ECS/EKS + ALB + RDS/DynamoDB

6. **Resolve ambiguous relationships:**
   - If relationship is unclear, search for service capabilities
   - Example: `search_documentation("DynamoDB Streams Lambda trigger")`
   - Ask user for clarification if needed

7. **Create relationship map:**
   ```
   Service Relationships:
   - API Gateway → Lambda (REST API integration)
   - Lambda → DynamoDB (read/write operations)
   - Lambda → S3 (file storage)
   - S3 → Lambda (event trigger on upload)
   - CloudFront → S3 (origin for static content)
   ```

**Important:** Use AWS documentation to understand:
- Valid integration patterns between services
- Service limits and constraints
- Best practices for service connections
- Security considerations (IAM roles, permissions)

## Step 3: Generate Diagram Code

Use the `aws-diagram` MCP server to generate the diagram.

### 3.1: List Available Icons

Use `list_icons` tool with `provider_filter="aws"` to discover available AWS service icons dynamically.

**Process:**

1. **Get all AWS icons:**
   ```
   Use list_icons(provider_filter="aws")
   ```

2. **Map detected services to icon names:**
   - The tool returns icon names like `aws.compute.Lambda`, `aws.storage.S3`, etc.
   - Match your detected services to these icon names
   - Icon naming pattern: `aws.[category].[ServiceName]`

3. **Handle services without exact icon match:**
   - If exact match not found, search for similar services
   - Example: "Amazon Bedrock" might be under `aws.ml.Bedrock` or `aws.ai.Bedrock`
   - Use `search_documentation` to find the correct service category
   - If no icon exists, use a generic icon from the same category

4. **Common icon categories:**
   - `aws.compute.*` - Lambda, EC2, ECS, etc.
   - `aws.storage.*` - S3, EBS, EFS, etc.
   - `aws.database.*` - DynamoDB, RDS, Aurora, etc.
   - `aws.network.*` - API Gateway, CloudFront, VPC, ALB, etc.
   - `aws.integration.*` - SQS, SNS, EventBridge, Step Functions, etc.
   - `aws.security.*` - Cognito, IAM, Secrets Manager, etc.
   - `aws.ml.*` - SageMaker, Bedrock, Rekognition, etc.
   - `aws.analytics.*` - Kinesis, Athena, Glue, etc.
   - `aws.management.*` - CloudWatch, CloudFormation, etc.

5. **Document unmapped services:**
   - If a service cannot be mapped to an icon, document it
   - Use a generic icon or text label
   - Report to user which services couldn't be visualized

**Example mapping:**
```
Detected Service → Icon Name
- Lambda → aws.compute.Lambda
- DynamoDB → aws.database.Dynamodb
- S3 → aws.storage.S3
- API Gateway → aws.network.APIGateway
- Bedrock Agent → aws.ml.Bedrock (or generic ML icon)
```

**Important:** Don't hardcode icon mappings. Use `list_icons` to discover available icons dynamically, as new services and icons are added regularly.

### 3.2: Build Diagram Code

Create Python code using the `diagrams` package syntax:

```python
from diagrams import Diagram, Cluster, Edge
from diagrams.aws.compute import Lambda
from diagrams.aws.database import Dynamodb
from diagrams.aws.network import APIGateway
# ... import other services as needed

with Diagram("Architecture Diagram", show=False, direction="LR"):
    # Create service nodes
    api = APIGateway("API Gateway")
    function = Lambda("Lambda Function")
    database = Dynamodb("DynamoDB")
    
    # Create connections
    api >> function >> database
```

**Important guidelines:**
- Use `show=False` to prevent display (we only want the file)
- Use `direction="LR"` for left-to-right flow (user/client on left)
- Use descriptive labels for each service
- Use `>>` for connections
- Use `Cluster` to group related services if applicable

**Example with grouping:**
```python
with Diagram("Architecture", show=False, direction="LR"):
    with Cluster("Frontend"):
        cloudfront = CloudFront("CDN")
        s3 = S3("Static Assets")
    
    with Cluster("Backend"):
        api = APIGateway("API")
        lambda_fn = Lambda("Handler")
        db = Dynamodb("Database")
    
    cloudfront >> s3
    cloudfront >> api >> lambda_fn >> db
```

### 3.3: Generate Diagram

Use `generate_diagram` tool from `aws-diagram` MCP server:
- Pass the Python code as a string
- Specify filename: `architecture-docs/architecture.png`
- The tool will create the PNG file in the architecture-docs directory
- If the directory doesn't exist, create it first

**Important:** Always save outputs to `architecture-docs/` directory to keep all architecture documentation organized.

## Step 4: Verify and Report

1. Confirm the diagram was generated successfully
2. Report to user:
   - Location of generated file: `architecture-docs/architecture.png`
   - Services detected: [list]
   - Relationships mapped: [count]

**CRITICAL:** Only generate `architecture.png`. Do NOT create:
- README.md
- INDICE.md
- RESUMEN_EJECUTIVO.md
- Any other documentation files

3. Ask if user wants to:
   - Regenerate with different layout
   - Add more services
   - Adjust grouping or labels
   - Run other capabilities (code analysis, WAFR, cost analysis)

## Error Handling

**If design document not found:**
- Ask user to specify file path or describe architecture

**If no AWS services detected:**
- Ask user to confirm the document contains AWS architecture
- Suggest checking for service mentions

**If diagram generation fails:**
- Check that GraphViz is installed
- Verify `aws-diagram` MCP server is running
- Check for syntax errors in generated Python code

**If service icon not found:**
- Use generic icon or closest match
- Document which services couldn't be mapped

## Example Workflow

**User request:** "Generate an architecture diagram from my design.md"

**Agent actions:**
1. Find `design.md` using glob
2. Read content with `fs_read`
3. Extract services: API Gateway, Lambda, DynamoDB, S3
4. Detect relationships: API Gateway → Lambda → DynamoDB, Lambda → S3
5. List icons with `list_icons(provider_filter="aws")`
6. Build diagram code with proper imports and connections
7. Generate diagram with `generate_diagram(code, filename="architecture.png")`
8. Report: "Created architecture.png with 4 services and 3 connections"

## Tips for Better Diagrams

- **Start with user/client on the left**: Place entry points (CloudFront, API Gateway) on the left
- **Flow left to right**: Data should flow from left to right
- **Group related services**: Use Cluster for logical grouping (Frontend, Backend, Data Layer)
- **Use descriptive labels**: "User Authentication" instead of just "Lambda"
- **Keep it simple**: Don't overcrowd the diagram, focus on main components
