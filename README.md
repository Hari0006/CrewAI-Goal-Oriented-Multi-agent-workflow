# CrewAI-Goal-Oriented-Multi-agent-workflow

**1. Introduction & Overview**

Modern SaaS platforms process high volumes of complex technical inquiries spanning login failures, API integration errors, database bottlenecks, and webhook exceptions. Manually triaging, searching product documentation, drafting customer responses, and routing critical bugs to engineering is time-consuming and inconsistent.

This project implements a goal-oriented, multi-agent workflow using CrewAI to automate technical support triage. The system uses four specialized agents arranged in a structured sequential dependency chain to analyze errors, cross-reference static product policies, generate customer-facing troubleshooting instructions, and execute strict escalation governance for server-level bugs.

**2. Agent Roles & Responsibilities**

The agent team features strict separation of concerns, ensuring each entity has a precise scope with minimal overlap:

- Technical Issue Diagnosis Specialist (diagnosis_agent)

   - Role: Technical Diagnostician

  - Goal: Analyze raw user support queries to determine error categories, severity levels, affected system components, and technical symptoms.

  - Backstory: Expert in SaaS diagnostics, log parsing, and system architecture.

- Product Knowledge Reasoning Specialist (knowledge_reasoning_agent)

  - Role: Knowledge & Policy Specialist

  - Goal: Apply static SaaS product policies, error-code lookups, and known workarounds without relying on live external RAG databases.

  - Backstory: Maintains an exhaustive mental catalog of SaaS documentation, API specifications, and system limitations.

- Troubleshooting Response Drafter (troubleshooting_response_agent)

  - Role: Customer Communication Specialist

  - Goal: Craft clear, professional, step-by-step remediation instructions and empathetic customer-facing replies.

  - Backstory: Skilled at translating technical fixes and error codes into actionable, reassuring guidance.

- Engineering Escalation Coordinator (engineering_escalation_agent)

  - Role: Escalation & Risk Coordinator

  - Goal: Evaluate system risk, severity, and uncertainty to decide whether an issue can be resolved locally or requires tier-3 engineering intervention.

  - Backstory: Guards system integrity by routing unresolvable bugs, data corruption, and 5xx server crashes to core engineering.

**3. Task Flow & Inter-Agent Handoffs**

The workflow operates via a strict sequential dependency chain, where each agent directly consumes and builds upon the structured output of the preceding agent:

 - Task 1 (diagnose_issue_task): Ingests the raw {customer_query}. The Diagnosis Specialist extracts structured metadata (Category, Severity, Affected Component, and Technical Summary).

 - Task 2 (reason_knowledge_task): Takes the structured technical diagnosis and cross-references it against our static SaaS documentation rules. It returns either a matching policy guide or triggers the fallback rule: KNOWN BUG / NO LOCAL FIX.

 - Task 3 (draft_response_task): Consumes the diagnosis and knowledge base output to draft a structured, empathetic, step-by-step troubleshooting guide for the user.

 - Task 4 (apply_escalation_logic): Evaluates the entire context, checking for critical severity ratings, system crashes, or knowledge base bug flags to issue a final status (resolved or requires_engineering_escalation).


**4. Escalation Logic & Governance Rules**

Escalation is treated as a strict operational boundary rather than an arbitrary decision, governed by three deterministic rules:

 - Severity Threshold: Any ticket classified with a severity of High or Critical (e.g., data loss, unhandled 5xx server exceptions) automatically triggers engineering routing.

 - Knowledge Gap / Bug Flag: If Task 2 outputs KNOWN BUG / NO LOCAL FIX (such as webhook payloads exceeding architectural size limits causing database deadlocks), local closure is overridden.

 - Uncertainty & Integration Risk: Complex custom integration failures beyond standard documentation bounds are escalated to prevent giving incorrect technical instructions.

**5. Sample Input & output**

*Input Query Tested in Code:* "We are receiving intermittent 500 Internal Server Errors and database deadlock exceptions whenever our webhook payload exceeds 5MB. This is breaking our production synchronization."

*Step-by-Step Execution Results:*

- Agent 1: Technical Issue Diagnosis Specialist

  - Category: api_error

  - Severity: critical

  - Affected Component: webhook_engine, database

  - Diagnosis: Webhook failures and database deadlocks caused by payloads exceeding 5MB, breaking production sync.

- Agent 2: Product Knowledge Reasoning Specialist

  - Rule Matched: Policy 3.1 (Max payload supported is 2MB; larger payloads causing 500 errors/deadlocks are a backend engine bug).

  - Output: KNOWN BUG / NO LOCAL FIX

- Agent 3: Troubleshooting Response Drafter

  - Output: Drafted an empathetic customer email acknowledging the issue and providing temporary workarounds (payload compression, batching requests, keeping payloads under 5MB).

- Agent 4: Engineering Escalation Coordinator

  - Final Status: requires_engineering_escalation

  - Reason: Critical severity, 500 server crashes, database deadlocks, and a KNOWN BUG tag required sending the issue directly to the engineering bug queue.

**6. Design Rationale**

- Tight Responsibility Boundaries: Preventing role overlap ensures that the diagnosis agent focuses strictly on log and symptom analysis, while the response drafter focuses entirely on tone and clarity, eliminating conflicting behaviors.

- Controlled Non-RAG Knowledge Store: Simulating knowledge sources using structured static policy blocks fulfills project constraints while modeling real enterprise SOPs cleanly and reliably without vector store overhead.

- Defensive Escalation Veto: Empowering the final escalation coordinator with absolute veto authority ensures that critical system bugs are never accidentally auto-closed by friendly customer support wording, protecting operational integrity.
