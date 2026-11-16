# 1Z0-1155-3 AI Agent Studio — 3-Day Exam Digests

Study notes for **Oracle Fusion AI Agent Studio Developer Professional — Rel 26-3 (1Z0-1155-3)**.

> **How to use this:** 11 short digests, each ~10–15 min. Day 1 = Digests 1–4, Day 2 = 5–8, Day 3 = 9–11 + practice questions. Every digest ends with **Exam traps** — the facts most likely to be tested. Format: multiple choice, 90 min, ~68% to pass (per the 26-2 version; confirm on your exam registration page).
>
> **Caveat:** I couldn't find Oracle's published objective list for the Rel 26-3 version. This guide is built from Oracle docs, Oracle Fusion COE blogs and the 26-2 exam themes. Numeric limits (code node, wait node) can change between releases — trust what your exam environment's release says.

## Day plan

| Day | Digests | Goal |
|---|---|---|
| Day 1 | 1 Big picture · 2 Building blocks · 3 Supervisor vs Workflow · 4 Tools | Know the vocabulary cold |
| Day 2 | 5 Workflow nodes · 6 Security · 7 Build/test/publish · 8 Monitoring & evaluation | Know the mechanics and exact names |
| Day 3 | 9 Deployment · 10 Scenario judgment · 11 Last-hour cram + practice quiz | Answer scenario questions like a developer |

## Digest 1 — Big picture

- **AI Agent Studio** = Fusion-native design-time environment to create, configure, test, deploy and monitor AI agents. Path: **Navigator → Tools → AI Agent Studio**.
- Included with the Fusion subscription (no extra license); custom agents that use non-standard LLMs may incur utilization cost.
- Because it's Fusion-native, agents inherit **Fusion business objects, roles, data security and business flows**. Agents do not bypass Fusion security.
- What you can do: use **preconfigured agent team templates** (Oracle + partner **AI Agent Marketplace** templates — Oracle tests/approves partner templates), build custom agents/teams from scratch, create **tools** and **topics**, upload documents for RAG, monitor and evaluate.
- **LLM choice:** Oracle-optimized models by default; also supports OpenAI, Anthropic, Cohere, Google, Meta, xAI.
- Newer features to recognize by name: **MCP support, A2A agent cards, Credential store, Monitoring dashboard, Agent tracing, Token usage, Prompt libraries & topic libraries, Agent builder assistant** (builds an agent from a high-level description), **AI Agent Studio FAQ agent, Multimodal RAG, RAG over external sources, Deterministic execution (workflow agents), Contextual/long-term memory, Content Intelligence, Business Value Dashboard (ROI), Agentic Applications Builder**.
- AI Configurator is deprecated; prompt editing moved into AI Agent Studio.

**Exam traps:** Studio is design-time + runtime-monitoring; security is inherited from Fusion; templates are copied then edited ("Use Template" opens the agent team canvas).

## Digest 2 — Building blocks

**Hierarchy:** Agent Team → Agents → Tools (+ Topics).

| Component | What it is | Memory hook |
|---|---|---|
| Agent team | Container for agents + artifacts; type is **Supervisor** or **Workflow** | The "app" |
| Agent | LLM-driven worker with persona/system prompt, topics, tools | The "person" |
| Topic | Focus area + instructions/rules that guide behaviour and tool choice (e.g., health coverage, enrollment) | Topics = **knowledge/instructions** |
| Tool | Reusable connector to data/actions; can be shared among agents | Tools = **action** |
| Prompt | System prompt: identity, job description, rules, tone | |

- **"Trinity" of agent intelligence:** Topics, Tools, Instructions.
- Agent roles: **Supervisor agent** (orchestrates/plans, delegates to workers), **Worker agent** (specialist), **User-proxy agent** (acts on behalf of a user; a.k.a. conversational agent).
- **Maximum Interactions** field = how many times the agent can interact with its topics and tools (guards against loops/cost).
- **Insert Expression** = add variables into prompts (e.g., **Current Date Time**).
- Agent team settings tabs to recognize: **LLM** (provider), **Security** (roles that can access the team), **Questions** (follow-up question prompt), **Output**, chat experience.
- Example template: Sales Quote Generator Advisor (supervisor) + Data Collector, Options, Presenter (workers).

**Exam traps:** Topics ≠ tools. Specialized workers beat one giant agent. Max Interactions is the control for runaway tool calls.

## Digest 3 — Supervisor (hierarchical) vs Workflow

| | Supervisor / Hierarchical | Workflow |
|---|---|---|
| Control | LLM plans, picks tools, delegates, synthesizes | Deterministic graph of nodes; LLM only at chosen decision points |
| Best for | Uncertainty, discovery, multi-specialist synthesis, open-ended chat | Known, repeatable processes with rules; auditability; SLA stability |
| Cost/latency | More LLM calls/tokens | Can avoid LLM nodes entirely → fewer tokens, lower latency |
| Triggers | User chat | Chat **or** background triggers (webhook, email, schedule) |

- A-Team benchmark: same RAG lookup — supervisor used 3 LLM calls / ~2,000 tokens; workflow used no LLM tokens. Lesson: **choose by task shape**; perf/cost reinforce, not replace, the design choice.
- **Hybrid is common:** a workflow handles the structured backbone and delegates open-ended steps to an Agent node (a worker or supervisor).

**Exam traps:** "Deterministic, auditable, repeatable, must follow policy" → Workflow. "Plan, choose path, coordinate specialists" → Supervisor.

## Digest 4 — Tools

| Tool | Does | Key facts |
|---|---|---|
| **Business Object** | Read/update Fusion business objects (invoices, workers, POs, custom objects) | Uses the **Fusion OpenAPI** spec; every call runs with the **logged-in user's Fusion token** → user's roles/data security apply |
| **Document (RAG)** | Grounds answers in uploaded/referenced unstructured docs (policies) | Curate sources; exclude retired docs; old Fusion RAG agents should be migrated to Studio document tools |
| **Deep Link** | Returns a URL that navigates the user to a specific Fusion page/transaction | Deep links pre-defined first; pairs well with BO tool (read data + send user to edit page) |
| **Email** | Sends email (e.g., via HCM alerts to a role) | |
| **External REST** | Calls third-party REST APIs (e.g., OIC integrations) | Needs privilege **ORA_FND_TRAP_PRIV**; credentials in credential store |
| **MCP** | Connects to an MCP-compliant server | You can pick **only a subset** of the server's tools |
| **User Session** | Returns signed-in user context (e.g., PersonNumber, Username) | Personalizes queries |
| **User Query / Multi-file processor (chat attachments reader)** | Session-context queries / read files uploaded at runtime | |
| **Calculator** | Math | Listed as pre-26C in newer lists |
| **Connector** | Newer tool type (e.g., Content Intelligence) | 26C+ |

- Tools are **reusable** across agents. External REST and Business Object tools can be used by both agents and workflows (26C).
- Identity propagation: the end-user identity can be carried from the agent → OIC → Fusion REST, so downstream APIs authorize the **real user**, not the agent.

**Exam traps:** BO tool = user's permissions, not admin's. External REST privilege = ORA_FND_TRAP_PRIV. MCP lets you select specific tools. Write tools need validation, required-field collection and safe error messages.

## Digest 5 — Workflow agent nodes

**Five node categories:** AI · Logic · Data · Workflow Control · Communication.

| Category | Nodes | Must-know constraints |
|---|---|---|
| **AI** | LLM node (classify/extract/summarize → structured JSON), Agent node (call specialist agent), Workflow (sub-flow) node, RAG/Document tool node | Use LLM only at real decision points; validate output against a schema. **Only published workflows** can be invoked as sub-flows |
| **Logic** | Set Variables, Code (JavaScript) | Code node: **~5 s limit, 10,000 iterations, 50 KB serialized output**, must be **side-effect free** (never write to systems of record). Outputs: Array, Boolean, Number, Object, String |
| **Data** | Business Object (read/write), Document Processor, External REST, Tool node (Deep Link, Chat Attachments Reader, Intent Change Indicator, User Session), Vector Read/Write | Parallelize reads, **serialize writes**; idempotency + retry on external calls; vector store = durable summaries, use UPSERT/OVERWRITE, DELETE stale |
| **Workflow Control** | If, Switch, Loop/For-each, While, Run in Parallel, Human Approval, Wait | No conflicting writes in parallel. **Human Approval** outcomes: **Approve / Reject / Request Info**, sent by email/chat with context. **Wait** max **60 min** |
| **Communication** | Notification / status / alert | Only at decision points, exceptions, completions (avoid notification fatigue) |

- Also mentioned: **Email Trigger** node, **Policy** node.
- **Four design patterns:** Chaining · Parallel execution · Switch (route by intent/policy) · Iteration/Looping.
- Canonical flow: **classify → normalize → route → act → (approve) → notify**.
- **Triggers** (workflow agents, 26A+): **Webhook, Email, Schedule**. Scheduled triggers need **ORA_DR_FAI_BATCH_JOB_MANAGER_DUTY**.

**Exam traps:** Thresholds/date math go in Logic/Code nodes, not the LLM. High-impact changes (e.g., supplier bank details) → Human Approval before write.

## Digest 6 — Security & access

**Setup order:**

1. Profile option **ORA_ASE_SAS_INTEGRATION_ENABLED = Yes** (Site) — lets Security Console work with permission groups.
2. Run **Import Resource Application Security Data**, then **Import User and Role Application Security Data** (sequential).
3. (AI help agent) **Index AI Agent Studio Assistant Documents** and **… Objects and Attributes**.
4. Security Console → Create Role → tick **Enable Permission Groups** (adds the Permission Groups tab).

**Admin/developer roles (by pillar):** e.g., HCM **ORA_HRC_HCM_AI_AGENT_MANAGEMENT_DUTY** + **ORA_DR_FAI_GENERATIVE_AI_AGENT_HCM_ADMINISTRATOR_DUTY**; CX **ORA_ZCA_MANAGE_CX_AI_AGENTS** + **…_CX_ADMINISTRATOR_DUTY**; all pillars **ORA_FAI_MANAGE_ALL_AI_AGENTS** (Manage All Intelligent Agents).

**End-user (runtime) access:** custom role with permission groups + **HRC_ACCESS_AI_AGENT_CHAT_PRIV** (Access Intelligent Agent Chat) + **ORA_DR_FAI_GENERATIVE_AI_AGENT_RUNTIME_DUTY**. Then add that role on the **agent team's Security tab** and **publish**.

- Admins who also need to *run* agents add the runtime duty too.
- Agent team Security tab **only lists roles that have permission groups enabled** — the #1 troubleshooting answer when a role is "missing".
- External REST tools → **ORA_FND_TRAP_PRIV**. Slack/Teams channels → permission groups ChannelManifest + ExternalChatCorrelation.
- Layered controls: who can build (admin roles) · who can use (team security) · what data (user token + Fusion data security) · what actions (tool scope) · audit/trace.

**Exam traps:** Least privilege; agent never gets more access than the invoking user; role missing from list → permission groups not enabled.

## Digest 7 — Build, test, publish lifecycle

- **Create:** from template (**Use Template** / copy → canvas) or from scratch (pick Supervisor or Workflow).
- **Test:** **Debug** icon on the agent team; **Playground** to edit system prompt etc. and **Save and Test** iteratively.
- **Status:** Draft → **Published**. The Agent Teams tab shows **published teams by default**; use the **Draft** button to see unpublished ones. Users see an agent only when it's published **and** their role matches its security.
- **Edit:** Agent Team Settings for LLM, chat experience, questions, security → Debug → Publish again.
- Treat prompts, tools, topics and grounding sources like code: versioning, change review, regression tests (evaluation sets), rollback plan. 26D adds CI/CD support (aistudio CLI, VS Code extension).
- Test sets should include valid, **ambiguous, unauthorized, malformed** inputs, **tool failures** and edge-case data — not just happy paths.

**Exam traps:** Not visible to users → not published or role not on Security tab. Debug before publish.

## Digest 8 — Monitoring & evaluation ("METRO")

- **Prerequisite:** run scheduled process **Aggregate AI Agent Usage and Metrics** (can be recurring, e.g., daily). No data on the dashboard → this hasn't run.
- Location: AI Agent Studio → **Monitoring and Evaluation** tab → **Monitoring** subtab. Shows aggregated metrics for supervisor and workflow runs **including drafts**.
- **Trace view:** step-by-step timeline of a session — which tools ran, duration, tokens; errors flagged inline.
- Metrics: **accuracy/correctness, latency (e.g., P50), token usage, error rate**, plus **RAG metrics** for document tools.
- **Evaluation sets:** questions + expected responses + metrics. **Specific to one agent; an agent can have multiple sets.** Tick **Enable document tool evaluation** to get RAG metrics. Metrics tab sets pass/fail thresholds (e.g., fail if Correctness < 0.8). Run via **Initiate Evaluation Run**, choosing the **agent team version**. Results compare expected vs actual. Uses LLM-as-a-judge.
- **METRO** (Metrics, Evaluation, Tracing, Reporting, Observability) = built-in; **ATLAS** = regression/release-gate testing tooling.

**Exam traps:** "Explain why the agent chose the wrong tool" → trace logs (prompt context, retrieved sources, tool chosen, payload, auth result). Monitoring = production; evaluation = pre-release quality.

## Digest 9 — Deployment options

| Path | Best for | Notes |
|---|---|---|
| **Agent Explorer** | Self-service hub of published agents | Zero setup; tabs **Needs Attention** and **Explore**; URL suffix `/ai-studio/agent-explore` |
| **Guided Journeys** | Agent task on a Redwood page/section | Two-step setup; can trigger a workflow agent on task complete/save |
| **Smart Actions (App Composer) / VB Studio** | Context-aware agent embedded in Redwood pages | Pulls page data automatically; select modules; developer config |
| **Triggers** | Background automation (workflow agents, 26A+) | Webhook, email, schedule |
| **/invokeAsync API** | Call agents from external systems | OCI IAM token setup |
| **Channels** | Slack / MS Teams | Configured from Credentials tab |
| **A2A** | Agent-to-agent with external agents | Agent cards |

**Exam traps:** "No configuration, all modules" → Agent Explorer. "Context-aware on the page" → Smart Actions/VB Studio. "External system calls agent" → invokeAsync.

## Digest 10 — Scenario judgment (how the exam thinks)

The 26-x exams favour the **safe, governed, developer-correct** answer. Pick the option that:

1. **Scopes first** — define scope, data sources, allowed actions, identity boundary and escalation points before building.
2. **Guards write tools** — validate required fields, check authorization, return safe errors; ask for missing fields instead of sending incomplete requests.
3. **Fixes grounding at the source** — wrong/conflicting answers from old docs → fix source selection, ranking, exclusion rules.
4. **Resolves ambiguity** — "update it" with two open records → confirm the record, the change and the user's authority.
5. **Respects security** — least privilege, user-aware data access, auditability.
6. **Fails gracefully** — API timeout → clear message, no secrets or stack traces, offer next step/escalation; never fabricate.
7. **Tests adversarially** — ambiguous, unauthorized, malformed, tool-failure cases.
8. **Is traceable** — logs of context, sources, tool, payload, auth result.
9. **Manages lifecycle** — version control, review, regression, rollback.
10. **Adds human oversight** for high-impact actions (approval + audit trail).
11. **Shows sources** — citations and the decision rule in responses.

Rule of thumb: if one answer is "let the LLM decide" and another adds validation/approval/least privilege, pick the latter.

## Digest 11 — Last-hour cram sheet

- Navigator → Tools → AI Agent Studio. Team types: **Supervisor, Workflow**.
- Topics = instructions/knowledge; Tools = actions; Max Interactions caps loops; Insert Expression adds variables.
- BO tool → Fusion OpenAPI + user token. External REST → ORA_FND_TRAP_PRIV. MCP → select subset of tools.
- Code node: 5 s / 10k iterations / 50 KB / no side effects. Wait ≤ 60 min. Human Approval: Approve/Reject/Request Info. Sub-flow must be published.
- Patterns: chaining, parallel, switch, loop. Parallelize reads, serialize writes.
- Triggers: webhook, email, schedule (+ ORA_DR_FAI_BATCH_JOB_MANAGER_DUTY).
- ORA_ASE_SAS_INTEGRATION_ENABLED; Enable Permission Groups; runtime duty ORA_DR_FAI_GENERATIVE_AI_AGENT_RUNTIME_DUTY; chat priv HRC_ACCESS_AI_AGENT_CHAT_PRIV; all-pillar admin ORA_FAI_MANAGE_ALL_AI_AGENTS.
- Role missing on team Security tab → permission groups not enabled.
- Agents tab shows published only by default → Draft button.
- Metrics empty → run **Aggregate AI Agent Usage and Metrics**. Eval sets: per agent, many allowed, version chosen at run time, enable doc tool eval for RAG metrics.
- Deploy: Agent Explorer (zero setup), Guided Journeys, Smart Actions/VB Studio, Triggers, invokeAsync, Slack/Teams, A2A.

## Practice and official sources

- Oracle docs: *How do I use AI Agent Studio?* (docs.oracle.com/en/cloud/saas/fusion-ai/aiaas) — Components, Access Requirements, Create Supervisor/Workflow agents, Monitor and Evaluate.
- Oracle Fusion COE learning path: https://blogs.oracle.com/fusioncoe/fusion-ai-agent-studio-learning-path (security, tools, workflow node types, monitoring, deployment).
- Oracle MyLearn: *Oracle Fusion AI Agent Studio* learning path + the official practice exam if your registration includes one.
- Free sample questions (26-2): masteryexamprep 1Z0-1155-2 page; certificationpractice.com.
- Hands-on: build one tiny supervisor agent (BO tool + deep link) and one workflow (LLM node → If → Human Approval). 30 minutes of hands-on fixes most of Digests 2–5.
