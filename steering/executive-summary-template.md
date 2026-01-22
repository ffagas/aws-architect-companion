# Executive Summary Template

Use this template when generating the Executive Summary for the complete report.

## File Information
- **Filename:** `ExecutiveSummary.md`
- **Location:** `architecture-docs/`
- **When to generate:** Only when user requests "complete report" or explicitly asks for executive summary

## Template Structure

```markdown
# 📊 Executive Summary - [Project Name]

**Date**: [Current Date]  
**Analysis**: [Architecture Type - e.g., "AWS Serverless Architecture", "Microservices on EKS"]

---

## 🎯 Key Findings

[2-3 sentence summary of the most important findings. Include overall WAFR score and critical recommendations.]

---

## 📈 Key Metrics

| Category | Metric | Value |
|----------|--------|-------|
| **Architecture** | AWS Services | [X] |
| **Architecture** | [Key Component] | [Count] |
| **Quality** | WAFR Score | [X.X]/10 [🟢/🟡/🔴] |
| **Costs** | Monthly (Low Usage) | $[X] USD |
| **Costs** | Monthly (Medium Usage) | $[X] USD |
| **Costs** | Monthly (High Usage) | $[X] USD |
| **Performance** | [Key Metric] | [Value] |
| **Availability** | SLA | [X]% |

---

## 🏗️ Technology Stack

### [Layer 1 - e.g., Frontend]
```
[Technology Stack]
↓
[AWS Service]
↓
[Distribution/CDN]
```

### [Layer 2 - e.g., Backend]
```
[Entry Point]
↓
[Compute]
↓
[Data/Storage/AI Services]
```

### Infrastructure
- **IaC**: [CDK/Terraform/CloudFormation]
- **Language**: [TypeScript/Python/etc.]
- **Deployment**: [Method]

---

## ⚠️ Critical Findings

### 🔴 High Priority
1. **[Issue Title]**
   - Impact: [Description]
   - Recommendation: [Action]
   - Effort: [Low/Medium/High]

### 🟡 Medium Priority
1. **[Issue Title]**
   - Impact: [Description]
   - Recommendation: [Action]
   - Effort: [Low/Medium/High]

---

## ✅ Strengths

1. **[Strength Title]**: [Description]
2. **[Strength Title]**: [Description]
3. **[Strength Title]**: [Description]

---

## 💰 Cost Analysis Summary

### Current Estimate
- **Low Usage**: $[X]/month - [Description of scenario]
- **Medium Usage**: $[X]/month - [Description of scenario]
- **High Usage**: $[X]/month - [Description of scenario]

### Cost Drivers
1. **[Service]** ([X]%) - $[X]/month
2. **[Service]** ([X]%) - $[X]/month
3. **[Service]** ([X]%) - $[X]/month

### Optimization Opportunities
- 💡 [Recommendation 1]: Potential savings $[X]/month
- 💡 [Recommendation 2]: Potential savings $[X]/month

---

## 🎯 Well-Architected Framework Review

| Pillar | Score | Status | Key Findings |
|--------|-------|--------|--------------|
| **Operational Excellence** | [X]/10 | [🟢/🟡/🔴] | [1-2 sentence summary] |
| **Security** | [X]/10 | [🟢/🟡/🔴] | [1-2 sentence summary] |
| **Reliability** | [X]/10 | [🟢/🟡/🔴] | [1-2 sentence summary] |
| **Performance Efficiency** | [X]/10 | [🟢/🟡/🔴] | [1-2 sentence summary] |
| **Cost Optimization** | [X]/10 | [🟢/🟡/🔴] | [1-2 sentence summary] |
| **Sustainability** | [X]/10 | [🟢/🟡/🔴] | [1-2 sentence summary] |

**Overall Score**: [X.X]/10 [🟢/🟡/🔴]

---

## 📋 Recommended Actions

### Immediate (Before Production)
1. [ ] [Action with specific steps]
2. [ ] [Action with specific steps]

### Short Term (1-3 months)
1. [ ] [Action with specific steps]
2. [ ] [Action with specific steps]

### Long Term (3-6 months)
1. [ ] [Action with specific steps]
2. [ ] [Action with specific steps]

---

## 📚 Documentation Generated

- ✅ `architecture.png` - Architecture diagram from design
- ✅ `architecture-actual.png` - Architecture diagram from code
- ✅ `differences.md` - Design vs implementation comparison
- ✅ `WellArchitectedFrameworkReview.md` - Detailed WAFR analysis
- ✅ `CostAnalysis.md` - Detailed cost breakdown
- ✅ `ExecutiveSummary.md` - This document

---

**Next Steps**: Review critical findings and implement high-priority recommendations before production deployment.
```

## Guidelines for Generation

1. **Be Concise**: Executive summary should be 2-3 pages max
2. **Use Emojis**: Makes it scannable (🎯 🔴 🟡 🟢 ✅ ⚠️ 💰 📊)
3. **Quantify Everything**: Use numbers, percentages, scores
4. **Prioritize**: Show what matters most first
5. **Actionable**: Every finding should have a clear recommendation
6. **Visual**: Use tables, code blocks, and formatting
7. **Status Indicators**:
   - 🟢 Good (8-10)
   - 🟡 Needs Improvement (5-7)
   - 🔴 Critical (0-4)

## When NOT to Generate

- User only asks for specific capability (diagram, WAFR, cost)
- User explicitly says "no summary" or "just the analysis"
- Incomplete data (missing WAFR or cost analysis)
