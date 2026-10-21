---
title: "Security Requirements for AI Agents"
abbrev: "Agent Security Requirements"
category: info

docname: draft-ni-a2a-ai-agent-security-requirements-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - AI Agent
 - Provisioning
 - Authentication
 - Authorization

venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG

  github: "liuchunchi/draft-ni-a2a-ai-agent-security-requirements"
  latest: "https://liuchunchi.github.io/draft-ni-a2a-ai-agent-security-requirements/draft-ni-a2a-ai-agent-security-requirements.html"

author:

  -
    fullname: Yuan Ni
    organization: Huawei
    email: niyuan1@huawei.com
  -
    fullname: Chunchi Peter Liu
    organization: Huawei
    email: liuchunchi@huawei.com
  -
    fullname: Qiangzhou Gao
    organization: Huawei
    email: gaoqiangzhou@huawei.com
  -
    fullname: Zhenbin Li
    email: robinli314@163.com

  -
    fullname: Yuna Xu
    organization: CESI
    email: xuyuna@cesi.cn

normative:
  RFC2119:
  RFC8174:

informative:
  I-D.ietf-oauth-identity-chaining:
  I-D.ietf-oauth-transaction-tokens:
  I-D.ni-wimse-ai-agent-identity:
  I-D.yao-dawn-agent-discovery-architect:
  I-D.liu-oauth-cross-domain-txn-token:
  I-D.ni-oauth-batch-authorization-delegation:

...

--- abstract

This document discusses security requirements for AI agents, covering different stages of security interactions. These include provisioning, registration, discovery, cross-domain interconnection, and access control.

--- middle

# Introduction

With the widespread application of agentic AI technology across various business scenarios, its security issues have become increasingly prominent.

This document aims to provide an architecture addressing security requirements across different stages of interactions in agentic AI use cases. These include provisioning, registration, discovery, cross-domain interconnection, and access control. This document establishes a starting point to guide discussions on agentic AI security design, development, and implementation.

The target audience of this document is IETF security experts who wish to understand AI agents' behavioral patterns and evaluate whether the proposed security requirements are worthy of further security designs.

# Architecture

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

~~~~
                 +----------------+
                 |  Master Agent  |
                 +-^--------------+
                 +-+--------------+
                 | |  Firewall    |
                 +-+--------------+
                   |
-------------- (2) | Inter-domain--------------
                   |
+------------------+---------------------------------+
|                +-+--------------+   Intra-domain   |
|                | |  Firewall    |                  |
|                +-+--------------+                  |
|                +-v--------------+                  |
|        +-------+  Master Agent  |                  |
|   (3)  |       +----------------+                  |
|        |                              (1)          |
|   +----v-----+    +----------+    +----------+     |
|   |  Agent   +---->  Agent   +---->  Agent   |     |
|   +----+-----+    +----------+    +----+-----+     |
|        |                               |           |
|     +--v---------+           +---------v---+       |
|     |    DB      |           |     API     |       |
|     +------------+           +-------------+       |
+----------------------------------------------------+
~~~~
*Figure 1. Architecture of Agent Security Control and Management*

The architecture of agent security control and management is illustrated in Figure 1. There are three types of security interactions, in sequential order:

1. Provisioning, Registration, and Discovery: Creating agent identities, establishing initial trust, provisioning agent secrets and credentials, and onboarding agents to enable discovery.

2. Cross-Domain Interconnection: Enabling secure, authenticated communication between agents across different trust domains.

3. Access Control: The Master Agent validates both intra-domain and inter-domain access tokens, creates internal workflows, and manages different credentials for heterogeneous systems.

Therefore, the architecture includes four components:

1. Firewall: A network security device designed to monitor, filter, and control incoming and outgoing network traffic based on predetermined security rules.

2. Master Agent: The central orchestrating entity that manages multi-agent operations, including cross-domain communication, workflow coordination, credential management, and security policy management.

3. Agents: Autonomous software entities deployed in various domains to perform specific tasks.

4. Heterogeneous Systems: API endpoints, microservices, tools, and databases.

The above architecture is presented from the perspective of a service flow. From the identity management perspective, we recommend reusing existing IETF work such as WIMSE. The draft {{I-D.ni-wimse-ai-agent-identity}} discusses the applicability of WIMSE to agentic AI.

# Provisioning, Registration, and Discovery

Figure 2 shows the diagram of provisioning and registration, which includes Agent Credential Authority (ACA) and Agent Registry Service (ARS):

1. ACA (Agent Credential Authority): A trusted third party that issues and manages credentials for agents. Credential formats include, but are not limited to, X.509 certificates and identity tokens.
2. ARS (Agent Registry Service): A system responsible for agent identity registration and discovery.

~~~~
+-------------------------------------------+
|                                           |
| +-----------------+  +------------------+ |
| |Agent Credential |  |  Agent Registry  | |
| |Authority (ACA)  |  |  Service (ARS)   | |
| +--------^--------+  +---------^--------+ |
|          |                     |          |
|         ++---------------------++         |
|         ||     +----------+    ||         |
|         |+----->   Agent  +----+|         |
|         |      +----------+     |         |
|         |                       |         |
|         |   device/container    |         |
|         +-----------------------+         |
|                                           |
+-------------------------------------------+
~~~~
*Figure 2. Diagram for Provisioning and Registration*

## Identity Provisioning and Management

Identity provisioning and management involve creating and assigning a verifiable digital identity to an agent.

* Initial Trust Establishment: Initial trust can be established through one or more of the following trust anchors, including, but not limited to: a manufacturer-embedded immutable credential like an IDevID certificate; a hardware root of trust like a Trusted Platform Module (TPM) or Hardware Security Module (HSM); identity documents like an AWS Instance Identity Document or an Azure Managed Service Identity token. This step verifies the agent's execution environment (device, container, etc.) as trustworthy, allows the device or container to join the network, thereby enabling secure operations for all subsequent steps.

* Credential Request: During a credential request, the agent must provide multiple proofs of its legitimacy, could include, for example, but not limited to:
  * Proof of Possession (PoP): A Certificate Signing Request (CSR) or other PoP forms signed with the agent's private key, demonstrating that the agent holds the private key corresponding to the requested identity.
  * Remote Attestation Evidence or Result: A set of security-relevant claims about the Target Environment submitted to a RATS Verifier (which could be the ACA), providing information about operational status, health, configuration, or construction.
  * AI Bill of Materials (AIBOM): A comprehensive inventory that details the agent's supply chain, including models, datasets, configurations, dependencies, and related infrastructure. This prevents the use of vulnerable AI components.
  * Provider Endorsement: A digital signature or credential from the Agent Provider, ensuring the agent originated from a trusted source.
  * Identity Binding: A cryptographic binding to a specific human user or an organizational role to specify on whose behalf the agent operates and its authorized scope.

* Credential Issuance: The ACA validates proofs and requests from the above two steps, if passed, it issues an agent-specific credential that may include its owner or requester identity, capabilities, locator, acceptable validation methods for the ARS.

* Credential Lifecycle Management: The ACA not only issues credentials but also defines and enforces revocation policies. These policies are triggered by specific events, such as a detected security compromise, the agent's scheduled decommissioning, or key rotation.

## Secret Management

AI agents SHOULD NOT have direct access to secrets due to new threats like Prompt Injection. AI Agents SHOULD reuse secret management modules on the platform it operates on, for example, cloud secret managers or TEE/keystore/keychains on smart devices. Best practices like secret/credential generation, rotation and revocation apply. Agents SHOULD only obtain temporary access tokens or signed messages via a secure API or other kind of trusted intermediary. Guardrails also apply for general secret information exfiltration prevention.

## Agent Registration

After receiving a credential from the ACA, the agent then sends it to the ARS to authenticate itself and start the registration process.

* Authentication: The ARS must verify the legitimacy of the credential submitted by the agent. It must be signed or otherwise endorsed by the ACA.

* Registration: The ARS then checks if the information signed by the ACA, such as the agent's capabilities, exactly matches the registration request sent by the agent. Upon successful validation, the ARS assigns the agent a unique identifier and establishes an agent record that links the identifier to its attributes.

* Record Management: The ARS removes expired credentials and synchronizes with the ACA to ensure timely revocation of credentials, preventing the use of invalid or compromised credentials.

## Agent Onboarding

Agent onboarding differs between campus and cloud environments. On campus, agents use protocols like EAP-TLS for network access. In the cloud, the process involves injected sidecars, which register agents to the central service mesh registry automatically to enable communication and management.

## Agent Discovery

After agent onboarding, the discovery process enables entities (e.g., a human user, an agent, etc.) to find and connect with registered agents.

* Authentication: The ARS must authenticate the entity initiating the discovery request. The requester is required to present a valid identity credential.

* Capability Filtering and Matching: The ARS performs dynamic filtering based on the requester’s identity and query and returns only agent records relevant to the query, enforcing the principle of least privilege at the discovery layer.

* Cross-Domain Agent Discovery: The ARS should support the discovery of trust bundles for verifying agent identities across administrative domains. {{I-D.yao-dawn-agent-discovery-architect}} proposes a DNS-like discovery architecture in which Agent Registries manage domain-level trust bundles, enabling agent identities registered in one administrative domain to be verified across domains.

# Cross-Domain Interconnection

## Cross-Domain Identifier Interoperability

Different domains may use distinct identifier schemas. Possible methods include:

* Pre-configured schema translation
* Cross-domain identifier synchronization
* A universal parsing framework or system

## Secure Cross-Domain Transmission

A mutual TLS (mTLS) connection is established between the external requesting agent and the master agent. The master agent terminates the mTLS connection and parses application-layer requests. In this case, the master agent functions as an OAuth resource server and manages internal task orchestration.

## Authenticating External Calls

The master agent then verifies the identity of the requesting agent, and whether or not it has permission to the requested service or agent. Different authentication methods might be possible:

* API keys
* Username-password
* Pre-shared secrets
* Assertions (for example, JWT Authorization Grants as discussed in {{I-D.ietf-oauth-identity-chaining}})

These methods may also be combined using AND/OR logic. During this process, the master agent might be able to identify the caller endpoint type:

* Human user via browser or app
* Human user via API
* AI agents
* Hardware or equipment via an IoT API

## IAM Integration

Since the agent may inherit its access rights from its owner or user, when authenticating requests, the validation might require integration of IAM systems for redirected verification.

# Access Control

## Authorization Handling

 The master agent acts as the OAuth 2.1 resource server and a Policy Enforcement Point (PEP). Its responsibilities are as follows:

* Token Validation: The master agent must validate access tokens as described in OAuth 2.1 Section 5.2. If validation fails, it must respond according to OAuth 2.1 Section 5.3 error handling requirements.

* Fine-Grained Policy Enforcement: The master agent serves as a PEP that queries a PDP (Policy Decision Point), such as Open Policy Agent (OPA), to evaluate the requester’s access request. The PDP functions by taking the master agent's query, pre-configured policies (supporting RBAC, ABAC, ReBAC models, etc.), and data as inputs to decide whether the requester is authorized for its intended action. The PDP then returns the final decision to the master agent for enforcement.

## Authorization Models

In enterprise environments, Role-based Access Control (RBAC) Attribute-based Access Control (ABAC) or Adaptive Access Control (AdBAC) are different access control models used in practice. Regarding access control models, there are 2 ways forward:

* whatever the authorization model used in the enterprise itself applies to AI Agents. This leaves 2 cases possible:
   * The agent carries the identity and inherit access rights from its owner (a human or a department). Carrying such human identity will help security control points make decisions with sufficient context, and to the discretion of its internal security policy plus access control model.
   * The agent does not carry the identity from its owner. It carries independent security contexts rich enough for access control.
* AI Agents require a new authorization model completely.


This section would require more discussion for best current practices.

## Authorization Chaining Across Domains

In an agentic AI use case, a request may traverse multiple master agents in multiple trust domains before completing. It is common that the requesting agent from domain A needs to access the master agent of domain B. During this process, the following information should be preserved:

* Original requesting agent identity
* Authorization context
  * Scope
  * Resource
  * Audience
  * Grant type
  * Assertion
* Agent-to-Agent Context

The Identity Chaining specification {{I-D.ietf-oauth-identity-chaining}} supports the propagation of identity and authorization information across domains. Building on this, the Cross-Domain Transaction Tokens draft {{I-D.liu-oauth-cross-domain-txn-token}} explores how workflow context can be securely transferred across domains to support the issuance of Transaction Tokens for downstream workflows.

## Converting to Internal Workflow

Complex tasks often require multi-agent collaboration. The master agent receives, parses, and extracts the original job request from the external requesting agent, then creates sequential workflows or parallel calls. This requires the master agent to have information about all callable internal API assets, agent capabilities, etc.

* Batch Authorization Delegation: Parallel execution by multiple downstream agents may require different permissions for each subtask. The Batch Authorization Delegation draft {{I-D.ni-oauth-batch-authorization-delegation}} proposes requesting multiple fine-grained permissions in a single authorization request, specifying the designated agent for each permission, and enabling each agent to obtain an appropriately scoped access token through token exchange.

* Downscoping: If the master agent intends to use a workflow, it extracts the original caller's identity and authorization context, and initiates a new internal workflow. It should follow the principle of least privilege through Transaction Token downscoping, as specified in {{I-D.ietf-oauth-transaction-tokens}}. The access rights to each downstream workload decrease.

* Agent-to-Agent Context: The agent-to-agent context and intent of the original requester must be preserved and propagated throughout the workflow to avoid authorization drift and context poisoning.

## Interoperability for Heterogeneous Systems

Within a domain, there might exist different types of heterogeneous systems or legacy systems that require different authentication methods. They could be API endpoints, microservices, tools or databases. The exact authentication methods are determined by the service itself, for example:

* Identity tokens
* API keys
* Pre-shared secrets
* Username-password
* X.509 certificates

As a result, the master agent also works as an intermediary credential manager that converts the formats, scopes, identity of the credential, bridging the gap between heterogeneous systems and platforms.

Examples include:

* Static secrets (API keys) to be exchanged to short-lived, on demand credentials (identity tokens)

## Zero Trust Analysis

The above information can be used as rich context that allows zero trust access control. There are three additional aspects that can be implemented to enhance the zero trust framework:

* Remote Attestation Results:  For the PEP at the master agent or the internal resource server, Remote attestation results could also be part of the inputs, which could include the following information:

  * Roots of Trust (RoT) and trust anchors
  * Identifiers
  * Affiliations
  * Posture assessment results
  * Capabilities

* Continuous Observability: The system should utilize OpenTelemetry (OTel) to track each call across agents, sending OTel’s telemetry, which records call frequency, error rates, and behavioral anomalies, etc. to the PDP for real-time assessment.

* Microsegmentation: Based on the telemetry data, PDP can issue software-defined security policies to PEP at the perimeter of each segment to enforce microsegmentation, in order to prevent lateral movement of security risks. Possible granularity of microsegmentation includes:
  * per IP segment/subnet
  * per workload
  * per tags and attributes (of workload), etc.

# IANA Considerations

This document has no IANA actions.

--- back

# Revision History
{:numbered="false"}

## Changes from -00 to -01
{:numbered="false"}

* Expanded the security architecture to include agent discovery.
* Added requirements for secret management and agent discovery.
* Refined authorization models, cross-domain authorization, and internal workflow handling.
* Expanded zero trust considerations, including microsegmentation.

## Changes from -01 to -02
{:numbered="false"}

* Corrected editorial, grammatical, and reference issues.
* Updated author information to match the published -01 version.
* Added cross-domain agent discovery and trust bundle considerations, referencing the agent discovery architecture draft.
* Added cross-domain workflow context propagation using Transaction Tokens.
* Added batch authorization delegation for parallel multi-agent tasks.
* Removed the obsolete A2A profile reference.
