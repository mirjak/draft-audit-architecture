# Agent Use of Delegation and Interaction Traceability (AUDIT) Working Group Charter

Autonomous and semi-autonomous software agents, including those based on LLM-based machine learning systems, are increasingly deployed to act on behalf of users, organizations, and services across the Internet. These agents interact across multiple administrative or trust domains and can initiate actions without direct human oversight at each step.

This introduces challenges for auditability, accountability, and transparency, including:

* Difficulty attributing actions to a specific user, agent instance, or delegation context
* Loss of visibility across long-running or distributed workflows
* Inconsistent capture of delegation relationships, authorization context, and identity transitions
* Cross-domain interactions lack interoperable means to exchange or verify audit-relevant information about the participating agents and their interactions

Agents participate in two distinct classes of interactions that must be audited:

* User-facing interactions, such as prompts, conversations, and approvals, capturing user intent and human-in-the-loop decisions
* System-facing interactions, such as API calls, tool usage, and delegation to other agents or services

Effective auditing requires linking user intent to resulting system actions across protocol and administrative boundaries. While traditional workflows support evolving authorization, these transitions are usually explicit and predefined. Agent systems introduce dynamic, fine-grained authorization changes that arise during execution, driven by agent decisions, delegation, and human interaction. Auditing must therefore capture authorization as a time-evolving state and correlate these transitions across interactions and domains.

Additionally, Agent behavior may be non-deterministic and not fully predefined, requiring auditing mechanisms to capture execution context and structure as they emerge. Auditing must also distinguish between user, agent, and service identities, and ensure audit data remains interpretable across systems without shared assumptions.

## Scope and Goals
The AUDIT working group will define interoperable mechanisms for auditing and accountability of Agents and delegated systems across Internet protocols.

The group will focus on architectures, protocol-layer specifications, and data representations that enable systems to record, exchange, and verify audit-relevant information across user-facing and system-facing interactions. This includes capturing delegation chains, evolving authorization state, and enabling consistent interpretation and correlation of audit data across domains.

The working group will not define auditing policies or compliance frameworks, but instead provide the technical building blocks needed to support them.

## Deliverables
The AUDIT working group is expected to produce:

1. **Architecture for AI Agent Auditing**
An Informational RFC describing roles, trust relationships, and data flows for interoperable auditing, including the relationship between user-facing and system-facing audit signals.

2. **Audit Data Models and Semantics**
One or more Standards Track RFCs defining data models for representing audit information, including interaction records, agent identity, delegation context, authorization state over time, and action provenance.

3. **Protocol Extensions or Profiles**
One or more Standards Track RFCs specifying extensions to existing IETF protocols (e.g., HTTP, OAuth, or token formats) to convey audit-related information.

4. **Best Practices for Deployment and Operation**
An Informational or BCP document providing guidance for secure, interoperable, and privacy-aware auditing, including correlation across interaction types.
