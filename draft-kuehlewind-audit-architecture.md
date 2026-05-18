---
title: "An Architecture for Auditing AI Agent Delegation and Interactions"
abbrev: "AUDIT Architecture"
docname: draft-kuehlewind-audit-architecture-latest
category: info
consensus: true
submissionType: IETF

ipr: trust200902
area: "Security"
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

TODO Introduction


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back


