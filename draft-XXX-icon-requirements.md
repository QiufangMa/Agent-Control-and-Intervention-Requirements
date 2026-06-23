---
title: "Requirements for Observability, Control and Intervention of Network Management Agents"
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


normative:

informative:

--- abstract

This document defines a set of requirements for ICON (Observability, Control, and Intervention for Network Management Agents).

It identifies gaps in existing mechanisms and specifies required interaction capabilities between humans (or other agent supervision systems) and network management agents across multi-vendor environments, specifically observability, control, and run-time intervention. The requirements aim to mitigate agent unreliability issues, and to minimize the negative impacts that agents might cause to networks when they deviate from expected behaviors.


--- middle

# Introduction

AI agents are increasingly deployed for network management tasks {{?I-D.wmz-nmrg-agent-ndt-arch}} — including service provisioning and network configuration change, service assurance and automated incident diagnosis and resolution. While the introduction of agents significantly improves the efficiency for network management, it also inevitably brings challenges such as hallucination and execution unreliability.

Existing mechanisms for agent assurance typically rely on static guardrails (e.g., input/output validation, operation allowlists/blocklists, pre-action approval), while assuming that all agent failure modes can be predefined. Unlike deterministic software systems, however, LLM-based agents exhibit emergent behaviors that cannot be fully anticipated or encoded in static rules. When agentic systems produce novel actions or reasoning paths that fall outside predefined static boundaries, it might lead to risks such as unintended configuration changes, policy violations, or cascading failures in the network.

This document defines requirements for ICON — Intervention, Control, and Observability for Network Management Agents. It emphasizes essential requirements that operators need when deploying agents in real networks for agent observability, control, and intervention.

This document specifies the communication requirements between the agent and the supervision system. It does not standardized the internal LLM architecture, planning algorithms, or training methodologies of the network management agents themselves.

This document does not specify a particular protocol, data model, or implementation API. Those topics are orthogonal to the operational requirements defined here, which are intended to be solution-neutral.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


 This document defines the following terms:

Observability:
: The visibility into an agent's internal state, decision-making logic, and workflow execution, enabling human operators or monitoring systems to understand what the agent is doing and why it behaves in a specific manner.

Control:
: A preventive mechanism that establishes a deterministic operational boundary for the agent before and during agent execution. By specifying the agent's behavior scopes, operational constraints, and security baselines, it fundamentally mitigates abnormal behaviors from agents.

Intervention:
: A reactive and emergency mechanism to intervene or take control of an agent with boundary violations, anomalies, failures, or risks. It addresses situations where agent control is insufficient, bypassed, or inapplicable.

# Existing Mechanisms for Agent Observability, Control, and Intervention

After receiving a user request, agents will perform a Chain-of-Thought (CoT) reasoning process, then it will autonomously decide whether to break down the task into subtasks, or dynamically decide to invoke multiple external tools, retrieve vector databases (RAG), or request more information from the user. Existing telemetry mechanisms are excellent for tracking traditional network infrastructure or software which are built for deterministic systems. However, they are facing severe limitations when applied to AI agents. For example, existing logging practices only record what action was taken, completely missing why it was taken, including the agent's internal reasoning provenance and confidence scores. Existing tracing mechanism designed for static and linear execution path also cannot capture the complex and dynamic execution trajectories of AI agents.

Existing AI guardrails primarily operate at static boundaries, such as input/output validation and pre-action checks. These mechanisms are designed to constrain AI agents within predefined operational and compliance boundaries, but they assume that all possible violations can be anticipated and encoded in static rules. As AI systems increasingly operate in non‑deterministic environments, these static measures are proving insufficient as they cannot detect, interrupt, and recover from unanticipated behaviours.

Although there are some modern agent systems that provide interrupt or kill switch capabilities, they remain framework-specific, insufficient, or proprietary.

These gaps motivate the requirements for agent observability, control, and intervention defined in {{requirements}}.

# Architecture

to be completed...

~~~~
+------------------------------------------------------+
|                                                      |
|                    Human Oversight                   |
|                                                      |
+----------------^------+----------+--------------+----+
 Agent Escalation|      |Policy    |Intervention  |Audit
 /Approval       |      |Injection |Command       |
 +---------------+------v----------v--------------v----+
 |                                                     |
 |                Agent Management Plane               |
 |                                                     |
 | +---------------+ +--------------++--------------+  |
 | | Observability | |   Control    || Intervention |  |
 | +------^--------+ +------+-------++-------+------+  |
 |        |                 |                |         |
 +--------+-----------------+----------------+---------+
          |                 |                |
     Telemetry    Static Guardrails  Real-time Instruction
          |                 |                |
  +-------+-----------------v----------------v---------+
  |Agent Execution Plane                               |
  | +-----------+    +-----------+       +-----------+ |
  | |           |    |           |       |           | |
  | |  Agent 1  <---->  Agent 2  <--...-->  Agent n  | |
  | |           |    |           |       |           | |
  | +-----^-----+    +-----^-----+       +-----^-----+ |
  |       |                |                   |       |
  | +-----v----------------v-------------------v-----+ |
  | |              Function Modules & Tools          | |
  | +------------------------------------------------+ |
  +-------------------------^--------------------------+
                            |
                            | Interaction
                            |
  +-------------------------v--------------------------+
  |             Network Infrastructure                 |
  +----------------------------------------------------+
~~~~

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
: The operator MUST be able to stop or redirect a running agent. Emergency response from a temporary pause that preserves state (e.g., when operators needs time to assess before deciding further action) to a hard stop that terminates execution (e.g., when agent is actively causing damage) regardless of task progress.

INT-2: Containment
: The operator MUST be able to limit the extent of a failure (impact scope). Stop further harm from accumulating without necessarily reversing what has already occurred.

INT-3: Rollback and Recovery
: The operator MUST be able to reverse actions already taken by an agent. Can be a single transaction or a coordinated cross-system reversal of an entire multi-agent workflow.

INT-4: Escalation
: The operator/agent MUST be able to route decisions, alerts, and conflicts to a higher authority. Used when the current level cannot (or should not) resolve the situation without supervision.

INT-5: Correction
: The operator MUST be able to correct deviations by modifying an agent's pending action, planned sequence, or internal state before execution proceeds.

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
