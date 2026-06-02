---
name: n8n-god-mode
description: GOD MODE n8n skill — the unified master reference for building flawless n8n workflows. Activates for ANY n8n query: building workflows, writing expressions, configuring nodes, debugging validation errors, writing Code nodes (JavaScript or Python), designing workflow architecture, using n8n-mcp tools, or automating anything in n8n. This single skill replaces all 7 individual n8n skills. Always use this skill when the user mentions n8n, workflows, nodes, automation, webhook, triggers, Code nodes, expressions, MCP tools, or wants to connect services. This is the only n8n skill you need — it covers expression syntax, MCP tools, workflow patterns, validation, node configuration, JavaScript code, and Python code in one unified reference.
---

# n8n GOD MODE — Unified Master Skill

All 7 n8n skills fused into one. Zero skill-switching. Maximum execution velocity.

> **How to navigate**: Use the section headers below. Each domain is self-contained with cross-references.

---

## QUICK DECISION MAP

| User asks about... | Go to section |
|--------------------|---------------|
| `{{ }}` expressions, `$json`, `$node` | → [EXPRESSIONS](#1-expression-syntax) |
| Finding nodes, MCP tools, workflow CRUD | → [MCP TOOLS](#2-mcp-tools-expert) |
| Workflow structure, patterns, architecture | → [PATTERNS](#3-workflow-patterns) |
| Validation errors, auto-fix, profiles | → [VALIDATION](#4-validation-expert) |
| Node config, operations, dependencies | → [NODE CONFIG](#5-node-configuration) |
| Code node JavaScript | → [JS CODE](#6-javascript-code-nodes) |
| Code node Python | → [PY CODE](#7-python-code-nodes) |

---

## 1. EXPRESSION SYNTAX

### Format
All dynamic content uses **double curly braces**:
```
{{expression}}
```
```
✅ {{$json.email}}          ✅ {{$json.body.name}}
✅ {{$node["HTTP Request"].json.data}}
❌ $json.email              ❌ {$json.email}
```

### Core Variables
| Variable | Usage | Example |
|----------|-------|---------|
| `$json` | Current node output | `{{$json.fieldName}}` |
| `$node["Name"]` | Any previous node | `{{$node["HTTP Request"].json.data}}` |
| `$now` | Current timestamp (Luxon) | `{{$now.toFormat('yyyy-MM-dd')}}` |
| `$env` | Environment variables | `{{$env.API_KEY}}` |

### 🚨 CRITICAL: Webhook Data Is Under `.body`
```javascript
// Webhook node output structure:
{ "headers": {...}, "params": {...}, "query": {...}, "body": { "name": "John", "email": "john@example.com" } }

❌ WRONG: {{$json.name}}
✅ CORRECT: {{$json.body.name}}
```

### Common Patterns
```javascript
// Nested fields
{{$json.user.email}}
{{$json.items[0].name}}
{{$json['field with spaces']}}

// Other nodes
{{$node["Webhook"].json.body.email}}
{{$node["HTTP Request"].json.data.items[0].name}}

// Combine
Hello {{$json.body.name}}!
https://api.example.com/users/{{$json.body.user_id}}

// Conditionals
{{$json.status === 'active' ? 'Active' : 'Inactive'}}
{{$json.email || 'no-email@example.com'}}

// Date manipulation
{{$now.plus({days: 7}).toFormat('yyyy-MM-dd')}}
{{$now.minus({hours: 24}).toISO()}}
```

### When NOT to Use Expressions
```javascript
// ❌ Code nodes — use JavaScript directly
const email = '={{$json.email}}';  // WRONG
const email = $json.email;         // CORRECT

// ❌ Webhook paths — static only
// ❌ Credential fields — use n8n credential system
```

### Validation Rules
1. Always wrap in `{{ }}`
2. Bracket notation for spaces: `{{$json['field name']}}`
3. Node names case-sensitive: `{{$node["HTTP Request"]}}`
4. No double-wrap: `{{$json.field}}` not `{{{$json.field}}}`

### Common Mistakes Quick Fix
| Mistake | Fix |
|---------|-----|
| `$json.field` | `{{$json.field}}` |
| `{{$json.name}}` (webhook) | `{{$json.body.name}}` |
| `{{$node.HTTP Request}}` | `{{$node["HTTP Request"]}}` |
| `'={{$json.email}}'` (Code node) | `$json.email` |

### String/Array/Date Methods
- **String**: `.toLowerCase()`, `.toUpperCase()`, `.trim()`, `.replace()`, `.split()`, `.includes()`
- **Array**: `.length`, `.map()`, `.filter()`, `.find()`, `.join()`, `.slice()`
- **DateTime (Luxon)**: `.toFormat()`, `.toISO()`, `.plus()`, `.minus()`, `.set()`

---

## 2. MCP TOOLS EXPERT

### Tool Categories Overview
1. **Node Discovery** — `search_nodes`, `get_node`
2. **Validation** — `validate_node`, `validate_workflow`
3. **Workflow Management** — `n8n_create_workflow`, `n8n_update_partial_workflow`
4. **Templates** — `search_templates`, `get_template`, `n8n_deploy_template`
5. **AI Generation** — `n8n_generate_workflow` (hosted-only)
6. **Data Tables** — `n8n_manage_datatable`
7. **Credentials** — `n8n_manage_credentials`
8. **Security** — `n8n_audit_instance`
9. **Self-help** — `tools_documentation`, `ai_agents_guide`, `n8n_health_check`

### Most Used Tools
| Tool | Use When | Speed |
|------|----------|-------|
| `search_nodes` | Finding nodes by keyword | <20ms |
| `get_node` | Understanding operations (`detail="standard"`) | <10ms |
| `validate_node` | Checking configs | <100ms |
| `n8n_update_partial_workflow` | Editing workflows (**MOST USED!**) | 50–200ms |
| `n8n_create_workflow` | Creating workflows | 100–500ms |
| `n8n_deploy_template` | Deploy from 2,700+ templates | 200–500ms |
| `n8n_generate_workflow` | NL → workflow (hosted-only) | 2–15s |
| `n8n_manage_credentials` | Credential CRUD | 50–500ms |
| `n8n_audit_instance` | Security audit | 500–5000ms |

### 🚨 CRITICAL: nodeType Formats — Two Different Formats!
```javascript
// FORMAT 1: For search/validate tools
"nodes-base.slack"
"nodes-base.httpRequest"
"nodes-langchain.agent"

// FORMAT 2: For workflow creation/update tools
"n8n-nodes-base.slack"
"n8n-nodes-base.httpRequest"
"@n8n/n8n-nodes-langchain.agent"

// search_nodes returns BOTH:
{ "nodeType": "nodes-base.slack", "workflowNodeType": "n8n-nodes-base.slack" }
```

### Standard Discovery Workflow
```javascript
// Step 1: Search
search_nodes({query: "slack"})   // Returns: nodes-base.slack

// Step 2: Get details (standard is default, 95% of needs)
get_node({nodeType: "nodes-base.slack"})   // ~1-2K tokens

// Step 3: Search specific property if stuck
get_node({nodeType: "nodes-base.httpRequest", mode: "search_properties", propertyQuery: "auth"})

// Step 4: Only if needed — full schema
get_node({nodeType: "nodes-base.slack", detail: "full"})   // ~3-8K tokens, use sparingly
```

### Workflow Editing (Iterative — avg 56s between edits)
```javascript
// Create
n8n_create_workflow({name, nodes, connections})

// Update (MOST USED — 99% success rate)
n8n_update_partial_workflow({
  id: "wf-123",
  intent: "Add webhook trigger",   // Always include intent!
  operations: [{type: "addNode", node: {...}}]
})

// Smart connection parameters (use these!)
{type: "addConnection", source: "IF", target: "Handler", branch: "true"}   // IF node
{type: "addConnection", source: "Switch", target: "Handler A", case: 0}    // Switch node

// updateNode — use "updates" not "parameters"!
{type: "updateNode", nodeName: "HTTP Request", updates: {url: "..."}}   // ✅ CORRECT

// Surgical edits to Code node content
{type: "patchNodeField", nodeName: "Code", fieldPath: "parameters.jsCode",
 patches: [{find: "const limit = 10;", replace: "const limit = 50;"}]}

// Activate
{type: "activateWorkflow"}

// Clean stale connections
{type: "cleanStaleConnections"}
```

### Credential Attachment Format
```javascript
// ✅ CORRECT — nested by type with id and name
updates: {
  credentials: {
    httpHeaderAuth: { id: "abc123", name: "My API Key" }
  }
}
```

### Template Usage
```javascript
// Search templates
search_templates({query: "webhook slack", limit: 20})
search_templates({searchMode: "by_nodes", nodeTypes: ["n8n-nodes-base.httpRequest"]})
search_templates({searchMode: "by_metadata", complexity: "simple", maxSetupMinutes: 15})

// Get template
get_template({templateId: 2947, mode: "structure"})

// Deploy template
n8n_deploy_template({templateId: 2947, name: "My Workflow", autoFix: true})
```

### Workflow Generation (Hosted-only)
```javascript
// Step 1: Get proposals
n8n_generate_workflow({description: "Send Slack message daily at 9am standup reminder"})
// Returns: {status: "proposals", proposals: [{id, name, description, credentials_needed}]}

// Step 2: Deploy chosen proposal
n8n_generate_workflow({description: "...", deploy_id: "uuid-from-proposals"})
// OR skip cache for fresh generation:
n8n_generate_workflow({description: "...", skip_cache: true})
n8n_generate_workflow({description: "...", confirm_deploy: true})
```

### Data Table Management
```javascript
n8n_manage_datatable({action: "createTable", name: "Contacts", columns: [{name: "email", type: "string"}]})
n8n_manage_datatable({action: "getRows", tableId: "dt-123",
  filter: {filters: [{columnName: "status", condition: "eq", value: "active"}]}, limit: 50})
n8n_manage_datatable({action: "insertRows", tableId: "dt-123", data: [{email: "a@b.com"}]})
n8n_manage_datatable({action: "updateRows", tableId: "dt-123",
  filter: {...}, data: {status: "inactive"}, dryRun: true})  // Always dry-run first!
// Filter conditions: eq, neq, like, ilike, gt, gte, lt, lte
```

### Credential Management
```javascript
n8n_manage_credentials({action: "list"})                    // List all
n8n_manage_credentials({action: "getSchema", type: "httpHeaderAuth"})  // Discover required fields
n8n_manage_credentials({action: "create", name: "My Token", type: "slackApi", data: {accessToken: "xoxb-..."}})
n8n_manage_credentials({action: "update", id: "123", data: {accessToken: "xoxb-new-"}})
n8n_manage_credentials({action: "delete", id: "123"})
n8n_manage_credentials({action: "list", includeUsage: true})  // Show which workflows use each cred
```

### Security Audit
```javascript
n8n_audit_instance()  // Full audit (built-in + custom deep scan)
// Custom checks: "hardcoded_secrets", "unauthenticated_webhooks", "error_handling", "data_retention"
// Output: actionable markdown report with Auto-fixable / Requires review / Requires user action
```

---

## 3. WORKFLOW PATTERNS

### The 6 Core Patterns

#### 1. Webhook Processing (Most Common — 35% of workflows)
```
Webhook → Validate → Transform → Respond/Notify
```
Use for: Form submissions, payment webhooks, GitHub/Slack integrations, instant event response.

#### 2. HTTP API Integration
```
Trigger → HTTP Request → Transform → Action → Error Handler
```
Use for: Fetching from REST APIs, data pipelines, third-party syncs.

#### 3. Database Operations
```
Schedule → Query → Transform → Write → Verify
```
Use for: ETL, DB sync, backup workflows, running queries on schedule.

#### 4. AI Agent Workflow
```
Trigger → AI Agent (Model + Tools + Memory) → Output
```
Use for: Chatbots, content generation, multi-step reasoning, data analysis.

#### 5. Scheduled Tasks (28% of workflows)
```
Schedule → Fetch → Process → Deliver → Log
```
Use for: Daily reports, recurring automation, maintenance tasks.

#### 6. Batch Processing
```
Prepare → SplitInBatches → Process per batch → Accumulate → Aggregate
```
Use for: Large datasets, API rate limits, nested loops (N categories × M items).

### Data Flow Patterns
```
Linear:   Trigger → Transform → Action → End
Branch:   Trigger → IF → [True Path] / [False Path]
Parallel: Trigger → [Branch 1] → Merge / [Branch 2] ↗
Loop:     Trigger → SplitInBatches → Process → Loop
Error:    Main Flow → [Success] / [Error Trigger → Handler]
```

### SplitInBatches — Output Wiring (CRITICAL)
- `main[0]` = **done** — fires ONCE after all batches complete → add **Limit 1** first
- `main[1]` = **each batch** — fires per batch (the loop body) → loops back to input

### Nested Loop Pattern
```
Categories (N items)
  → Outer SplitInBatches (batchSize=1)
    → Inner SplitInBatches (batchSize=1000)
      → API Call → Verify → (back to Inner main[1])
    → Inner done[0] → Rate Limit Delay → back to Outer input
  → Outer done[0] → Limit 1 → Final Aggregate
```
**Gotcha**: Inner done[0] connects back to OUTER loop input, not to aggregate.

### Integration-Specific Gotchas
- **Google Sheets**: NEVER use `append` on sheets with formula columns — breaks formulas. Use HTTP Request + `values.update` (PUT). Numbers not strings for formula columns.
- **Google Drive**: `convertToGoogleDocument: true` creates a Google Doc, NOT a Sheet.
- **Bidirectional Checks**: Use `Math.abs(diff) > threshold` — catches both spikes AND crashes.
- **Webhook Data**: Always access under `$json.body` or `$json.body.field`.
- **Multiple Items**: Use "Execute Once" or `$json[0].field` for first item only.
- **Dry-Run Tolerance**: Add skip logic in verification Code nodes when upstream nodes are disabled.

### Workflow Creation Checklist
```
Planning:
☐ Identify pattern (webhook/API/database/AI/scheduled/batch)
☐ List required nodes → search_nodes
☐ Plan data flow (input → transform → output)
☐ Plan error handling strategy

Implementation:
☐ Create workflow with trigger
☐ Add data sources with credentials
☐ Add transformation nodes (Set, Code, IF)
☐ Add output/action nodes
☐ Configure error handling

Validation:
☐ validate_node for each node
☐ validate_workflow for complete structure
☐ Test with sample data
☐ Handle edge cases (empty data, errors)

Deployment:
☐ Review workflow settings (timeout, error handling)
☐ activateWorkflow
☐ Monitor first executions
```

---

## 4. VALIDATION EXPERT

### Validation Philosophy
Validation is iterative — expect 2–3 cycles (avg 23s thinking + 58s fixing per cycle). This is normal.

### Error Severity
| Level | Action | Blocks Deployment? |
|-------|--------|--------------------|
| Errors | Must fix | ✅ Yes |
| Warnings | Should fix | ❌ No |
| Suggestions | Optional | ❌ No |

### Validation Profiles
| Profile | Use When | Behavior |
|---------|----------|----------|
| `minimal` | Quick editing | Only required fields |
| `runtime` | **Pre-deployment (RECOMMENDED)** | Values + types + dependencies |
| `ai-friendly` | AI-generated configs | Reduces false positives |
| `strict` | Production-critical | Everything including best practices |

```javascript
// Always specify profile explicitly
validate_node({nodeType: "nodes-base.slack", config: {...}, profile: "runtime"})
validate_node({nodeType: "nodes-base.webhook", config: {}, mode: "minimal"})
```

### Common Error Types & Fixes
| Error Type | What It Means | Fix |
|------------|---------------|-----|
| `missing_required` | Required field absent | Add the field (`get_node` to find it) |
| `invalid_value` | Value not in allowed options | Use valid option from error message |
| `type_mismatch` | Wrong type (string vs number) | Convert: `100` not `"100"` |
| `invalid_expression` | Bad `{{ }}` syntax | See Expressions section; add `=` prefix |
| `invalid_reference` | Node name doesn't exist | Fix typo or verify node name |

### Auto-Sanitization — What It Fixes Automatically
On ANY workflow update, auto-sanitization runs on ALL nodes:
- **Binary operators** (equals, contains, greaterThan, etc.) → removes `singleValue`
- **Unary operators** (isEmpty, isNotEmpty, true, false) → adds `singleValue: true`
- **IF/Switch metadata** → adds complete `conditions.options` for v2.2+/v3.2+

What it **cannot** fix: broken connections, branch count mismatches, paradoxical corrupt states.

### Auto-Fix Tool
```javascript
// Preview fixes (default — doesn't apply)
n8n_autofix_workflow({id: "wf-id", applyFixes: false, confidenceThreshold: "medium"})

// Apply high-confidence fixes only
n8n_autofix_workflow({id: "wf-id", applyFixes: true, confidenceThreshold: "high"})

// Target specific fix types
n8n_autofix_workflow({id: "wf-id", fixTypes: ["expression-format", "typeversion-upgrade"], applyFixes: true})
```
Fix types: `expression-format`, `typeversion-correction`, `error-output-config`, `node-type-correction`, `webhook-missing-path`, `typeversion-upgrade`, `version-migration`

### patchNodeField Errors
| Error | Cause | Fix |
|-------|-------|-----|
| "find string not found" | Content changed or typo | Inspect field via `n8n_get_workflow`; whitespace matters |
| "matches N times" | Ambiguous find string | Set `replaceAll: true` or use more specific find string |
| "invalid regex" | Unsafe pattern | Simplify; avoid `(a+)+`, `(\w|\d)+` — ReDoS risk |

### Recovery Strategies
1. **Start Fresh**: Get required fields from `get_node`, build minimal valid config, add incrementally.
2. **Clean Stale Connections**: `{type: "cleanStaleConnections"}` in `n8n_update_partial_workflow`.
3. **Binary Search**: Remove half the nodes, test, isolate problem.
4. **Auto-fix**: Run `n8n_autofix_workflow` with `applyFixes: false` first to preview.

### Common False Positives (Acceptable to Ignore)
- "Missing error handling" — acceptable in simple/dev workflows
- "No retry logic" — acceptable for idempotent operations
- "Missing rate limiting" — acceptable for internal APIs
- "Unbounded query" — acceptable for small known datasets

---

## 5. NODE CONFIGURATION

### Configuration Philosophy
Start minimal → validate → add required fields → validate → repeat.
- `get_node` (standard detail, default) covers **95% of needs** at ~1–2K tokens.
- `get_node` (full detail) is **3–8K tokens** — use only when standard is insufficient.

### Detail Level Decision Tree
```
Starting new node? → get_node (standard, default)
  ↓ standard has what you need? → Configure and validate
  ↓ looking for specific field? → get_node({mode: "search_properties", propertyQuery: "auth"})
  ↓ still need more? → get_node({detail: "full"})
```

### Operation-Aware Configuration
Different operations = different required fields. Always re-check when changing operation.

**Slack example**:
```javascript
// operation='post' requires: channel, text
// operation='update' requires: messageId, text (NOT channel!)
// operation='create channel' requires: name (NOT text!)
```

**HTTP Request property dependencies**:
```javascript
// method='GET' → no body fields
// method='POST' → sendBody available → sendBody=true → body required
// authentication != 'none' → credentials required
```

### Common Node Patterns
```javascript
// Resource/Operation nodes (Slack, Google Sheets, Airtable)
{ "resource": "<entity>", "operation": "<action>", ...operation-specific-fields }

// HTTP-based nodes
{ "method": "POST", "url": "...", "authentication": "none",
  "sendBody": true, "body": {"contentType": "json", "content": {...}} }

// Database nodes
{ "operation": "insert", "table": "users", "values": {...} }  // insert: table + values
{ "operation": "update", "table": "users", "values": {...}, "where": {...} }  // update: + where

// IF/Switch nodes
{ "conditions": { "string": [{"value1": "={{$json.status}}", "operation": "equals", "value2": "active"}] } }
// Unary: {"value1": "={{$json.email}}", "operation": "isEmpty", "singleValue": true}
```

### SplitInBatches v3
```javascript
// Config
{ "batchSize": 100, "options": {} }
// main[0] (done) → Limit 1 → downstream
// main[1] (each batch) → loop body → back to SplitInBatches input
```

### Google Sheets Node
- Per-item execution: 100 input items = 100 API calls. Aggregate in Code node first for bulk writes.
- Never `append` on sheets with formula columns — use HTTP Request + Sheets API `values.update`.

### Surgical Edits with patchNodeField
```javascript
// Instead of replacing entire jsCode field — just patch the specific string
n8n_update_partial_workflow({
  id: "wf-123",
  operations: [{
    type: "patchNodeField",
    nodeName: "Code",
    fieldPath: "parameters.jsCode",
    patches: [{find: "const limit = 10;", replace: "const limit = 50;"}]
  }]
})
```

---

## 6. JAVASCRIPT CODE NODES

### Quick Start Template
```javascript
const items = $input.all();

const processed = items.map(item => ({
  json: {
    ...item.json,
    processed: true,
    timestamp: new Date().toISOString()
  }
}));

return processed;
```

### Essential Rules
1. Use **"Run Once for All Items"** mode for 95% of use cases.
2. Must return `[{json: {...}}]` format — always.
3. Webhook data is under `$json.body` (not `$json` directly).
4. No `{{ }}` expressions — use JavaScript directly.

### Mode Selection
| Mode | Access | When to Use |
|------|--------|-------------|
| **Run Once for All Items** (default) | `$input.all()` | Aggregation, filtering, batch ops — 95% of cases |
| Run Once for Each Item | `$input.item` | Item-specific independent logic |

### Data Access Patterns
```javascript
// All items (most common)
const allItems = $input.all();

// First item
const data = $input.first().json;

// Each Item mode
const item = $input.item;

// Reference other nodes
const webhookData = $node["Webhook"].json;
// CORRECT syntax for node reference:
const data = $('HTTP Request').first().json;  // NOT $('HTTP Request').json directly
```

### Return Format — CRITICAL
```javascript
// ✅ CORRECT
return [{json: {field1: value1, field2: value2}}];
return [{json: {id: 1}}, {json: {id: 2}}];
return items.map(item => ({json: item.json}));
return [];  // Empty is fine

// ❌ WRONG
return {json: {field: value}};       // Missing array
return [{field: value}];             // Missing json wrapper
return "processed";                  // Plain string
return $input.all();                 // No .map()
```

### 🚨 Webhook Data
```javascript
// ❌ WRONG
const name = $json.name;

// ✅ CORRECT
const name = $json.body.name;
const email = $input.first().json.body.email;
```

### Top 5 Error Patterns
```javascript
// 1. Missing return → always return
return items.map(item => ({json: item.json}));

// 2. Expression syntax confusion
const value = $input.first().json.field;   // NOT '={{$json.field}}'

// 3. Wrong return wrapper
return [{json: {result: 'success'}}];      // NOT {json: {...}}

// 4. Missing null checks
const value = item.json?.user?.email || 'no-email@example.com';

// 5. Webhook body nesting
const email = $json.body.email;            // NOT $json.email
```

### Built-in Functions
```javascript
// HTTP requests
const response = await $helpers.httpRequest({
  method: 'GET',
  url: 'https://api.example.com/data',
  headers: {'Authorization': 'Bearer token'}
});

// DateTime (Luxon)
const now = DateTime.now();
const formatted = now.toFormat('yyyy-MM-dd');
const tomorrow = now.plus({days: 1});

// JMESPath queries
const adults = $jmespath(data, 'users[?age >= `18`]');
const names = $jmespath(data, 'users[*].name');
```
**Not available**: `$helpers.httpRequestWithAuthentication`, `require()` (unless allowlisted), `$env` (when `N8N_BLOCK_ENV_ACCESS_IN_NODE=true`)

### Production Patterns

#### Multi-Source Aggregation
```javascript
const allItems = $input.all();
const results = [];
for (const item of allItems) {
  if (item.json.source === 'API1' && item.json.data) {
    results.push({json: {title: item.json.data.title, source: 'API1'}});
  }
}
return results;
```

#### Aggregation & Stats
```javascript
const items = $input.all();
const total = items.reduce((sum, item) => sum + (item.json.amount || 0), 0);
return [{json: {total, count: items.length, average: total / items.length}}];
```

#### Top N Ranking
```javascript
const topItems = $input.all()
  .sort((a, b) => (b.json.score || 0) - (a.json.score || 0))
  .slice(0, 10);
return topItems.map(item => ({json: item.json}));
```

#### Data Transform & Enrich
```javascript
return $input.all().map(item => {
  const nameParts = item.json.name.split(' ');
  return {json: {first_name: nameParts[0], last_name: nameParts.slice(1).join(' '), email: item.json.email}};
});
```

### SplitInBatches Loop Patterns

**Cross-Iteration Data Accumulation** (CRITICAL — `$('Node').all()` only returns LAST batch):
```javascript
// BEFORE the loop (reset):
const staticData = $getWorkflowStaticData('global');
staticData.results = [];
return $input.all();

// INSIDE the loop body (accumulate):
const staticData = $getWorkflowStaticData('global');
const batch = $input.all().map(item => ({...item.json, processed: true}));
staticData.results.push(...batch);
return batch.map(b => ({json: b}));

// AFTER the loop (read all accumulated):
const staticData = $getWorkflowStaticData('global');
return staticData.results.map(r => ({json: r}));
```

**pairedItem for new output items** (prevents `paired_item_no_info` errors):
```javascript
const results = [];
for (let i = 0; i < $input.all().length; i++) {
  results.push({json: {/* new data */}, pairedItem: {item: i}});
}
return results;
```

**Float Comparison for Prices**:
```javascript
// ✅ Reliable cent-level comparison
if (Math.round(newPrice * 100) !== Math.round(oldPrice * 100)) { /* real change */ }
```

### Use Code Node When...
✅ Complex multi-step transformations, custom calculations, aggregation across items, API response parsing.
❌ Simple field mapping → **Set node** | Basic filtering → **Filter node** | Simple conditions → **IF/Switch** | HTTP only → **HTTP Request node**

### Pre-Deploy Checklist
- [ ] Code not empty; return statement exists
- [ ] Return format: `[{json: {...}}]`
- [ ] Data access: `$input.all()`, `$input.first()`, or `$input.item`
- [ ] No `{{ }}` expressions in code
- [ ] Null/undefined guards present
- [ ] Webhook data via `.body`
- [ ] All code paths return same structure

---

## 7. PYTHON CODE NODES

> **⚠️ Use JavaScript for 95% of use cases.** Python only when: specific standard library needed, or significantly more comfortable with Python syntax.

### Quick Start
```python
items = _input.all()
processed = []
for item in items:
    processed.append({"json": {**item["json"], "processed": True}})
return processed
```

### Essential Rules
1. Consider JavaScript first — better n8n integration, `$helpers.httpRequest()`, Luxon DateTime.
2. Must return `[{"json": {...}}]` — same structure as JS.
3. Webhook data under `_json["body"]`.
4. **NO external libraries** — only Python standard library.

### Data Access
```python
all_items = _input.all()          # All items
first = _input.first()["json"]    # First item
item = _input.item                # Each Item mode only
data = _node["Webhook"]["json"]   # Other nodes (use .first() first!)
```

### 🚨 Webhook Data
```python
# ❌ WRONG
name = _json["name"]

# ✅ CORRECT
name = _json["body"]["name"]
name = _json.get("body", {}).get("name", "Unknown")  # Safer
```

### Return Format
```python
# ✅ CORRECT
return [{"json": {"field": value}}]
return [{"json": {"id": 1}}, {"json": {"id": 2}}]
return []

# ❌ WRONG
return {"json": {"field": value}}  # Missing list
return [{"field": value}]          # Missing json key
```

### 🚨 CRITICAL: No External Libraries
```python
# ❌ FAILS — ModuleNotFoundError
import requests, pandas, numpy, scipy
from bs4 import BeautifulSoup

# ✅ Standard library only
import json, re, base64, hashlib, math, random, statistics
from datetime import datetime, timedelta
import urllib.parse
```

**Workarounds**:
- Need HTTP? → Use **HTTP Request node** before Code node, OR switch to JS with `$helpers.httpRequest()`
- Need pandas/numpy? → Use `statistics` module, or switch to JS
- Need web scraping? → Use **HTTP Request** + **HTML Extract** nodes

### Python Modes
- **Python (Beta)** [RECOMMENDED]: `_input`, `_json`, `_node`, `_now` helpers available
- **Python Native (Beta)**: Only `_items`, `_item` — more limited, no helpers

### Common Patterns
```python
# Data transformation
return [{"json": {"name": item["json"].get("name", "").upper(), "id": item["json"].get("id")}}
        for item in _input.all()]

# Aggregation
items = _input.all()
total = sum(item["json"].get("amount", 0) for item in items)
return [{"json": {"total": total, "count": len(items)}}]

# Regex extraction
import re
emails = re.findall(r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b', text)
return [{"json": {"emails": list(set(emails))}}]

# Validation
errors = []
if not data.get("email"): errors.append("Email required")
return [{"json": {**data, "valid": not errors, "errors": errors or None}}]

# Statistics
from statistics import mean, median, stdev
values = [item["json"].get("value", 0) for item in _input.all() if "value" in item["json"]]
return [{"json": {"mean": mean(values), "median": median(values), "min": min(values), "max": max(values)}}]
```

### Standard Library Quick Reference
```python
import json; json.loads(s); json.dumps(obj)
from datetime import datetime, timedelta; datetime.now(); datetime.now() + timedelta(days=7)
import re; re.findall(r'\d+', text); re.sub(r'[^\w\s]', '', text)
import base64; base64.b64encode(data).decode(); base64.b64decode(encoded)
import hashlib; hashlib.sha256(text.encode()).hexdigest()
import urllib.parse; urllib.parse.urlencode({"key": "val"}); urllib.parse.urlparse(url)
from statistics import mean, median, stdev
```

### Top 5 Python Mistakes
1. **Importing external libraries** → Use standard library or JS
2. **Missing return** → Always return a list
3. **Dict instead of list return** → `[{"json": {...}}]` not `{"json": {...}}`
4. **Direct dict access (KeyError)** → Use `.get()`: `item["json"].get("field", default)`
5. **Webhook body nesting** → `_json.get("body", {}).get("field")`

### Python vs JS Decision
| Need | Use |
|------|-----|
| HTTP requests | JavaScript (`$helpers.httpRequest`) |
| Advanced date/time | JavaScript (Luxon) |
| Statistical functions | Python (`statistics` module) |
| 95% of cases | JavaScript |

### Pre-Deploy Checklist (Python)
- [ ] Considered JavaScript first
- [ ] Return: `[{"json": {...}}]`
- [ ] No external library imports
- [ ] `.get()` for all dict access
- [ ] Webhook data via `["body"]`
- [ ] All code paths return same structure

---

## SKILLS INTEGRATION MAP

These domains compose seamlessly. When you ask **"Build and validate a webhook to Slack workflow"**:
1. **[PATTERNS](#3-workflow-patterns)** → Identifies webhook processing pattern
2. **[MCP TOOLS](#2-mcp-tools-expert)** → `search_nodes` → `get_node` for webhook + Slack
3. **[NODE CONFIG](#5-node-configuration)** → Operation-aware config for each node
4. **[JS CODE](#6-javascript-code-nodes)** → Process webhook data with `.body` access
5. **[EXPRESSIONS](#1-expression-syntax)** → Data mapping in non-code nodes
6. **[VALIDATION](#4-validation-expert)** → `validate_workflow` before activation

---

## GOD MODE GOLDEN RULES

1. **Webhook data is under `.body`** — always. No exceptions.
2. **nodeType formats differ**: `nodes-base.*` for search/validate vs `n8n-nodes-base.*` for workflows.
3. **Build iteratively** — avg 56s between edits; never one-shot complex workflows.
4. **Validate after every significant change** — 2–3 cycles is normal.
5. **`get_node` standard detail first** — covers 95% of needs at 1/4 the tokens of full.
6. **Code nodes**: always return `[{json: {...}}]`, never use `{{ }}` syntax.
7. **Python**: no external libraries; use JavaScript for HTTP requests.
8. **SplitInBatches `main[0]` = done, `main[1]` = each batch** — not intuitive but critical.
9. **Cross-iteration data needs `$getWorkflowStaticData('global')`** — `.all()` only returns last batch.
10. **Auto-sanitization handles operator structure** — don't manually fight it.
