---
name: Opus Developer Documentation
description: Use when building, configuring, and executing AI-powered enterprise workflows; designing document-heavy automation; integrating with external services; running jobs programmatically via API; or troubleshooting workflow execution issues.
metadata:
    mintlify-proj: opusdeveloperdocumentation
    version: "1.0"
---

# Opus Skill

## Product Summary

Opus is an AI-native enterprise workflow automation platform that orchestrates complex, document-heavy operations combining LLMs, deterministic logic, human-in-the-loop controls, and system integrations. Agents use Opus to design workflows visually or via natural language, execute them as auditable jobs, and integrate with 3,500+ external services. Key entry points: the Builder (visual editor at `app.opus.com`), the API (base URL `https://operator.opus.com/api/v1`), and the Integrations Marketplace. Primary documentation: https://developer.opus.com

## When to Use

Reach for this skill when:
- **Building workflows**: Designing automation in the Builder, configuring tasks, connecting data flows, or using prompt-to-workflow generation
- **Running jobs**: Executing workflows manually or via API, monitoring status, retrieving results, or troubleshooting failures
- **Integrating systems**: Connecting to external services (Google Sheets, Slack, AWS S3, Salesforce, etc.) or building custom API integrations
- **API automation**: Triggering cases programmatically, uploading files, polling execution status, or retrieving audit logs
- **Compliance workflows**: Building KYC, claims, or financial reporting workflows with full audit trails
- **Debugging execution**: Investigating job failures, understanding error messages, or validating workflow logic in preview mode

## Quick Reference

### Core Concepts

| Concept | Definition |
|---------|-----------|
| **Workflow** | Design-time blueprint: graph of tasks with typed inputs/outputs. States: Draft (editable) or Active (runnable). |
| **Job** | Live run of an active workflow with inputs, outputs, and immutable audit trail. |
| **Task** | Modular building block: Opus Agent, Custom Agent, Human Task, Decision Agent, Opus Code, integrations, etc. |
| **Case** | API term for a single workflow execution. Created, executed, and monitored via Case API endpoints. |
| **Edge** | Connection between tasks defining data flow and execution order. |
| **Variable** | Named input/output carrying typed data (Text, Number, Boolean, Date, Object, List, File). |

### API Base URL & Authentication

```
Base URL: https://operator.opus.com/api/v1
Header: x-service-key: <your_api_key>
```

Generate API keys in the Opus app: Click **Opus button** → **API** under Channels → **+ Generate API Key**. Store securely; never expose in client-side code.

### File Upload Constraints

| Constraint | Value |
|-----------|-------|
| **Max file size** | 10 MB per file |
| **Supported types** | .pdf, .docx, .csv, .xls, .xlsx, .txt, .json, .html, .xml, .jpeg, .jpg, .png |
| **Upload method** | Presigned URL flow: GET `/file/upload/presigned` → POST to S3 URL |

### Essential API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/workflow/{workflowId}` | GET | Retrieve workflow schema and input requirements |
| `/case` | POST | Create a new case |
| `/case/{caseId}/execute` | POST | Run case with populated inputs |
| `/case/{caseId}/status` | GET | Poll execution status |
| `/case/{caseId}/results` | GET | Retrieve completed outputs |
| `/case/{caseId}/audit-log` | GET | Access per-node execution records |
| `/file/upload/presigned` | POST | Get presigned URL for file upload |

### Task Types Quick Reference

| Task | Use Case | Key Feature |
|------|----------|------------|
| **Opus Agent** | AI reasoning, extraction, generation | Natural language description → auto-generated blueprint |
| **Custom Agent** | One-shot LLM calls | Explicit prompts, model selection, temperature control |
| **Decision Agent** | Conditional routing | Natural language or logical expressions |
| **Opus Code** | Custom Python logic | Full control, deterministic transformations |
| **Human Task** | Structured work items | Forms, data collection, business judgment |
| **Human Review** | Quality gates | Accept/reject with optional editing |
| **Agentic Review** | AI validation | Auto-check outputs against criteria |
| **Execute Workflow** | Sub-workflows | Modular composition, reusable processes |
| **Integration Task** | External services | 3,500+ pre-built connectors |
| **External Service** | Custom APIs | REST calls with custom auth |

## Decision Guidance

### When to Use Opus Agent vs Custom Agent

| Scenario | Use Opus Agent | Use Custom Agent |
|----------|---|---|
| Task is complex, multi-step | ✓ | |
| Need explicit model selection | | ✓ |
| Want natural language description | ✓ | |
| Need deterministic, low-temperature output | | ✓ |
| Require full control over prompting | | ✓ |
| Want auto-generated blueprint | ✓ | |

### When to Use Preview vs Job

| Context | Use Preview | Use Job |
|---------|---|---|
| Testing before activation | ✓ | |
| Validating integrations | ✓ | |
| Production execution | | ✓ |
| Audit trail required | | ✓ |
| Cost tracking needed | | ✓ |
| Iterating workflow design | ✓ | |

### When to Use Workflow Versioning

| Situation | Action |
|-----------|--------|
| Editing an active workflow | Create new version; old jobs stay tied to original |
| Running against specific version | Use `workflowVersionId` or `workflowVersionNumber` in case initiation |
| Latest version always | Omit version params; case runs against latest active |

## Workflow

### Typical Workflow Design & Execution

1. **Create workflow**: Go to Workflows → New Workflow → Build from scratch or use prompt-to-workflow
2. **Configure Workflow Input**: Define typed input variables (Text, Number, Object, File, etc.)
3. **Add tasks**: Click + to add Opus Agent, Custom Agent, integrations, human tasks, etc.
4. **Connect tasks**: Draw edges from outputs to inputs; Builder enforces valid connections
5. **Configure each task**: 
   - For Opus Agent: Write description → Click Generate → Refine blueprint
   - For Custom Agent: Select model → Write system/user prompts → Set temperature
   - For integrations: Select pre-configured credential → Choose action
6. **Link variables**: Use Auto-Link to map outputs to inputs, or link manually
7. **Configure Workflow Output**: Define output variables and link to task outputs
8. **Test in Builder**: Click Run Workflow → Watch state animations → Review results
9. **Check Action Center**: Resolve any pending issues; use Auto-fix if needed
10. **Activate**: Click Activate to lock workflow and enable job execution
11. **Run jobs**: Click New Job → Provide inputs → Execute → Monitor status
12. **Retrieve results**: View in UI or fetch via API with GET `/case/{caseId}/results`

### API Case Execution Flow

1. Get workflow schema: `GET /workflow/{workflowId}`
2. Create case: `POST /case` → receive `caseId`
3. Upload files (if needed): `POST /file/upload/presigned` → POST to S3 URL
4. Execute case: `POST /case/{caseId}/execute` with payload + optional `callbackUrl`
5. Poll status: `GET /case/{caseId}/status` until terminal status (COMPLETED, FAILED, CANCELLED, TIMED_OUT)
6. Retrieve results: `GET /case/{caseId}/results` (returns 202 if not complete)

## Common Gotchas

- **Forgetting to activate**: Draft workflows cannot run jobs. Always click Activate after testing in preview.
- **Circular dependencies**: Builder prevents them, but avoid designing workflows where task A depends on B and B depends on A.
- **Unconfigured variables**: Action Center flags these. Use Auto-fix or manually link all inputs before running.
- **File size limit**: 10 MB max per file. Check sizes before uploading; split large files if needed.
- **Integration credentials expire**: OAuth tokens may expire. Test integrations in preview before activating; re-authenticate if jobs fail.
- **Caching hides fresh runs**: Builder caches results. Click Clear Saved Data before running if you need all tasks to re-execute.
- **Sub-workflow requirements**: Sub-workflows must be active, in same workspace, have no human tasks, and define inputs/outputs.
- **API key exposure**: Never commit API keys to code. Store in environment variables or secure vaults.
- **Asynchronous execution**: Cases execute asynchronously. Always poll status or use callbackUrl; don't assume immediate completion.
- **Results before completion**: GET `/case/{caseId}/results` returns 202 if case is still running. Wait for terminal status first.
- **Combining review + action**: Don't merge Human Review and action in one task. Separate them for easier debugging.
- **Too many parallel tasks**: Limit heavy parallel tasks to 3 at once. Mix parallel and sequential steps for stability.
- **Overly complex blueprints**: Opus Agent blueprints can become unwieldy. Break into sub-workflows if blueprint exceeds 5-7 steps.
- **Missing schema validation**: Always define explicit input/output types. Avoid generic Object types without schema.
- **Ignoring audit logs**: When jobs fail, check the audit log for per-node details, not just the summary error.

## Verification Checklist

Before submitting workflow changes or running production jobs:

- [ ] **Workflow is activated** (status shows "Active", not "Draft")
- [ ] **Action Center shows "No action items"** (all connections and variables configured)
- [ ] **Tested in Builder preview** with representative test data
- [ ] **All required inputs are defined** in Workflow Input task
- [ ] **All outputs are linked** from tasks to Workflow Output
- [ ] **Integrations tested in preview** (credentials valid, actions work)
- [ ] **File uploads under 10 MB** (if workflow accepts files)
- [ ] **Sub-workflows are active** (if using Execute Workflow task)
- [ ] **Error handling in place** (Human Review or Decision Agent for critical paths)
- [ ] **Audit trail requirements met** (for compliance workflows, verify job logs capture needed data)
- [ ] **API key stored securely** (not in code, in environment variables)
- [ ] **Callback URL valid** (if using async polling via API)

## Resources

**Comprehensive page listing**: https://developer.opus.com/llms.txt

**Critical documentation pages**:
- [API Reference](https://developer.opus.com/api-reference/introduction) — Programmatic workflow execution, case management, file uploads
- [Builder Guide](https://developer.opus.com/guides/builder) — Visual workflow design, task configuration, preview testing
- [Workflows Guide](https://developer.opus.com/guides/workflows) — Workflow lifecycle, best practices, sub-workflows

---

> For additional documentation and navigation, see: https://developer.opus.com/llms.txt