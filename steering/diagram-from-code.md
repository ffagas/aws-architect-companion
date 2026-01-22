# Diagram from Code - Steering File

Generate architecture diagrams from infrastructure as code (CDK, Terraform, CloudFormation, SAM).

## Step 1: Detect IaC Type

Identify what type of infrastructure code exists in the workspace:

### 1.1: Check for CDK
Use `glob` to search for CDK files:
- Pattern: `**/*.ts` or `**/*.py` with CDK imports
- Look for: `import * as cdk from 'aws-cdk-lib'` or `from aws_cdk import`
- Common locations: `lib/`, `src/`, `infrastructure/`, `cdk/`

### 1.2: Check for Terraform
Use `glob` to search for Terraform files:
- Pattern: `**/*.tf`
- Look for: `resource "aws_*"` or `provider "aws"`
- Common locations: root directory, `terraform/`, `infra/`

### 1.3: Check for CloudFormation
Use `glob` to search for CloudFormation templates:
- Pattern: `**/*.yaml`, `**/*.yml`, `**/*.json`
- Look for: `AWSTemplateFormatVersion` key
- Common names: `template.yaml`, `cloudformation.yaml`, `stack.yaml`

### 1.4: Check for SAM
Use `glob` to search for SAM templates:
- Pattern: `**/template.yaml`, `**/template.yml`
- Look for: `Transform: AWS::Serverless-2016-10-31`
- Common location: root directory

**If multiple types found:** Ask user which one to analyze

**If none found:** Ask user to specify the IaC type and location

## Step 2: Analyze Infrastructure Code

### Option A: Use MCP Server Analysis (Recommended for CDK/Terraform)

The `aws-pricing` MCP server provides built-in analysis tools:

**For CDK projects:**
```
Use analyze_cdk_project(project_path) tool
- Returns: List of AWS services detected
- Returns: Service configurations
- Returns: Relationships between resources
```

**For Terraform projects:**
```
Use analyze_terraform_project(project_path) tool
- Returns: List of AWS services detected
- Returns: Resource configurations
- Returns: Dependencies
```

**Advantages:**
- Automatic service detection
- Handles complex constructs
- Extracts configurations
- Identifies relationships

**Process after MCP analysis:**
1. Review the list of services returned
2. Use `search_documentation` to understand any unfamiliar services
3. Verify service relationships make sense
4. Check for any services that might be missing

### Option B: Manual Analysis (For CloudFormation/SAM or if MCP fails)

**Strategy:** Use your knowledge of IaC syntax and AWS services to identify resources.

#### For CDK (TypeScript/Python):

**Process:**

1. **Search for AWS service constructs:**
   - Use `grep` to find patterns like `new [service].[Resource](`
   - Example: `grep -r "new lambda\\.Function" .`
   - Example: `grep -r "new s3\\.Bucket" .`

2. **Identify services from constructs:**
   - Don't rely on a hardcoded list
   - Use your knowledge: `new lambda.Function` → Lambda service
   - Use `search_documentation` to verify: `search_documentation("AWS CDK Lambda construct")`

3. **Extract configurations:**
   - Look at construct properties (memory, timeout, environment, etc.)
   - These will be useful for cost analysis and WAR

4. **Detect relationships:**
   - `.grantReadWrite()` → Permission relationship
   - `.addEventSource()` → Event trigger
   - `.addTarget()` → Target relationship
   - Use `search_documentation` to understand relationship patterns

#### For Terraform:

**Process:**

1. **Search for AWS resources:**
   - Use `grep` to find `resource "aws_*"`
   - Example: `grep -r 'resource "aws_' .`

2. **Identify services from resource types:**
   - Pattern: `resource "aws_[service]_[resource]"`
   - Example: `aws_lambda_function` → Lambda service
   - Example: `aws_dynamodb_table` → DynamoDB service
   - Use `search_documentation` to verify service names

3. **Extract configurations:**
   - Look at resource attributes
   - Check for variables and locals

4. **Detect relationships:**
   - `depends_on = [...]` → Explicit dependency
   - `${aws_[service].[name].[attribute]}` → Reference/relationship
   - Use `search_documentation` to understand Terraform AWS provider patterns

#### For CloudFormation/SAM:

**Process:**

1. **Search for AWS resource types:**
   - Use `grep` to find `Type: AWS::`
   - Example: `grep -r "Type: AWS::" .`

2. **Identify services from resource types:**
   - Pattern: `Type: AWS::[Service]::[Resource]`
   - Example: `AWS::Lambda::Function` → Lambda service
   - Example: `AWS::DynamoDB::Table` → DynamoDB service
   - Use `search_documentation` to verify resource types

3. **Extract configurations:**
   - Look at Properties section
   - Check for Parameters and Mappings

4. **Detect relationships:**
   - `DependsOn:` → Explicit dependency
   - `!Ref` or `!GetAtt` → Reference/relationship
   - Use `search_documentation` to understand CloudFormation intrinsic functions

### Verify Services with Documentation

**For any detected service:**

1. **Confirm it's a valid AWS service:**
   ```
   Use search_documentation("[service name] AWS service")
   ```

2. **Understand service purpose:**
   - Read documentation to understand what the service does
   - This helps with diagram labeling and relationship inference

3. **Check for service limits and constraints:**
   ```
   Use search_documentation("[service name] service limits")
   ```

4. **Identify common integration patterns:**
   ```
   Use search_documentation("[service A] [service B] integration")
   ```

**Important:** AWS has 200+ services and new ones are added regularly. Don't rely on hardcoded lists. Use your knowledge, code analysis, and aws-documentation MCP to identify services dynamically.

## Step 3: Map Services to Diagram Icons

Use `list_icons` tool from `aws-diagram` MCP server with `provider_filter="aws"` to discover available icons dynamically.

**Process:**

1. **Get all AWS icons:**
   ```
   Use list_icons(provider_filter="aws")
   ```

2. **Map detected services to icon names:**
   - Match service names from code to icon names
   - Icon naming pattern: `aws.[category].[ServiceName]`
   - Example: Lambda service → `aws.compute.Lambda`

3. **Handle services without exact match:**
   - Search for similar services in the icon list
   - Use `search_documentation` to find the correct service category
   - Example: "Bedrock Agent" → search for ML/AI category icons
   - If no icon exists, use generic icon from same category

4. **Common icon categories:**
   - `aws.compute.*` - Lambda, EC2, ECS, Fargate, etc.
   - `aws.storage.*` - S3, EBS, EFS, FSx, etc.
   - `aws.database.*` - DynamoDB, RDS, Aurora, DocumentDB, etc.
   - `aws.network.*` - API Gateway, CloudFront, VPC, ALB, NLB, etc.
   - `aws.integration.*` - SQS, SNS, EventBridge, Step Functions, AppSync, etc.
   - `aws.security.*` - Cognito, IAM, Secrets Manager, KMS, etc.
   - `aws.ml.*` - SageMaker, Bedrock, Rekognition, Comprehend, etc.
   - `aws.analytics.*` - Kinesis, Athena, Glue, EMR, etc.
   - `aws.management.*` - CloudWatch, CloudFormation, Systems Manager, etc.
   - `aws.containers.*` - ECS, EKS, ECR, etc.

5. **Verify icon availability:**
   - Check that the icon exists in the list returned by `list_icons`
   - If not available, choose closest alternative
   - Document any services that couldn't be visualized

**Important:** Don't hardcode icon mappings. AWS adds new services regularly, and the diagrams package is updated to include new icons. Always use `list_icons` to discover available icons dynamically.

## Step 4: Generate Diagram Code

Build Python code using the `diagrams` package:

```python
from diagrams import Diagram, Cluster, Edge
from diagrams.aws.compute import Lambda
from diagrams.aws.database import Dynamodb
from diagrams.aws.network import APIGateway
# ... import detected services

with Diagram("Architecture - Actual Implementation", show=False, direction="LR"):
    # Create nodes for detected services
    api = APIGateway("API Gateway")
    function = Lambda("Lambda Function")
    database = Dynamodb("DynamoDB Table")
    
    # Create connections based on detected relationships
    api >> function >> database
```

**Guidelines:**
- Use `show=False` to save file without displaying
- Use `direction="LR"` for left-to-right flow
- Use actual resource names from code as labels
- Group related resources with `Cluster` if applicable
- Filename: `architecture-actual.png`

**Example with multiple resources:**
```python
with Diagram("Actual Architecture", show=False, direction="LR"):
    with Cluster("API Layer"):
        api = APIGateway("UserAPI")
        auth = Lambda("AuthFunction")
    
    with Cluster("Business Logic"):
        process = Lambda("ProcessFunction")
        validate = Lambda("ValidateFunction")
    
    with Cluster("Data Layer"):
        users_db = Dynamodb("UsersTable")
        orders_db = Dynamodb("OrdersTable")
    
    api >> auth >> process >> users_db
    process >> validate >> orders_db
```

## Step 5: Generate Diagram

Use `generate_diagram` tool from `aws-diagram` MCP server:
- Pass the Python code as string
- Specify filename: `architecture-docs/architecture-actual.png`
- Tool creates PNG in architecture-docs directory
- Create directory if it doesn't exist

**Important:** Always save outputs to `architecture-docs/` directory to keep all architecture documentation organized.

## Step 6: Compare with Design (Optional)

If `design.md` exists in the workspace:

1. Check if design document was already analyzed
2. If not, follow steps from `diagram-from-design.md` to extract designed services
3. Compare detected services:
   - Services in design but NOT in code
   - Services in code but NOT in design
   - Services in both (matching)

4. Create comparison report (see `design-vs-code.md` for details)

## Step 7: Report Results

Report to user:
- **IaC type detected:** CDK/Terraform/CloudFormation/SAM
- **Services found:** [list with counts]
- **Diagram generated:** `architecture-docs/architecture-actual.png`
- **Relationships mapped:** [count]

**CRITICAL:** Only generate `architecture-actual.png`. Do NOT create:
- README.md
- INDICE.md
- RESUMEN_EJECUTIVO.md
- Any other documentation files

If design.md exists:
- **Comparison available:** Suggest running design vs code comparison

Ask user if they want to:
- Regenerate with different grouping
- Add missing services manually
- Run design vs code comparison
- Generate Well-Architected Framework Review (WAFR)

## Error Handling

**If no IaC found:**
- Ask user to specify IaC type and location
- Suggest checking if files are in subdirectories

**If MCP analysis fails:**
- Fall back to manual grep-based analysis
- Report which services were detected manually

**If service icons not found:**
- Use closest match or generic icon
- Document unmapped services

**If diagram generation fails:**
- Verify GraphViz is installed
- Check Python code syntax
- Verify `aws-diagram` MCP server is running

## Example Workflow

**User request:** "Generate a diagram from my CDK code"

**Agent actions:**
1. Search for CDK files with glob: `**/*.ts`
2. Detect CDK project in `lib/` directory
3. Use `analyze_cdk_project(".")` from aws-pricing MCP
4. Extract services: API Gateway, 3 Lambda functions, DynamoDB, S3
5. Map to icons with `list_icons(provider_filter="aws")`
6. Build diagram code with proper imports and connections
7. Generate with `generate_diagram(code, filename="architecture-docs/architecture-actual.png")`
8. Check for design.md → Found
9. Report: "Created architecture-docs/architecture-actual.png with 6 services. Design.md found - would you like to compare?"

## Tips for Accurate Diagrams

- **Use MCP analysis when possible:** More accurate than grep
- **Check for implicit relationships:** Permissions often indicate connections
- **Group by logical layers:** API, Business Logic, Data, Infrastructure
- **Use actual resource names:** Makes diagram more meaningful
- **Include VPC/networking if defined:** Shows security boundaries
- **Document unmapped services:** Help improve future versions
