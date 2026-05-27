---
title: "An Architecture for Auditing Agent Delegation and Interactions"
abbrev: "Agent Auditing Architecture"
docname: draft-kuehlewind-audit-architecture-latest
category: info
consensus: true
submissionType: IETF

ipr: trust200902
# area: "Security"
keyword:
 - audit
 - accountability
 - delegation
 - agent
 - attestation
 - transparency
 - traceability

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
 - name: Mirja Kuehlewind
   organization: Ericsson
   email: mirja.kuehlewind@ericsson.com
 - name: Henk Birkholz
   organization: Fraunhofer SIT
   email: henk.birkholz@ietf.contact

normative:
  RFC9334: rats-arch
  I-D.ietf-scitt-architecture: scitt-arch
  I-D.ietf-wimse-arch: wimse-arch
  I-D.ietf-wimse-identifier: wimse-id
  I-D.ietf-wimse-workload-creds: wimse-creds
  I-D.ietf-oauth-transaction-tokens: trat
  I-D.ietf-oauth-identity-chaining: id-chain
  I-D.ietf-oauth-attestation-based-client-auth: oauth-attest
  I-D.ietf-oauth-identity-assertion-authz-grant: id-jag
  I-D.ietf-oauth-status-list: status-list
  I-D.ietf-oauth-spiffe-client-auth: spiffe-oauth
  RFC6749: oauth2
  RFC8693: token-exchange
  RFC9396: rar
  RFC9449: dpop

informative:
  I-D.birkholz-verifiable-agent-conversations: vac
  I-D.mcguinness-oauth-actor-profile: actor-profile
  I-D.mw-oauth-actor-chain: actor-chain
  I-D.johansson-direct-presentation-arch: direct-presentation

--- abstract

This document describes an architecture for auditing of agent-driven interactions on the Internet.
Autonomous and semi-autonomous software agents, including those based on artificial intelligence, increasingly act on behalf of users, organizations, and services.
Existing auditing mechanisms often capture isolated system events but do not consistently represent delegation relationships, user intent, or evolving authorization.
In agent-driven systems, auditability requires linking intent, delegation, authorization, and execution.
The proposed architecture enables this through distributed audit record generation, propagation of audit context, optional attestation, and additonal logging for transparency.


--- middle

# Introduction

Autonomous and semi-autonomous software agents, including those based on large language models (LLMs) and similar non-deterministic systems, are deployed to take consequential actions on behalf of users, organizations, and services across the Internet.
These agents interact across administrative and trust domains, delegate tasks and authority to other agents or tools, and initiate consequential actions without per-step human oversight.
The question of whether the recorded actions of an agent faithfully represent what the agent actually did has acquired new urgency.

Autonomous agents may run long-lived workflows without tight user interaction or may be very short-lived, e.g. for a delegated sub-tasks.
Agents may be authenticated to several services, request step-up approval from a human, spawn further sub-agents, and produc records that long outlive its own process.
Existing auditing mechanisms often capture isolated system events but do not consistently represent delegation relationships, user intent, or evolving authorization.
In agent-driven systems, auditability requires linking intent, delegation, authorization, and execution.

This document describes an architecture that enables this form of auditing through three layers:
- distributed audit record generation by each participant,
- propagation of audit context across protocol interactions, and
- optional attestation and independent third-party logging for transparency that provides verifiable assurances about the content and origin of records.

The architecture in this document identifies roles and their duties, describes the classes of interaction that must be audited, and discusses an example data model to make audits interoperable across vendors, domains, and time. While AI agents is the driving use case, as further discussed in {{usecases}}, the proposed architecture provides a general purpose auditing system for complex workloads delegated authorization chains, potentially over the Internet.

Two principles frame the rest of this document:

1. Agents participate in *two distinct classes of interaction* that must each be auditable: user-facing interactions (prompts, approvals, human-in-the-loop confirmations) and system-facing interactions (API calls, tool invocations, delegation to other agents or services).
   Effective auditing requires linking user intent to resulting system actions across protocol and administrative boundaries.

2. Unlike traditional delegated workflows in which authorization transitions are explicit and predefined, complex agent systems introduce dynamic, fine-grained authorization changes that arise during execution and are driven by agent decisions, sub-agent delegation, and human interaction.
   Auditing must therefore capture authorization as a *time-evolving state* and must correlate transitions across interactions and domains by maintaining common context.

## Relationship to Other IETF Work

The architecture is designed to compose existing IETF building blocks to make verifiable what these layers already do rather than redefining them.

Remote attestation follows RATS {{-rats-arch}} supplies the environmental evidence of the record's origin: RATS Evidence, Attestation Results, and Endorsements are reused verbatim as the vocabulary for environmental claims about audit-record producers.

Transparency follows SCITT {{-scitt-arch}}that makes a record's existence later un-deniable: SCITT Signed Statements, Receipts, and Transparent Statements are the canonical artifacts, and SCITT-compatible Transparency Services are the canonical substrate for non-repudiable custody.

The Verifiable Agent Conversations data model {{-vac}} could be utilizd as an Interaction Record.

HTTP may be used as the transport mechanism for conveying audit context alongside requests. JSON-based formats, including JWT and COSE, can proide representations for audit records and attestations, along with mechanisms for cryptographic protection.

Authority and delegation are based on by OAuth 2.0 {{-oauth2}}, Token Exchange {{-token-exchange}}, Transaction Tokens {{-trat}}, Identity Chaining {{-id-chain}}, Identity Assertion Authorization Grants {{-id-jag}}, RAR {{-rar}}, attestation-based client authentication {{-oauth-attest}}, DPoP {{-dpop}}, Status Lists {{-status-list}}, and SPIFFE client authentication {{-spiffe-oauth}}.

Workload identity follows WIMSE {{-wimse-arch}}, which this document specializes for more complex agents like AI agents.

The three principal acting roles as described in the next section and shown in {{fig-arch}} have parallel counterparts in OAuth and WIMSE; a deployment may use both.
The following table shows a simple mapping:

| Actor View (this document) | OAuth view {{-oauth2}} | WIMSE view {{-wimse-arch}} |
|---|---|---|
| User | Resource Owner | Principal of a run; may also be a Workload in machine-only runs |
| Agent | OAuth Client | Workload (with `sub_profile=ai_agent`) presenting a Workload Identity Credential |
| External Service / Tool | Resource Server | Service-side Workload or external endpoint of another Trust Domain |
<!--{: #tbl-role-mappings title="OAuth and WIMSE role mappings for the three principal acting roles."}-->

The auditing layer adds the cross-layer artifacts (audit records and context, agent interaction records, attestation references, and transparency receipts) that turn isolated layer events into a verifiable audit trail.

# Motivating Use Cases {#usecases}

The need for interoperable auditing of agent-driven systems arises from both regulatory requirements and user trust expectations.
The following examples highlight scenarios where traditional logging is insufficient and where an explicit auditing architecture for agents provides value.

The proposed auditing architecture provides traceability of data access and enables reconstruction of the full chain from user intent to execution, including the full delegation chain that might change dynamically and authorization decisions, providing the desired verifiable audit trail suitable for compliance and review.

## Financial Transactions by Agents

Agents may execute financial operations such as payments or procurement actions on behalf of users, often involving multiple systems.
Regulatory frameworks (e.g., SOX, PSD2) require that transactions be attributable, authorized, and auditable.

Traditional logs typically capture only execution events (e.g., “payment executed”), without clearly linking them to user intent, approvals, or delegation steps.
This makes it difficult to verify whether actions were properly authorized.

## Long-Running Autonomous Agents

Some agents operate continuously over extended periods, making decisions and performing actions based on changing conditions.
For example, a procurement agent may manage ordering and inventory over days or weeks.
In such scenarios, authorization evolves over time due to policy changes, approvals, or context-dependent decisions.
Delegation paths may also change dynamically.

## Everyday Agent Interaction

Even simple consumer use cases benefit from improved auditing.
For example, an assistant may book a restaurant on behalf of a user by selecting a venue and interacting with a booking service.
If the result is unexpected, traditional logs provide limited insight into how the decision was made or which services were involved.

## Data Sharing and User Trust

Agents often access and share (sensitive) user data with external services.
For example, an assistant may send a user’s address to a delivery provider.
Without detailed auditing, it is difficult to verify what data was accessed, what was shared, and with whom. Unintended disclosure can undermine user trust.

# Architectural Overview {#overview}

{{fig-arch}} shows a high level end-to-end view of the proposed architecture: three principal acting roles produce records about their own behaviour. These records flow into supporting services that store, attest, and expose them to consumers that can verify their authenticity and provenance.

~~~ aasvg
+---------------+        +---------------+        +---------------+
|     User      |        |     Agent     |        |   External    |
|               +------->+               +------->+   Services /  |
+---------------+        +---------------+        |   Tools       |
         |                        |               +---------------+
         |                        |                       |
         v                        v                       v
   Interaction Records     Agent Records           Service Records
         |                        |                       |
         +------------------------+-----------------------+
                                  |
                                  v
       +--------------------------+---------------------------+
       |                   Auditing Service                   |
       |  +--------------+  +--------------+  +------------+  |
       |  |  Attestation |  |   Audit      |  |Transparency|  |
       |  |              |  |   Store      |  |Log         |  |
       |  +--------------+  +--------------+  +------------+  |
       +------------------------------------------------------+
                                  |
                                  v
                +-----------------+-------------------+
                |    Audit Consumers / Verifiers      |
                +-------------------------------------+
~~~
{: #fig-arch title="Roles view: principal acting roles, auditing services, and the records that flow among them."}

The proposed architecture enables interoperable auditing of agent-driven interactions by combining distributed audit record generation, audit context propagation, and optional attestation and transparency logging.
Audit information is produced by multiple actors operating across administrative domains and is later reconstructed and validated by audit consumers through a shared audit context and may be accompanied by attestations.

Audit records are generated distributively.
Each principal acting role (user, agent, service/tool) records its own behaviour rather than relying on a central observer.
These records are linked via a propagated audit context with a coherent trail:
Interaction Records flow are generated by the User; Action and Delegation Records flowby the Agent; Service Records flow by external Services/Tools.
Each actor may produce mutiple records to, e.g., audit actions, delegation, or authorization changes.
Distributed record generation limits the trust placed in any single point.
Two trust mechanisms are composed (attestation and transparency logging).
An auditing system may select either, both, or neither according to the strength of evidence its Auditors require.

The auditing services (Attestation, Audit Store, and Transparency Log) are distinct from the interaction and authorization layers that drives the three acting roles.
They can be or need to be operated by different parties dependeing on trust requirements of auditor.
Attestations may be directly provided by the producer or supplied on request by an Attestation Service.
An Audit Store canonicalises observable agent signals into the records and exposes them to Audit Consumers and Verifiers.
The Store is logically distinct from the Agent it records, because an agent that records itself can produce useful telemetry but cannot, by itself, deliver non-repudiation to a third party.
Transparency receipts provided by the Transparency Log are embedded in or referenced from the records.

## Audit Records and Context

Audit records are generated independently by participating actors and reflect different perspectives of an interaction.
User-facing systems produce records capturing intent, such as prompts or approvals.
Agents produce records describing decisions, actions, and delegation steps.
Services produce records reflecting execution outcomes.

These records are linked through a shared audit context that is propagated across protocol interactions.
This context carries identifiers such as a trace identifier and references to prior events, enabling reconstruction of a causal chain.
It also carries identity information, including the acting entity and, where applicable, the entity on whose behalf the action is performed.

Transport mechanisms such as HTTP are used for propagating this context, for example via headers or message metadata.
Existing approaches such as W3C's trace context propagation can serve as a basis but require extension to include identity, delegation, and authorization state.
As such, correlation is not performed by a centralized component.
Instead, it emerges from consistent use of shared identifiers and structures across all participants.

## Attestation Model

Audit records may include attestation evidence that provides verifiable assurances about their content.
An attestation binds a statement to a cryptographic identity and allows relying parties to validate claims about events, delegation, or execution.

Attestations are generated at or on behalf of the entity asserting a claim and are associated with audit records at creation time.
Attestation can have different levels of assurance, depending on the type of origin (ranging from self-assertions to RATS evidence).
An agent or service may self-attest its actions using mechanisms such as JSON Web Signatures or COSE-based signatures.
In other cases, the producer interacts (taking on the role of a RATS Attester) with an external remote attestation service (i.e., a RATS Verifier) to obtain an additional statement (an Attestation Result).

Further, the transparency service may attest that a record has been recorded in an append-only log.
And the audit store may provide additional attestations, such as timestamping or proof of inclusion.
But this does not replace attestations generated by record producers.

## Identity Substrate

An Agent is treated as a specialisation of a Workload {{-wimse-arch}}, with a Workload Identifier {{-wimse-id}} scoped within a Trust Domain and a Workload Identity Credential (e.g., a WIT {{-wimse-creds}}) bound to a key the workload generates and retains.
The credential is never a bearer token and is normally short-lived.
For auditing it is benefical if an Agent also carries a role profile (per {{-actor-profile}}'s `sub_profile` convention) so that downstream parties can distinguish an AI-driven Workload from a human-operated client or a traditional service.
The role-profile vocabulary is the subject of a separate specification (WI-2 in {{work-items}}).

# Roles {#roles}

A role is a function, not a deployment unit.
The same entity may take on several roles.

## Principle Agent Interaction Actors

The User is the human or organisation on whose behalf an Agent acts.
Where the User is a natural person they are also the OAuth Principal of the run: the `sub` of any token issued for the run and the original authorizing party in any delegation chain.
On the audit layer the User's duty is to issue intent in a verifiable form--prompt, approval, or signed grant--and to respond to step-up escalations.

An Agent is a Workload {{-wimse-arch}} whose behaviour is driven, in whole or in part, by a non-deterministic decision process (typically an LLM).
Agents operate within the authorization they were issued, propagate the upstream Principal's identity and the delegation context
unmodified except where an exchange explicitly authorises a change, and emit observable signals, such as prompts, actions, tool calls, sub-agent invocations, terminations, that an Audit Store can canonicalise.
Where an Agent signs records itself, it signs with a key bound to its Workload Identity Credential, never with a long-lived shared key.
