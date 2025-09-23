# CritSit Advisor

**AI-powered decision support for high-impact customer escalations**  

---

## Problem
During critical customer escalations (CritSit), managers face:
- Information overload from Dynamics, Teams, and email.
- Fragmented ownership and unclear responsibilities.
- Delays in mitigation due to blockers and resource gaps.

---

## Solution
**CritSit Advisor** is an AI-powered assistant that:
- Consumes **case summaries** (Dynamics) and **chat snapshots** (Teams).
- Applies a structured **six-module framework**:
- 
  1. Mitigation Priorities
  
  2. Blockers & Resource Gaps
  
  3. Team Responsibilities
  
  4. Tactical Action Plan
  
  5. Communication Drafts
  
  6. Monitoring Checklist
  
  
- Provides managers with **actionable insights** and clear decision paths.

---

## MVP (Hackathon Scope)
- Manual copy-paste of **case summaries** + **chat snapshots** as inputs.  
- Advisor generates structured recommendations and communication drafts.  
- Goal: validate **speed**, **accuracy**, and **transparency** improvements.

---

## Roadmap
- **Phase 1 (MVP):** Manual input testing.  
- **Phase 2:** Automated integration with Teams Copilot summaries.  
- **Phase 3:** Direct Dynamics API integration.  
- **Phase 4:** Expand to global CE&S adoption.

---

## How It Works
1. Copy Dynamics Case Summary + Teams Snapshot.  
2. Paste into CritSit Advisor input.  
3. Select output module (e.g., Mitigation Priority, Communication Draft).  
4. Receive structured guidance & suggested communication.  

---

## Team
- Project Owner: Ting Li  
- Hackathon Team: 2025 Hackathon Week  

---

## Value
**Speed** — faster incident resolution  
**Accuracy** — structured, role-aware guidance  
**Transparency** — clear, auditable recommendations  


---

# Advisor Agent — Detailed Project Brief 

> This document is organized to follow the CritSit Advisor system prompt and its Six-Part Playbook. Each Playbook section includes: purpose, required inputs, processing steps, concrete output format / templates, sample output, manager actions, and implementation notes.

---

## 0. Role & Purpose (recap)

You are the **CritSit Advisor** — an advisory AI embedded in Teams for SEV1 / SEV A incidents. Mission: provide tactical & strategic advice (no commands), clarify responsibilities, highlight blockers, recommend mitigations, and generate communications — enabling Managers to act fast and confidently.

---

## 1. Context Awareness & Inputs (MVP → future)

### Inputs (MVP + future)

1. **Case Summary** (from Dynamics/DFM) — required fields:

   * CaseNumber, Title, Impact (severity), AffectedCustomers/Regions, StartTime, BusinessImpactText, RecentActions, CurrentOwners
   * *Format (MVP):* free-text block copied from DFM. *Recommended structured schema (for automation)* shown below.

2. **Teams Snapshot** (from Teams Copilot or manual):

   * Timeline (timestamped key events)
   * Action Items Checklist (task, owner, status)
   * Responsibility Matrix (team | scope | current action | next step)
   * Open Blockers / Pending Items

3. **Knowledge Sources (RAG)**:

   * CMAT Wiki (escalation playbooks)
   * Team support pages (roles & contact)
   * Organizational policies (SLA, escalation paths)
   * Optional: historical RCA summaries, monitoring dashboards (Power BI)

4. **Optional telemetry / metrics**: current metric values (auth success rate, latency, error rates).

### Recommended automation-ready JSON schema (example)

```json
{
  "caseNumber": "DFM-2025-1234",
  "title": "EU auth failures",
  "impact": "SEV A",
  "affectedRegions": ["EU"],
  "startTime": "2025-09-10T03:00:00Z",
  "businessImpact": "30% login failures, major customers affected",
  "recentActions": [
    {"time":"2025-09-10T03:40Z","action":"DB failover","owner":"DB Team","result":"partial relief"}
  ],
  "currentOwners":["Infra Team","DB Team","NET Team"],
  "teamsSnapshot": {
    "timeline":[{"time":"2025-09-10T03:00Z","event":"Auth spike"}],
    "actions":[{"task":"capture packets","owner":"NET","status":"pending"}],
    "responsibilityMatrix":[{"team":"DB","scope":"SQL cluster","currentAction":"failover","nextStep":"monitor replication lag"}],
    "blockers":["need packet capture access","no schema traces"]
  }
}
```

---

## 2. Processing Overview (how the Agent produces Playbook outputs)

1. **Preprocessing**

   * Normalize timestamps, extract entities (teams, components, case numbers), redact PII per policy.
   * Map teams & systems using responsibility KB (CMET / support pages).

2. **Context Fusion**

   * Merge Case Summary + Teams Snapshot + Knowledge docs into a consolidated context object.
   * Index by entities (caseNumber, service, region).

3. **LLM Reasoning Stage**

   * System prompt instructs model to: prioritize mitigation over RCA for high impact; produce 6 structured outputs (the Playbook).
   * Use retrieval-augmented generation (RAG) to supply relevant wiki passages or prior RCAs.

4. **Structured Output Generation**

   * Produce JSON + human readable Markdown outputs for each Playbook module (templates below).
   * Attach "confidence" + "rationale" for major recommendations.

5. **Post-processing**

   * Format comms drafts, canonicalize owner names, prepare monitoring checklist items and suggested metric thresholds.

6. **Delivery**

   * Return as a structured Playbook for Manager in 1:1 chat; optionally export to case notes or email.

---

## 3. The Six-Part Playbook — Detailed Modules

> For each module: Purpose → Inputs used → LLM prompt pattern → Output template → Example (short) → Manager action.

### A. Mitigation Priority Recommendation

**Purpose:** Rapidly identify top business risk(s) and immediate mitigation actions (top 2), and justify why mitigation should precede RCA.

**Inputs:** Case summary (impact), timeline, key metrics, blockers.

**LLM prompt pattern (short):**

```
[System] You are CritSit Advisor. Given the case context, summarize the top business risk and recommend the top 2 immediate mitigation actions. Prioritize actions that reduce customer impact ASAP; explain why mitigation should be prioritized over RCA.
[Context] {case_json}
```

**Output template (Markdown + small JSON):**

```markdown
### Mitigation Priority
- **Top Business Risk:** <one-sentence>
- **Top Mitigation Actions (Immediate):**
  1. Action A — Owner: <team> — ETA: <minutes/hours> — Expected Impact: <e.g., reduce error rate by X%>
  2. Action B — Owner: <team> — ETA: ...
- **Justification:** <why mitigation before RCA>
- **Confidence:** High/Medium/Low
```

**Example:**

* Top Business Risk: EU auth failures causing SLA breach.
* 1. Route EU auth traffic to secondary cluster (Infra/DB) — reduces failures immediately.
* 2. Apply edge retry + rate-limiting (App Team) — reduces load spikes.
* Justification: immediate customer impact reduction; RCA can run in parallel.

**Manager action:** Approve/assign mitigations, notify Comms for customer messaging.

---

### B. Blockers & Resource Gaps

**Purpose:** Identify what is preventing progress and what roles/resources are missing.

**Inputs:** Responsibility matrix, open blockers, timeline.

**Prompt pattern:**

```
Identify each blocker in the context, describe its impact, and specify the missing role/access/resource required to resolve it. Recommend who to engage.
```

**Output template:**

```markdown
### Blockers & Resource Gaps
- **Blocker #1:** <short description>
  - Impact: <business/technical>
  - Required: <role/system/access>
  - Suggested Owner to pull in: <person/team>
  - ETA to unblock: <est>
- ...
```

**Example:**

* Blocker: No packet capture access. Impact: cannot confirm network root cause. Required: NET ops elevated access or vendor packet capture. Suggested: Assign NET lead + request infra admin.

**Manager action:** Assign coordinator to secure access; escalate to infra ops.

---

### C. Team Responsibility Clarification

**Purpose:** Remove scope confusion, clarify who owns what, and provide engagement guidance.

**Inputs:** Responsibility matrix, knowledge base mapping of team scopes.

**Prompt pattern:**

```
Check responsibilities for overlapping scopes or misrouted tasks. For each ambiguous item, recommend the correct owning team and provide quick engagement guidance (what to ask/do).
```

**Output template (table):**

| Task/Area            | Current Owner (from context) | Recommended Owner | Engagement Guidance                  |
| -------------------- | ---------------------------: | ----------------- | ------------------------------------ |
| DB replication check |                      DB Team | DB Team           | Request replication logs + timeframe |

**Example Recommendation:**

* App team is responsible for providing schema trace logs (not DB team). Ask App to provide traces within 30 minutes.

**Manager action:** Update case notes with clarified owners; message channels tagging owners with explicit asks.

---

### D. Tactical Plan (2-hour action plan)

**Purpose:** Provide a time-boxed, owner-assigned tactical plan for the next 2 hours.

**Inputs:** Mitigation actions, blockers, available owners.

**Prompt pattern:**

```
Create a tactical 2-hour plan: time slices (0-30, 30-60, 60-120 min), owner, specific actions, success criteria for each step.
```

**Output template (timeline table):**

| Window | Action                                      | Owner   | Success Criteria                                     |
| ------ | ------------------------------------------- | ------- | ---------------------------------------------------- |
| 0–30m  | Assign Incident Coordinator; confirm access | Manager | Coordinator assigned, access confirmed               |
| 30–60m | NET: capture packets; DB: check replication | NET/DB  | Packet capture uploaded, replication lag < threshold |

**Example** (see earlier sample).

**Manager action:** Convert timeline into tasks in case system; call quick standups at milestones.

---

### E. Communication Draft Generation

**Purpose:** Produce ready-to-send internal and customer communications with appropriate tone and content.

**Inputs:** Mitigation actions, current status, expected ETA for next update.

**Prompt pattern:**

```
Generate: (A) internal update (technical) for case group; (B) customer update (concise, business-facing) with next update ETA.
```

**Internal template (short):**

```
Subject: [CaseNumber] – Current Status & Actions
Summary: <one-liner>
Actions taken: - ...
Open blockers: - ...
Owners & next steps: - ...
Next update: <time>
```

**Customer template (short/passable):**

```
We are aware of elevated login issues affecting customers in [region]. Our team applied temporary mitigations and is investigating root causes. We will provide next update by [time].
```

**Manager action:** Review and send after minimal edits; set cadence for updates.

---

### F. Monitoring & Documentation Guidance

**Purpose:** Define what to monitor after mitigations and what to record in case notes to support RCA.

**Inputs:** System metrics, actions performed, alerts.

**Prompt pattern:**

```
Provide a monitoring checklist post-mitigation, including metrics to watch, log entries to capture, alert thresholds, and case note fields to fill.
```

**Output checklist example:**

* Monitor auth success rate EU — alert if >5% failure for 5 minutes.
* Monitor DB replication lag — alert if >500ms.
* Capture packet capture file and link to case notes.
* Document timestamped action log: who, what, why.

**Manager action:** Assign monitoring responsibilities; create alerts; confirm logs saved in case notes.

---

## 4. CritSit Playbook — Exact Manager Prompts & Example Outputs

Below are **ready-to-use prompts** (copy/paste) the Manager can feed the Advisor after pasting the Case Summary + Teams Snapshot.

### Q1 — Mitigation Priority

**Prompt to Advisor:**

```
Mitigation Priority — Based on the case context, state the top business risk and list the top 2 immediate mitigation actions. For each action include owner, ETA, expected customer impact reduction, and a one-sentence justification why to prioritize mitigation over RCA.
```

**Example output:** (as earlier) — includes two actions, owners, ETAs, expected impact and justification.

### Q2 — Blockers & Resource Gaps

**Prompt:**

```
Blockers & Gaps — From the context, enumerate key blockers, describe their impact, list the missing roles/resources, and recommend whom to contact to unblock them.
```

### Q3 — Team Responsibility

**Prompt:**

```
Team Responsibility — Review the responsibility matrix and identify any scope overlaps or misrouted tasks. Recommend the correct owner for each unresolved task and provide short engagement guidance.
```

### Q4 — Tactical 2-hour Plan

**Prompt:**

```
Tactical Plan (2 hours) — Provide a time-boxed plan with windows 0–30, 30–60, 60–120 minutes. For each window specify owner, concrete action, and success criteria.
```

### Q5 — Communication Drafts

**Prompt:**

```
Communications — Draft (A) internal technical update for the case group and (B) customer-facing update. Keep customer message simple and include next update ETA.
```

### Q6 — Monitoring Checklist

**Prompt:**

```
Monitoring — After the recommended mitigations, list the monitoring items, thresholds/alerts, logs to capture, and case note entries to record.
```

---

## 5. Implementation Guidance (MVP → Automation → Production)

### MVP (what to ship first)

* **Manual Inputs:** Manager copies Case Summary & Teams Snapshot (Teams Copilot output) and pastes into CritSit Advisor 1:1.
* **LLM:** Azure OpenAI or another enterprise LLM with system prompt that enforces the six outputs.
* **Output UI:** 1:1 chat with structured Markdown + JSON attachments, and copy-to-case-notes action.

**Why MVP:** fastest path to value; validates usefulness and collects real-user feedback.

### v2 (semi-automated)

* **Teams Copilot Structured Snapshot:** standard prompt (provided earlier) produces a consistent Snapshot (Timeline, Checklist, Responsibility Matrix). Manager clicks “Export to Advisor” (or copies + paste).
* **Prebuilt Prompts / Buttons:** in Teams, create action buttons to ask each Playbook question automatically.

### v3 (connector & realtime) — Innovation Program (production path)

* **Teams Connector by Case Number:**

  * User pastes CaseNumber; system queries Teams graph to find channels/threads with that case number in name/subject and pulls messages.
  * Real-time ingestion: agent watches the thread and triggers incremental Playbook updates as chat evolves.
* **Dynamics API Integration:** fetch case summary, actions, customer contact automatically.
* **RAG & KB Integration:** query CMAT wiki and internal docs via vector DB (Azure Cognitive Search) to enrich LLM prompts.
* **Governance:** role-based access, PII redaction, audit logs.

### Architecture (textual)

* Ingestion layer (DFM API / Teams Graph / manual paste) → Normalizer (entity extraction, redaction) → Vector DB + KB Retriever → LLM + system prompt → Output Formatter → UI (Teams 1:1 + Case Notes + Export).

### Security & Compliance

* Enforce PII redaction before LLM call.
* Logging & audit trail of what was fed to the model.
* RBAC for who can request automatic chat pulls.
* Data residency per org policy.

---

## 6. Usage Scenarios & Example Walkthrough (end-to-end)

### Quick Manager workflow (MVP)

1. Prepare: Copy CaseSummary from DFM.
2. Run Teams Copilot prompt to generate Snapshot (or use prepared Snapshot).
3. Paste both into CritSit Advisor 1:1.
4. Ask Playbook questions (use the six prepared prompts).
5. Receive Playbook outputs; assign owners; post internal update; set monitoring.

### Example: from your demo (concise)

* Input: CaseNumber DFM-2025-0457 + Teams Snapshot with timeline and blockers.
* Ask Q1…Q6 → outputs: prioritized actions (fallback & retries), blockers (packet capture access), responsibility clarification (App vs DB), 2h tactical plan, internal + customer updates, monitoring checklist.
* Manager assigns tasks and copies communications to case notes.

---

## 7. Metrics / Success Criteria (what to measure)

* **Time-to-first-decision** (median) — target reduce by 80–90% vs baseline.
* **Time-to-mitigation** — minutes from case open to mitigation action.
* **Customer-visible downtime** (minutes reduced).
* **Adoption**: # of managers using Advisor per month.
* **Accuracy / Satisfaction**: manager rating of the Advisor recommendations (1–5).
* **False positive recommendations**: ratio of suggestions judged incorrect.

---

## 8. Risks & Mitigations

* **Over-reliance on LLM** — always mark outputs as advisory; require human sign-off.
* **Data leakage / PII** — mandatory redaction + minimize logs retention.
* **Wrong ownership routing** — keep simple escalation rules; add confirmation steps before reassigning.
* **Insufficient context** — encourage including certain structured fields; auto-prompt for missing critical data.

---

## 9. Roadmap & Next Steps (recommended)

1. **Now (MVP):** stabilize 1:1 flow, collect feedback from 3 pilot users, record use cases & improvement backlog.
2. **Near-term (2–6 weeks):** build Teams Copilot Snapshot prompt template and integrate a “Copy to Advisor” button or macro.
3. **Mid-term (1–3 months):** prototype Teams connector by CaseNumber in dev tenant; prototype Dynamics API retrieval; pilot with select customers/CMET.
4. **Long-term:** apply for Innovation Program to harden, add RBAC, scale across CE\&S, full production path, integrate with monitoring dashboards (Power BI) and automation runbooks.

---

## 10. Appendix — Ready-to-use artifacts

### A. Teams Copilot Snapshot Prompt (recommended)

```
Summarize the last 1–3 days of this incident chat into a structured Snapshot for CritSit Manager support. Output as Markdown.

Sections:
1) Timeline of Key Events — [timestamp] Event — Owner — Result
2) Actions Taken — [Team] — [Action] — [Outcome]
3) Open Blockers / Pending Items — [Blocker] — [Impact] — [Waiting on]
4) Responsibility Matrix (table: Team | Scope | Action Taken | Next Step)
5) Next Steps Planned — [Action] — [Owner] — [Due]
Keep it concise so it can be pasted to CritSit Advisor.
```

### B. Manager Playbook Prompts (copy/paste)

(See Q1–Q6 prompts in the Playbook section above — ready to paste.)

### C. Internal & Customer Template (copy/paste)

**Internal:**

```
Subject: [CaseNumber] — Status: [Short summary]
Summary: [one line]
Actions taken: ...
Open blockers: ...
Owners & next steps: ...
Next update: [time]
```

**Customer:**

```
We are aware of elevated [service] issues affecting [region]. Our teams have applied temporary mitigations and are actively investigating. Next update: [time]. Thank you for your patience.
```

---
