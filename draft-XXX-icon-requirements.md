---
title: "Reuqirements for Observability, Control and Intervention of Network Management Agents"
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

TODO Abstract


--- middle

# Introduction

Agentic operations are opaque.

This document defines requirements for ICON: Observability, Control, and Intervention for Network Management Agents.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


 This document defines the following terms:

Control:
:

Intervention:
: XX

# Existing Mechanisms for Agent Observability, Control, and Intervention

Existing AI guardrails primarily operate at static boundaries, such as input validation, output filtering, and pre-action checks.

# Requirements

## Observability Requirements

OBS-1: XX
: TBD

OBS-2: XX
: TBD

OBS-3: XX
: TBD

## Control Requirements

CTL-1: Access & Permission Control
: Define and enforce what the agent is allowed to access and operate. Assume if
  an agent cannot access a system, field, or tool, it cannot cause harm through it.

CTL-2: Authorization and Approval Controls
: Govern what actions require a human decision or quality checked before they execute.

CTL-3: Execution Integrity Controls
: Ensure the agent executes tasks correctly in the right order, with expected outcomes.

CTL-4: Monitoring and Observability Controls
: Maintain continuous visibility of agent behavior. Anomalies, Agent drift, and out-of-turn activity before they go beyond the recovery point.

CTL-5: Temporal and Data/Context Validity Controls
: Ensure the agent is acting on current, accurate context. Prevent staleness, deadline/metrics-driven shortcut by agent, and decisions based on a staled state.

CTL-6: Rollback and Recovery Controls
: Ensure that agent action footprint can be identified and reversed.

CTL-7: Trust and Security Controls
: Protect the agent pipeline from adversarial manipulation.

CTL-8: Alignment and Calibration Control
: Ensure the agent optimize for what the operator actually intends. Detect and correct divergence between the agent's measured objective, its world model, and the operational reality.

## Intervention Requirements

INT-1: Human in/on the loop
: Introduce human approval or monitoring into the execution path before the agent proceeds. Used when the agent encounters a decision that exceeds its authority, confidence, or validated operational context.

INT-2: Execution Interruption
: Stop or redirect a running agent. Emergency response from a temporary pause that preserves state to a hard stop that terminates execution regardless of task progress.

INT-3: Containment
: Limit the extend of a failure (blast radius). Stop further harm from accumulating without necessarily reversing what has already occurred.

INT-4: Rollback and Recovery
: Reverse agent actions already taken. Can be a single transaction or a coordinated cross-system reversal of an entire multi-agent workflow.

INT-5: Escalation
: Route decisions, alerts, and conflicts to a higher authority. Used when the current level (human or agent) cannot (or should not) resolve the situation without supervision.

INT-6: Audit
: The post-failure capability that enables accountability, quantifies impact.

INT-7: Correction
: Correct failures by rectifying the agent's model, objective, or training distribution.

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
