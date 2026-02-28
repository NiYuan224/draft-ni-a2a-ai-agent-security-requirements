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

normative:
  RFC2119:
  RFC8174:
informative:
  I-D.draft-ietf-oauth-identity-chaining-06:
  I-D.draft-tulshibagwale-oauth-transaction-tokens-05:
  I-D.draft-ni-wimse-ai-agent-identity-01:

...

--- abstract

This document discusses security requirements for  AI agents, covering different stages of security interactions. These include provisioning, registration, cross-domain interconnection, and access control.
--- middle

# Introduction
With the widespread application of agentic AI technology across various business scenarios, its security issues have become increasingly prominent.

This document aims to provide an architecture addressing security requirements across different stages of interactions of Agentic AI use cases. These includes provisioning, registration, cross-domain interconnection, access control, and so on. This document establishes a starting point to guide Agentic AI security design, development, and implementation consideration discussions.

The target audience of this document would be IETF security experts that wish to understand AI Agent's behaviorial patterns, so to evaluate if the proposed security requirements are worthy of further security designs.


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

The architecture of agent security control and management is illustrated in Figure 1. There are four types of security interactions, in a sequential order:

1. Provisioning and Registration: Creating agent identity, establishing initial trust, provisioning agent secrets and credentials.
2. Cross-domain Interconnection: Enabling secure, authenticated communication between agents across different trust domains.
3. Access Control: The Master Agent validates both intra-domain and inter-domain access tokens, creates internal workflow and manages different credentials for heterogeneous systems.


Therefore, the architecture includes four components:

1. Firewall:  A network security device designed to monitor, filter, and control incoming and outgoing network traffic based on predetermined security rules.

2. Master Agent: The central orchestrating entity that manages multi-agent operations, including cross-domain communication, workflow coordination, credential management, and security policy management.

3. Agents: Autonomous software entities deployed in various domains to perform specific tasks.

4. Heterogeneous systems:  API endpoints, microservices, tools, and databases.

The above architecture is from the perspective of a service flow. From the identity management perspective, we recommend reusing IETF works like WIMSE. This draft {{I-D.draft-ni-wimse-ai-agent-identity-01}} discusses WIMSE applicability to Agentic AI.

# Provisioning and Registration

Figure 2 shows the diagram of provisioning and registration, which includes Agent Credential Authority(ACA) and Agent Registry Service(ARS):

1. ACA (Agent Credential Authority): A trusted third party that issues and manages credentials for agents. Credential formats include but not limited to: X.509 certificates, identity tokens, etc.
2. ARS (Agent Registry Service): A system responsible for agent identity registration and discovery-matching.

~~~~
+-------------------------------------------+
|                                           |
| +-----------------+  +------------------+ |
| |Agent Credential |  |  Agent Registry  | |
| |Authority (ACA)  |  |  Service(ARS)    | |
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
Identity provisioning and management are the process of creating and assigning a verifiable digital identity to an agent.

* Initial Trust Establishment: Intial trust can be established through one or more of the following trust anchors, including, but are not limited to: a manufacturer-embedded immutable credential like an IDevID certificate; a hardware root of trust like a Trusted Platform Module (TPM) or Hardware Security Module (HSM); identity documents like an AWS Instance Identity Document or an Azure Managed Service Identity token. This step verifies the agent's execution environment (device, container, etc.) as trustworthy, allows the device or container to join the network, thereby enabling secure operations for all subsequent steps.

* Credential Request: During a credential request, the agent must provide multiple proofs of its legitimacy, such as a Certificate Signing Request or a Proof of Possession by signing with the corresponding private key, as well as remote attestation by collecting and submitting evidence to a RATS (Remote Attestation Procedures) Verifier. Additionally, to define the agent's operational scope, the request should incorporate user identity context, binding the credential to a specific human user or an organizational role.

* Credential Issuance: The ACA validates proofs and requests from the above two steps, if passed, it issues an agent-specific credential that may include its owner or requester identity, capabilities, locator, acceptable validation methods for the ARS.

* Credential Lifecycle Mangement: The ACA not only issues credentials but also defines and enforces revocation policies. These policies are triggered by specific events, such as a detected security compromise, the agent's scheduled decommissioning, or a key rotation.

## Secret Management 

AI Agents SHOULD NOT have direct access to secrets due to new threats like Prompt Injection. AI Agents SHOULD reuse secret management modules on the platform it operates on, for example, cloud secret managers or TEE/keystore/keychains on smart devices. Best practices like secret/credential generation, rotation and revocation apply. Agents SHOULD only obtain temporary access tokens or signed messages via a secure API or other kind of trusted intermediary. Guardrails also apply for general secret information exfiltration prevention.

## Agent Registration
After receiving a credential from the ACA, the agent then sends it to the ARS to authenticate itself and start the registration process.

* Authentication: The ARS must verify the legitimacy of the credential submitted by the agent. It must be signed or otherwise endorsed by the ACA.

* Registration: The ARS then checks if the information signed by the ACA, such as the agent's capabilities, exactly matches the registration request sent by the agent. Upon successful validation, the ARS assigns the agent a unique identifier and establishes an agent record that links the identifier to its attributes.

* Record Management: This step automatically removes expired credentials and synchronizes with the ACA to ensure timely revocation of credentials, preventing the use of invalid or compromised credentials.

## Agent Onboarding
Agent onboarding differs between campus and cloud environments. On campus, agents use protocols like EAP-TLS for network access. In the cloud, the process involves injected sidecars, which register agents to the central service mesh registry automatically to enable communication and management.

# Cross-Domain Interconnection

## Cross-Domain Identifier Interoperability
Different domains may use distinct identifier schemas. Possible methods include:

* pre-configured schema translation
* cross-domain identifier synchronization
* a universal parsing framework or system

## Secure Cross-Domain Transmission

Mutual TLS (mTLS) connection starts from the external requesting agent to the master agent. The master agent terminates the mTLS connection and parses the application layer requests. In this case, the master agent functions as an OAuth resource server, and manages internal task orchestration.

## Authenticating External Calls
The master agent then verifies the identity of the requesting agent, and whether or not it has permission to the requested service or agent. Different authentication methods might be possible:

* API keys
* Username-password
* Pre-shared secrets
* Assertions (for example, JWT Authorization Grant{{I-D.draft-ietf-oauth-identity-chaining-06}})

which can even be combined with AND/OR logic. During this process, the master agent might be able to identify the caller endpoint type:

* human user via browser or app
* human user via API
* AI agents
* Hardware or equipment via an IoT API

## IAM Integration

Since the agent may inherit its access rights from its owner or user, when authenticating requests, the validation might require integration of IAM systems for redirected verification.

# Access Control

## Authorization Handling

The master agent acts as the role of OAuth 2.1 resource server. It must validate access tokens as described in OAuth 2.1 Section 5.2. If validation fails, it must respond according to OAuth 2.1 Section 5.3 error handling requirements.

## Authorization Models

In enterprise situation, Role-based Access Control (RBAC) Attribute-based Access Control (ABAC) or Adaptive Access Control (AdBAC) are different access control models used in practice. Regarding access control models, there are 2 ways forward:

1. whatever the authorization model used in the enterprise itself applies to AI Agents. This leaves 2 cases possible:
   1. The agent carry the identity and inherit access rights from its owner (a human or a department). Carring such human identity will help security control points make decisions with sufficient context, and to the discretion of its internal security policy plus access control model.
   2. The agent does not carry the identity from its owner. It carries independent security contexts rich enough for access control.
2. AI Agents require a new authorization model completely.

This section would require more discussion for best current practices.

## Authorization Chaining Across Domains

In an agentic AI use case, a request may traverse multiple resource servers in multiple trust domains before completing. It will be common that the requesting agent from domain A needs to access the resource server (master agent) of domain B. During this process, the following information should be preserved:

* Original requesting agent identity
* Authorization context
  * Scope
  * Resource
  * Audience
  * Grant type
  * Assertion

The current best practice is {{I-D.draft-ietf-oauth-identity-chaining-06}}.

## Converting to Internal Workflow

* Workflow Generation: Complex tasks often require multi-agent collaboration. The master agent receives, parses, and extracts the original job request from the external requesting agent, then create sequential workflows or parallel calls. This requires the master agent to have information of all callable internal API assets, agent capabilities, etc.

* Downscoping: If the master agent intends to use a workflow, it extracts the original caller's identity and authorization context, and initiates a new internal workflow. It should follow the current least privilege best practice of downscoping-Transaction Tokens as specified in {{I-D.draft-tulshibagwale-oauth-transaction-tokens-05}}. The access rights to each downstream workload decrease.

## Interoperability for Heterogeneous Systems

Within a domain, there might exist different types of heterogeneous systems or legacy systems that require different authentication methods. They could be API endpoints, microservices, tools or databases. The exact authentication methods are determined by the service itself, for example,

* identity tokens
* API keys
* pre-shared secrets
* username-passwords
* X.509 certificates, etc.

As a result, the master agent also works as an intermediary credential manager that converts the formats, scopes, identity of the credential, bridging the gap between heterogeneous systems and platforms.

Examples include:

* Static secrets (API keys) to be exchanged to short-lived, on demand credentials (identity tokens)

## Zero Trust Analysis

The above information can be used as rich context that allow zero trust access control. Remote attestation results of the requesting agent could also be part of access policy decision point's inputs. Remote attestation results of the requesting agent could include the following information:

* RoT and trust anchors
* Identifiers
* Affiliations
* Posture assessment results
* Capabilities

The overall information will be used as input of Policy Engine (PE) and Policy Decision Point (PDP).


## Microsegmentation

Microsegmentation may be enforced to prevent lateral movement of security risks. Possible granularity of microsegmentation includes:

* per IP segment/subnet
* per each workload
* per tags and attributes (of workload), etc.


There should be policy enforcement points (PEP) at the perimeter of each segment. Each PEP can receive software-defined security policies issued by PE/PDP.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
