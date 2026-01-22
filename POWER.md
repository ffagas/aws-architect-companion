---
name: "aws-architect-companion"
displayName: "AWS Architect Companion"
description: "AWS Solution Architect companion for architecture diagrams, design validation, Well-Architected reviews, and cost analysis"
keywords: ["aws", "architecture", "diagram", "well-architected", "cost", "analysis", "infrastructure", "design", "cdk", "terraform", "cloudformation"]
---

# AWS Architect Companion

Your AI-powered companion for AWS architecture design, validation, and cost analysis.

## How to Activate This Power

**This power activates automatically** when you mention relevant keywords in your conversation with Kiro:

**Keywords:** aws, architecture, diagram, well-architected, wafr, cost, analysis, infrastructure, design, cdk, terraform, cloudformation

**Examples:**
- "Help me design an AWS architecture"
- "Generate a diagram from my design"
- "Review my architecture"
- "Estimate costs"

Just start talking about AWS architecture, and the power will load automatically!

---

## What is AWS Architect Companion?

AWS Architect Companion provides five key capabilities to help AWS Solution Architects and developers:

**0. Architecture Design Assistant** - Interactive design help for new solutions
- Get architecture recommendations based on your requirements
- Leverage AWS best practices from official documentation
- Generate architecture.md with detailed design proposal
- Automatically flows into diagram generation and analysis

1. **Architecture Diagrams from Design** - Generate professional diagrams from your design.md or specification documents
2. **Architecture Diagrams from Code** - Analyze your IaC (CDK, Terraform, CloudFormation) and generate diagrams of your actual implementation
3. **Well-Architected Framework Review (WAFR)** - Automated evaluation against AWS Well-Architected Framework's 6 pillars
4. **Cost Analysis** - Estimate infrastructure costs with interactive assumption collection

## Onboarding

### Step 1: Validate Prerequisites

Before using AWS Architect Companion, ensure the following are installed:

**Required for all capabilities:**
- **uv**: Python package manager (install from https://docs.astral.sh/uv/)
- **Python 3.10+**: Install via `uv python install 3.10`
- **GraphViz**: Required for diagram generation (https://www.graphviz.org/)
  - macOS: `brew install graphviz`
  - Ubuntu/Debian: `sudo apt-get install graphviz`
  - Windows: Download from graphviz.org

**Required only for Cost Analysis (Capability 4):**
- **AWS Credentials**: Configure with `aws configure` or environment variables
- **IAM Permissions**: Your AWS user/role needs `pricing:*` permissions

### Step 2: Verify MCP Servers

This power uses three MCP servers that should be automatically configured:
- `aws-documentation` - For Well-Architected Framework documentation
- `aws-diagram` - For generating architecture diagrams
- `aws-pricing` - For cost analysis (requires AWS credentials)

If you encounter issues, check your `~/.kiro/settings/mcp.json` configuration.

## Understanding Capabilities

**Capability 0: Architecture Design Assistant**
- Interactive conversational design help
- Input: Your requirements (e.g., "Design an ecommerce platform")
- Process: 
  1. Asks clarifying questions about your requirements
  2. Searches AWS documentation for best practices
  3. Proposes architecture with AWS services
  4. Generates `architecture.md` with detailed design
- Output: `architecture.md` (design document)
- Automatically flows into diagram generation
- No AWS credentials needed

**Capability 1: Diagrams from Design**
- Works with: design.md, architecture.md, or any markdown with AWS services
- Output: `architecture-docs/architecture.png`
- No AWS credentials needed

**Capability 2: Diagrams from Code**
- Works with: CDK (TypeScript/Python), Terraform, CloudFormation, SAM
- Output: `architecture-docs/architecture-actual.png`
- No AWS credentials needed

**Capability 3: Design vs Code Comparison**
- Requires: Both design document and infrastructure code
- Output: `architecture-docs/differences.md`
- No AWS credentials needed

**Capability 4: Well-Architected Framework Review (WAFR)**
- Evaluates against 6 pillars: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability
- Output: `architecture-docs/WellArchitectedFrameworkReview.md`
- No AWS credentials needed

**Capability 5: Cost Analysis**
- Interactive assumption collection (users, requests, storage, etc.)
- Output: `architecture-docs/CostAnalysis.md`
- **Requires AWS credentials**

## Output Directory

All outputs from this power are saved to the `architecture-docs/` directory in your project:

```
your-project/
├── architecture.md                       # Architecture design (if using design assistant)
├── architecture-docs/                    # Created by AWS Architect Companion
│   ├── architecture.png                  # From design document
│   ├── architecture-actual.png           # From infrastructure code
│   ├── differences.md                    # Design vs code comparison
│   ├── WellArchitectedFrameworkReview.md # WAFR analysis
│   ├── CostAnalysis.md                   # Cost estimation
│   └── ExecutiveSummary.md               # Executive summary (complete report only)
├── lib/                                  # Your CDK/code (if applicable)
└── ...
```

**Important:** Only these files will be generated. No additional README.md or index files.

This keeps all architecture documentation organized in one place.

## How the Power Works

The power intelligently guides you through the process:

### 1. Understand Your Request

The power interprets natural language requests:
- **"Help me design..."** → Architecture Design Assistant
- "Review my architecture" → Well-Architected Framework Review
- "Estimate costs" → Cost Analysis
- "Compare design vs code" → Design vs Code Comparison
- "Document my infrastructure" → Generate Diagrams

### 2. Architecture Design Assistant (New Solutions)

**When user asks for design help** (e.g., "Help me design an ecommerce platform"):

**Step 1: Gather Requirements**
- Ask clarifying questions:
  - Expected scale? (users, requests/sec)
  - Key features?
  - Compliance requirements?
  - Budget constraints?
  - Deployment preference? (serverless, containers, VMs)

**Step 2: Research Best Practices**
- Use `aws-documentation` MCP to search:
  - Reference architectures for the use case
  - AWS Well-Architected best practices
  - Service-specific recommendations

**Step 3: Propose Architecture**
- Recommend AWS services based on requirements
- Explain service selection rationale
- Highlight trade-offs and alternatives

**Step 4: Generate architecture.md**
- Create detailed design document with:
  - Overview and requirements
  - Architecture description
  - Service selection rationale
  - Data flow explanation
  - Security considerations
  - Scalability approach
  - Implementation phases
- Save to project root: `architecture.md`

**Step 5: Automatic Flow**
- Offer to generate diagram → WAFR → Cost Analysis

**Example:**
```
User: "Help me design a serverless ecommerce platform"

Power: [Asks questions about scale, features, compliance]
Power: [Searches AWS docs for best practices]
Power: "I recommend: API Gateway + Lambda + DynamoDB + S3 + CloudFront + Cognito..."
Power: "Created architecture.md. Generate diagram now?"
```

### 3. Detect Available Inputs

Automatically checks your project:
- ✅ design.md or architecture.md found
- ✅ CDK code detected in lib/
- ✅ Terraform files found
- ✅ CloudFormation templates present

### 4. Handle Prerequisites Automatically

**Smart prerequisite handling:**

If you ask for **Cost Analysis** but no architecture is analyzed:
```
Power: "To estimate costs, I need to identify your services first.
       I found CDK code. Let me analyze it..."
       [Analyzes code → Identifies services → Then asks for cost assumptions]
```

If you ask for **Well-Architected Review** but no services identified:
```
Power: "I'll run a Well-Architected Review. First, let me understand your architecture...
       I found design.md. Analyzing services..."
       [Analyzes design → Identifies services → Then runs WAR]
```

If you ask for **Design vs Code Comparison** but only have code:
```
Power: "I found CDK code but no design.md. 
       I can generate a diagram from your code, but I need a design document to compare.
       Would you like me to just generate the diagram?"
```

### 4. Execute Workflows

Loads the appropriate steering file and executes:
- `diagram-from-design.md` - For design documents
- `diagram-from-code.md` - For infrastructure code
- `design-vs-code.md` - For comparisons
- `well-architected.md` - For WAR
- `cost-analysis.md` - For cost estimation

### 5. Generate Organized Outputs

All outputs saved to `architecture-docs/`:
- Creates directory if it doesn't exist
- Saves files with clear names
- Reports what was generated
- Suggests next steps

## When to Load Steering Files

This power uses specialized steering files for specific workflows. Kiro will automatically load the appropriate file based on your request:

- Generating architecture diagrams from design documents → `steering/diagram-from-design.md`
- Generating architecture diagrams from infrastructure code → `steering/diagram-from-code.md`
- Comparing design documentation vs implementation → `steering/design-vs-code.md`
- Running Well-Architected Framework review → `steering/well-architected.md`
- Estimating infrastructure costs → `steering/cost-analysis.md`

## Complete Report Workflow

**When user requests complete analysis** ("Give me a complete report", "Analyze everything", "Dame un reporte completo"):

Execute all capabilities in this specific order:

### Step 1: Generate Architecture Diagrams (FIRST)

**Purpose:** Identify all services before any analysis.

1. Check for design.md or similar:
   - If found → Follow `steering/diagram-from-design.md`
   - Output: `architecture-docs/architecture.png`

2. Check for infrastructure code (CDK/Terraform/CloudFormation/SAM):
   - If found → Follow `steering/diagram-from-code.md`
   - Output: `architecture-docs/architecture-actual.png`

3. Report progress:
   ```
   ✅ Step 1/4: Architecture diagrams generated
      - Identified [X] services
      - Generated [Y] diagrams
   ```

**CRITICAL:** Do NOT proceed until services are identified.

---

### Step 2: Compare Design vs Implementation (If Both Exist)

**Condition:** Only if BOTH design.md AND infrastructure code exist.

1. Follow `steering/design-vs-code.md`
2. Output: `architecture-docs/differences.md`

3. Report progress:
   ```
   ✅ Step 2/4: Design vs implementation compared
      - [X] matching services
      - [Y] differences found
   ```

**If only one exists:** Skip this step and inform user.

---

### Step 3: Well-Architected Framework Review (WAFR)

**Prerequisite:** Services identified in Step 1.

1. Follow `steering/well-architected.md`
2. Use services from Step 1
3. Output: `architecture-docs/WellArchitectedFrameworkReview.md`

4. Report progress:
   ```
   ✅ Step 3/5: Well-Architected Framework Review completed
      - Overall score: [X]/10
      - [Y] recommendations generated
   ```

---

### Step 4: Cost Analysis (Interactive)

**Prerequisite:** Services identified in Step 1.

**CRITICAL:** This step is INTERACTIVE and requires user input.

1. Inform user:
   ```
   ⏳ Step 4/5: Cost Analysis (interactive)
      
      To estimate costs, I need some usage assumptions.
      I'll ask you a few questions...
   ```

2. Follow `steering/cost-analysis.md`
3. **MUST ask for assumptions interactively**
4. Output: `architecture-docs/CostAnalysis.md`

5. Report progress:
   ```
   ✅ Step 4/5: Cost Analysis completed
      - Estimated cost: $[X]/month
   ```

---

### Step 5: Executive Summary (LAST)

**Prerequisite:** All previous steps completed.

**Purpose:** Create a concise executive summary combining all findings.

1. Follow template in `steering/executive-summary-template.md`
2. Synthesize data from:
   - Architecture diagrams (services count, components)
   - WAFR (overall score, critical findings)
   - Cost analysis (estimates, cost drivers)
   - Comparison (if available)

3. Output: `architecture-docs/ExecutiveSummary.md`

4. Report completion:
   ```
   ✅ Complete Report Generated!
   
   All documentation saved to architecture-docs/:
   - architecture.png (from design)
   - architecture-actual.png (from code)
   - differences.md (comparison)
   - WellArchitectedFrameworkReview.md (score: [X]/10)
   - CostAnalysis.md (estimated: $[Y]/month)
   - ExecutiveSummary.md (2-page summary)
   
   📊 Executive Summary highlights:
   - Overall WAFR Score: [X]/10
   - Monthly Cost: $[Y]
   - Critical Issues: [Z]
   - Key Recommendations: [W]
   ```

**CRITICAL:** Do NOT generate additional files like README.md, INDICE.md, or any other documentation. Only the 6 files listed above.

---

## Handling Multiple Cost Scenarios

**User may request additional cost analyses with different assumptions:**

```
User: "Now estimate for 20,000 users"
User: "What if we have 2,000 transactions per second?"
User: "Estimate for 200 images per day"
```

**How to handle:**

1. **Recognize it's a new scenario:**
   - User mentions different numbers/assumptions
   - Keywords: "now", "what if", "estimate for", "para [number]"

2. **Ask for ALL assumptions again:**
   ```
   "I'll create a new cost analysis for 20,000 users.
   Let me ask a few questions to complete the estimate..."
   ```

3. **Save with descriptive filename:**
   - Pattern: `CostAnalysis-[scenario].md`
   - Examples:
     - `CostAnalysis-20k-users.md`
     - `CostAnalysis-2k-tps.md`
     - `CostAnalysis-200-images-day.md`

4. **Compare with previous:**
   ```
   "Comparison with previous analysis:
   - Previous (1k users): $247.50/month
   - New (20k users): $1,850.00/month
   - Increase: 7.5x"
   ```

**Important:** Each cost scenario is independent. Always ask for complete assumptions, don't reuse previous values unless user explicitly says so.

## Usage Examples

### Natural Language Requests

The power understands natural language and will guide you through the process:

**Analyze architecture:**
```
User: "Review my AWS architecture"
User: "Revisa mi arquitectura de AWS"
Power: "I'll analyze your architecture against AWS Well-Architected Framework. 
       First, let me check what you have..."
       [Detects design.md or code]
       "I found [design.md/CDK code]. I'll analyze it and generate a Well-Architected Review."
Output: architecture-docs/WellArchitectedReview.md
```

**Estimate costs:**
```
User: "Estimate costs for my architecture"
User: "Estima los costos de mi arquitectura"
Power: "To estimate costs, I need to understand your architecture first.
       Let me check what you have..."
       [If no diagram exists]
       "I found CDK code. I'll first analyze it to identify services, then estimate costs."
       [Generates diagram first, then asks for assumptions]
Output: architecture-docs/architecture-actual.png + architecture-docs/CostAnalysis.md
```

**Complete report (recommended):**
```
User: "Give me a complete report"
User: "Dame un reporte completo"
User: "Analyze everything about my architecture"
User: "Analiza todo sobre mi arquitectura"
Power: "I'll generate a complete architecture report including:
       1. Architecture diagrams (from design and/or code)
       2. Design vs implementation comparison (if both exist)
       3. Well-Architected Review
       4. Cost analysis
       
       This will take a few moments. Let me start..."
       [Runs all capabilities in sequence]
Output: All files in architecture-docs/
```

**Compare design vs implementation:**
```
User: "Compare my architecture with what's implemented"
User: "Compara mi arquitectura con lo implementado"
User: "Check if my code matches the design"
Power: "I'll compare your design document with the infrastructure code."
       [Checks for both design.md and code]
       "Found design.md and CDK code. Comparing..."
Output: architecture-docs/differences.md
```

**Document architecture:**
```
User: "Document my AWS infrastructure"
User: "Documenta mi infraestructura de AWS"
User: "Generate architecture diagrams"
Power: "I can generate diagrams from your design document or infrastructure code.
       What would you like to use?"
User: "Use my CDK code"
Output: architecture-docs/architecture-actual.png
```

### Language Support

**The power works in multiple languages:**
- ✅ English: "Review my architecture"
- ✅ Spanish: "Revisa mi arquitectura"
- ✅ Portuguese: "Revise minha arquitetura"
- ✅ French: "Examinez mon architecture"
- ✅ German: "Überprüfen Sie meine Architektur"
- ✅ And more...

**Note:** The power understands requests in any language, but outputs (diagrams, reports) are generated in English as they follow AWS documentation standards.

### Specific Capability Requests

**Generate diagram from design:**
```
User: "Generate an architecture diagram from my design.md"
Power: "I found design.md. Analyzing services and relationships..."
Output: architecture-docs/architecture.png
```

**Generate diagram from code:**
```
User: "Analyze my CDK code and create a diagram"
User: "Diagram my Terraform infrastructure"
Power: "I found [CDK/Terraform] code. Analyzing resources..."
Output: architecture-docs/architecture-actual.png
```

**Run Well-Architected Review:**
```
User: "Run a Well-Architected review"
User: "Evaluate my architecture against AWS best practices"
Power: "I'll evaluate against the 6 pillars. Let me analyze your architecture..."
       [If no architecture analyzed yet, will analyze first]
Output: architecture-docs/WellArchitectedReview.md
```

**Estimate costs:**
```
User: "How much will this cost?"
User: "Estimate monthly costs"
Power: "I'll estimate costs. First, let me identify your services..."
       [If no architecture analyzed, will analyze first]
       "I found: Lambda, DynamoDB, S3, API Gateway.
       Now I need some usage assumptions..."
       [Asks for users, requests, storage, etc.]
Output: architecture-docs/CostAnalysis.md
```

**Complete analysis:**
```
User: "Analyze everything about my architecture"
User: "Give me a complete report"
User: "Dame un reporte completo"
Power: "I'll run all capabilities:
       1. Generate diagrams (from design and code)
       2. Compare design vs implementation
       3. Run Well-Architected Review
       4. Estimate costs
       
       This will take a few moments..."
Output: All files in architecture-docs/
```

### Language Support

The power understands requests in multiple languages:
- English, Spanish, Portuguese, French, German, and more
- Outputs are in English (following AWS documentation standards)

### Important Behaviors

**Automatic prerequisite handling:**
- If you ask for cost estimation but no architecture is analyzed, the power will analyze it first
- If you ask for Well-Architected Review but no services are identified, it will identify them first
- If you ask for comparison but only have code, it will inform you that design.md is needed

**Smart detection:**
- The power automatically detects what you have (design.md, CDK, Terraform, etc.)
- It will inform you what it found and what it can do
- It will ask for clarification if multiple options are available

**Conversational flow:**
- You don't need to know exact commands
- The power understands intent and guides you
- It will ask for missing information when needed

## Important Notes

- **Diagrams**: Currently generates PNG only. Draw.io export planned for v1.1
- **Cost Analysis**: Requires AWS credentials. Other capabilities work without credentials
- **Pricing Data**: All pricing API calls are free of charge
- **Accuracy**: Cost estimates are based on assumptions. Always validate with AWS Pricing Calculator for production planning

## Troubleshooting

**"GraphViz not found" error:**
- Install GraphViz: https://www.graphviz.org/download/
- Restart Kiro after installation

**"AWS credentials not configured" error (Cost Analysis only):**
- Run `aws configure` to set up credentials
- Or set environment variables: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_REGION
- Ensure IAM user/role has `pricing:*` permissions

**MCP server not responding:**
- Check `~/.kiro/settings/mcp.json` configuration
- Ensure `uv` is installed and in PATH
- Check Kiro logs for detailed error messages

## Limitations

- Diagram generation: PNG only (draw.io planned for v1.1)
- IaC support: CDK, Terraform, CloudFormation, SAM (others may work with limited support)
- Cost estimates: Based on assumptions, not guaranteed to be exact
- Well-Architected Review: Automated analysis, not a replacement for manual review

## Learn More

- AWS Well-Architected Framework: https://aws.amazon.com/architecture/well-architected/
- AWS Pricing: https://aws.amazon.com/pricing/
- Python Diagrams Package: https://diagrams.mingrammer.com/
