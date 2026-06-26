---
title: "Architecture and Requirements for Observability, Control and Intervention of Network Management Agents"
abbrev: "icon requirements"
category: info

docname: draft-XXX-icon-requirements-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - Network Management Agents
 - Observability
 - Control
 - Intervention

author:
-
   fullname: Qiufang Ma
   organization: Huawei
   role: editor
   street: 101 Software Avenue, Yuhua District
   city: Nanjing, Jiangsu
   code: 210012
   country: China
   email: maqiufang1@huawei.com
-
   fullname: Daniele Ceccarelli
   organization: Cisco
   email: dceccare@cisco.com
-
   fullname: Qin Wu
   organization: Huawei
   street: 101 Software Avenue, Yuhua District
   city: Nanjing, Jiangsu
   code: 210012
   country: China
   email: bill.wu@huawei.com

contributor:
-
  fullname: Yuanyuan Yang
  organization: Huawei
  street: 101 Software Avenue, Yuhua District
  city: Jiangsu
  code: 210012
  country: China
  email: yangyuanyuan55@huawei.com

normative:

informative:

--- abstract

This document defines architecture and a set of requirements for ICON (Observability, Control, and Intervention for Network Management Agents).

It identifies gaps in existing mechanisms and specifies required interaction capabilities between Agent supervision systems and network management agents across multi-vendor environments, specifically observability, control, and runtime intervention. The requirements aim to mitigate agent unreliability issues, and to minimize the negative impacts that agents might cause to networks when they deviate from expected behaviors.


--- middle

# Introduction

AI agents are increasingly deployed for network management tasks {{?I-D.wmz-nmrg-agent-ndt-arch}} — including service provisioning and network configuration change, service assurance and automated incident diagnosis and resolution. While the introduction of agents significantly improves the efficiency for network management, it also inevitably brings challenges such as hallucination and execution unreliability.

Existing mechanisms for agent assurance typically rely on static guardrails (e.g., input/output validation, operation allowlists/blocklists, pre-action approval), while assuming that all agent failure modes can be predefined. Unlike deterministic software systems, however, LLM-based agents exhibit emergent behaviors that cannot be fully anticipated or encoded in static rules. When agentic systems produce novel actions or reasoning paths that fall outside predefined static boundaries, it might lead to risks such as unintended configuration changes, policy violations, or cascading failures in the network.

This document defines architecture for ICON — Intervention, Control, and Observability for Network Management Agents. It also emphasizes essential requirements that supervisors need when deploying agents in real networks for agent observability, control, and intervention.

This document specifies the architecture and communication requirements between the agent and the supervision system. It does not standardized the internal LLM architecture, planning algorithms, or training methodologies of the network management agents themselves.

This document does not specify a particular protocol, data model, or implementation API. Those topics are orthogonal to the operational requirements defined here, which are intended to be solution-neutral.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


 This document defines the following terms:

Observability:
: The visibility into an agent's internal state, decision-making logic, and workflow execution from its external telemetry outputs (e.g., logs, traces, metrics), enabling a supervisor to understand what the agent is doing and why it behaves in a specific manner.

Control:
: A preventive mechanism that establishes a deterministic operational boundary for the agent before and during agent execution. By specifying the agent's behavior scopes, operational constraints, and security baselines, it fundamentally mitigates abnormal behaviors from agents.

Intervention:
: A reactive and emergency mechanism to intervene or take control of an agent with boundary violations, anomalies, failures, or risks. It addresses situations where agent control is insufficient, bypassed, or inapplicable.

Supervisor:
:The entity responsible for monitoring, controlling, and intervening in the agent's lifecycle. A supervisor can be a human operator, an automated high-privilege governance system, or an orchestrator.

# Existing Mechanisms for Agent Observability, Control, and Intervention

After receiving a user request, agents will perform a chain-of-thought (CoT) reasoning process, then it will autonomously decide whether to break down the task into subtasks, or dynamically decide to invoke multiple external tools, retrieve vector databases (RAG), or request more information from the supervisor. Existing telemetry mechanisms are excellent for tracking traditional network infrastructure or software which are built for deterministic systems. However, they are facing severe limitations when applied to AI agents. For example, existing logging practices only record what action was taken, completely missing why it was taken, including the agent's internal reasoning provenance and confidence scores. existing tracing mechanism designed for static and linear execution path also cannot capture the complex and dynamic execution trajectories of AI agents.

Existing AI guardrails primarily operate at static boundaries, such as input/output validation and pre-action checks. These mechanisms are designed to constrain AI agents within predefined operational and compliance boundaries, but they assume that all possible violations can be anticipated and encoded in static rules. As AI systems increasingly operate in non‑deterministic environments, these static measures are proving insufficient as they cannot detect, interrupt, and recover from unanticipated behaviours.

Although there are some modern agent systems that provide interrupt or kill switch capabilities, they remain framework-specific, insufficient, or proprietary.

These gaps motivate the architectural framework and requirements for agent observability, control, and intervention defined in {{architecture}} and {{requirements}}, respectively.


# Architectural Framework for ICON {#architecture}

This section describes the reference architecture for ICON. The architecture defined in {{arch}} serves as the structural foundation to derive the requirements specified in {{requirements}}.

~~~~
+----------------------------------------------------------+
|     Agent Governance Plane                               |
|     +-----------------------------------------------+    |
|     |             Human Oversight                   |    |
|     +-----------------------------------------------+    |
|     +-----------------------------------------+          |
|     |ICON Client                              |          |
|     |  +-------------++-------++------------+ |          |
|     |  |Observability||Control||Intervention| |   ...    |
|     |  +-----^-------++--+----++------^-----+ |          |
|     +--------+-----------+------------+-------+          |
+--------------+-----------+------------+------------------+
               |           |            |
               |           |            | ICON Interface
               |           |            |
+--------------+-----------v------------v------------------+
|              ICON Enforcement Component                  |
+----------------------------------------------------------+
+----------------------------------------------------------+
|    Agent Execution Plane                                 |
|    +-----------+    +-----------+       +-----------+    |
|    |           |    |           |       |           |    |
|    |  Agent 1  <---->  Agent 2  <--...-->  Agent n  |    |
|    |           |    |           |       |           |    |
|    +-----^-----+    +-----^-----+       +-----^-----+    |
|          |                |                   |          |
|    +-----v----------------v-------------------v-----+    |
|    |              Function Modules & Tools          |    |
|    +------------------------------------------------+    |
+----------------------------^-----------------------------+
                             |
                             | Interaction
                             |
+----------------------------v-----------------------------+
|                Network Infrastructure                    |
+----------------------------------------------------------+
~~~~
{: #arch title="ICON Architecture" artwork-align="center"}

## Agent Gonvernance Plane

Agent governance layer is the Agent supervision and management layer which is used to manage, monitor, and regulate autonomous AI agents. It might include other technical and operational pilars such as agent identity management, which are out of the scope of ICON.

### Human Oversight

Human oversight represents the top-level authority of the agent governance. It provides the post-execution feedback, injects global policies, reviews agent escalation requests, and issues high-level intervention commands during crises or anomalies.

 * Policy and Constraint Injection:
 : Human operators could express high-level operational constraints or boundaries. These intents are translated into machine-readable policies by ICON client and sent to the policy enforcement component.

 * Escalation Handling:
 : When an active agent encounters an ambiguous scenario, a conflict between different policies, or a decision whose confidence score falls below a predefined threshold, the execution plane suspends the task and escalates it to operators. A human operator could either approve, reject, or modify the agent's pending action sequence.

 * Emergancy Intervention Trigger:
 : In the scenario of an unforeseen and deviated agent behavior (e.g., an agent entering an infinite inference loop or executing based on outdated data or incorrect assumption), human oversight allows immediate, manual injection of high-priority override instructions (e.g., global kill switches or behavior corrections).

 * Post-Execution Feedback:
 : Beyond runtime intervention, operators could also provide a critical retrospective evaluation feedback. Following an incident, anomaly, or successful resolution, human operators may inject multi-dimensional feedback (e.g., critiquing the agent’s reasoning paths, correcting intermediate planning errors, or evaluating the quality of tool selection). This retrospective feedback could be used to update the prompt templates or refine downstream guardrail policies, preventing the recurrence of similar behavioral drifts.


It is worth mentioning that human operators rarely send raw ICON protocol payloads directly to ICON enforcement component. They could use more flexible and human-friendly formatting such as natural language which is relayed to the ICON client to translate into structured ICON signals for normalization and forwarding.

### ICON Client

The ICON client is the logical entity which acts on behalf of human operators to monitor and control Agents, and to intervene in their behaviors when necessary. It is responsible for the multi-Agent observability aggregation, policy control, and emergency intervention logic for heterogeneous multi-Agent autonomous networks.

 * Observability:
 : It receives normalized observation streams transmitted from downstream ICON enforcement components. It provides human operators with comprehensive agent behavioral visibility and identifying operational anomalies or performance drifts.

 * Control:
 : It acts as the centralized Policy Decision Point (PDP) {{?RFC3198}} that translates human operational guidelines into agent behavioral boundaries, guardrails, or operational constraints. It dynamically pushes a set of structured rules or policy constraints down to enforcement components.

 * Intervention:
 : It hosts the emergency orchestration logic required to reactively instruct agents in response to boundary violations, anomalies, failures, or operational risks. Upon detecting critical policy violations or receiving manual override commands from human oversight, it generates specific instructions (such as pause or terminate) and pushes them down to the enforcement component. In addition, it also receives upstream messages initiated by agents, such as escalation requests that proactively require human intervention.

In practical deployments, ICON client could be embedded within network management systems/OSS, an external Agent governance platform, or a even upper-layer supervisor Agent.

## ICON Enforcement Component (ICON Server)

ICON enforcement component serves as the unified bridge between Agent supervision signals and native Agent execution workflows. It abstracts heterogenous agent runtime and exposes standardized ICON interaction endpoints.

 * Observability Enforcement:
 : It collects raw runtime observation data from local multi-agent systems, normalizes raw logs, traces and metrics into unified formats, and transmits observation streams upward to the remote ICON Client for centralized storage, analysis, and visualization.

 * Control Enforcement:
 : It receives and enforces operational constraint rules or policies pushed by ICON Client as a Policy Enforcement Point (PEP) {{?RFC3198}}. Examples include access control for the agent's invocation of tools, and triggers approval request workflows according to the predefined rules.

 * Intervention Enforcement:
 : It accepts runtime override instructions delivered from ICON client, and executes corresponding immediate actions on specific running agent instance, such as suspending ongoing agent operation while retaining a snapshot of the execution state and context for recovery, or reversing a specific action taken by the agent.

In practical deployments, ICON enforcement component could be implemented at the AI Agent gateway, or deployed as a runtime wrapper around individual agent instances.



# Requirements {#requirements}

## Observability Requirements

OBS-1: Execution Trajectory Capture
: The framework MUST support visibility into complete agent execution trajectories, including reasoning chain/chain-of-thought, actions planning, executed steps, and observations in specific format.

OBS-2: Reasoning Provenance Capture
: The framework MUST support visibility into reasoning provenance, including intent understanding, inference, confidence scores, evidence chains. Agents should expose why a decision was made, not only what action was taken.

OBS-3: Metrics Collection
: The framework MUST support collection of metrics characterizing agent operational health, including action execution latency, error rates, token consumption, resource usage, task completion rates, and confidence calibration.

OBS-4: Multi-Agent Correlation
: The framework MUST support logging and trace correlation across distributed multiple agent execution, supporting querying and analysis.

## Control Requirements

CTL-1: Access and Permission
: The framework MUST provide mechanisms to define and enforce what systems, actions, skills, tools, data fields, and network domain an agent is permitted to access and operate.

CTL-2: Intent Validation and Alignment
: The framework MUST ensure the agent validate intents from the operator or other agents before execution. It MUST also ensure the agent optimize for what the operator actually intends.

CTL-3: Temporal and Data/Context Validity
: The framework MUST ensure the agent is acting within authorized time windows and under valid operational conditions such as accurate context and data.

CTL-4: Authorization and Approval (Escalation)
: The framework MUST support the designation of certain actions or decisions as requiring explicit human approval before execution. It SHOULD also support configurable escalation chain and communication methods/channels to route approval requests sequentially to designated personnel.

CTL-5: Failure and Liveness
: The framework MUST allow specifying agent behaviors when encountering predefined failure modes and enable agents to periodically report their liveness status for health monitoring.


## Intervention Requirements

INT-1: Execution Interruption
: The supervisor must be able to stop or redirect a running agent. Emergency response from a temporary pause that preserves state (e.g., when operators needs time to assess before deciding further action) to a hard stop that terminates execution (e.g., when agent is actively causing damage) regardless of task progress.

INT-2: Containment
: The supervisor must be able to limit the extend of a failure (blast radius). Stop further harm from accumulating without necessarily reversing what has already occurred.

INT-3: Rollback and Recovery
: The supervisor must be able to reverse actions already taken by an agent. Can be a single transaction or a coordinated cross-system reversal of an entire multi-agent workflow.

INT-4: Escalation
: The supervisor must be able to route decisions, alerts, and conflicts to a higher authority. Used when the current level cannot (or should not) resolve the situation without supervision.

INT-5: Correction
: The supervisor must be able to correct failures by modifying an agent's pending action, planned sequence, or internal state before execution proceeds.


INT-6: Auditability and Accountability
: The framework MUST support attribution of failures to responsible entities (e.g., agents, humans, or systems), quantification of consequences (e.g., resource impact, downtime, cost), and traceability from failure through intervention to recovery.
This post-failure capability MUST enable accountability and quantify operational impact.


# Security Considerations

This document defines a set of functional requirements for observability, control, and intervention of AI agents in the context of network management.

The requirements themselves do not introduce additional security vulnerabilities. Rather, this document requirements some security safeguards such as access control, identity authentication, and integrity guarantees that should be enforced by the implementation and deployed systems.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
