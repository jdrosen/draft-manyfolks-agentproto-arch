---
title: "Terminology, Framework and Architecture for the Agent Communications Protocol (ACP)"
abbrev: "AgentProto Arch"
category: info

docname: draft-manyfolks-agentproto-arch-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "Agent Communication Protocols"
keyword:
 - AI
 - Agent Protocols
 - Agentic
venue:
  group: "Agent Communication Protocols"
  type: "Working Group"
  mail: "agentproto@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/agentproto"
  github: "jdrosen/draft-rosenberg-agentproto-requirements"
  
author:
 -
    fullname: "Jonathan Rosenberg"
    organization: jdrosen.net
    email: "jdrosen@jdrosen.net"
 -
    ins: Z. Sarker
    name: Zaheduzzaman Sarker
    organization: Nokia
    email: zaheduzzaman.sarker@nokia.com
  -
    fullname: Tirumaleswar Reddy
    organization: Nokia
    email: k.tirumaleswar_reddy@nokia.com
  -
    name: Kehan Yao
    organization: China Mobile
    email: yaokehan@chinamobile.com
  -
    name: Dapeng Liu
    organization: Alibaba Cloud
    email: max.ldp@alibaba-inc.com
  -
    name: Suresh Krishnan
    organization: Cisco
    email: suresh.krishnan@gmail.com

normative:

informative:

A2A:
    title: "Agent2Agent Protocol Specification"
    author:
      - org: Google
    target: https://a2a-protocol.org/v0.2.5/specification/
    date: 2025
  MCP:
    title: "Model Context Protocol"
    author:
      - org: Anthropic
    target: https://modelcontextprotocol.io/specification
    date: 2024

  anthropic-context-def:
    title: "Effective Context Engineering for AI Agents"
    author: 
      - org: Anthropic
    target: https://www.anthropic.com/research/effective-context-engineering-for-ai-agents
    date: 2025


  RFC3261:

  I-D.rosenberg-aiproto-framework:

  I-D.agentic-ai-usecases-requirements:

  I-D.rosenberg-aiproto-a2t:


--- abstract

The Agent Communications Protocol (ACP) - also known as agentproto - is a new protocol to be developed in the IETF. It is meant to be used between user agent clients and AI agents, or between AI Agents and other AI Agents, or between AI Agents and tools. All three of these use cases - and the many variations therein - all require a session-type protocol which handles lifecycle management, including creation, modification, deletion, and so on. These sessions differ from the type of sessions created by the Session Initiation Protocol (SIP) in several important ways. These sessions can be extremely long lived - running for hours or even days. An example is a lengthy coding session. They can also be highly intermittent with asynchronous responses. All of these mean a new type of session protocol is needed. This document introduces concepts, terminology, reference architecture and the protocol functional blocks for the Agentic Communications Protocol (ACP).  



--- middle

# Introduction {#intro}

The widespread usage of AI Agents on the Internet has introduced new forms of networked communications. User agent clients connect to AI agents running elswehere, AI agents communicate with other AI agents, and AI agents communicate with tools. All three of these use cases - and the many variations therein - all require a session-type protocol which handles lifecycle management, including creation, modification, deletion, cancellation, and so on. These sessions differ from the type of sessions created by the Session Initiation Protocol (SIP) [RFC3261] in several important ways. These sessions can be extremely long lived - running for hours or even days. An example is a lengthy coding session. They can also be highly intermittent with asynchronous responses. The protocol also requires conveyance of many modalities from one side to the other, including text, audio, video, and structured data. This content can be exchanged sporadically, spanning different transport connections and IP hosts. 

All of these mean a new type of session protocol is needed - the Agentic Communications Protocol (ACP). 

This document introduces concepts, terminology, reference architecture and the protocol functional blocks for ACP, which manages agentic dialogs. An agentic dialog is defined as a session between two parties - a principal whose role is to provide input to, and receive output from, an agent - and the agent, which is running a process which has properties which are agentic in nature. The agentic properties that are relevant here are (1) conversational contuity, and (2) driven by natural language in each direction. These concepts are discussed in more detail in this document.

This document breaks ACP down into three functional blocks - agentic dialog management, authorization and delegation, and identity management. As a session-layer protocol, ACP rides ontop of transport protocols. This document identifies requirements of these transport protocols to aid in their selection. 

The use cases and requirements that drive this architecture are described in
{{I-D.agentic-ai-usecases-requirements}} and {{I-D.rosenberg-aiproto-framework}}. The wire formats and protocol elements of the agent communication protocol are outside the scope of this document. This document describes the the functional
decomposition of the reference architecture.

MCP {{MCP}} and A2A {{A2A}} are application-layer protocols
maintained by the Linux Foundation. They use IETF protocols such as HTTP,
OAuth 2.0, and TLS. MCP addresses agent-to-tool interactions, and A2A
addresses agent-to-agent interactions. The agent communication protocol in
this architecture is intended to be usable by these protocols through
well-defined extension points, not to replace them.

# Terminology {#terminology}

Administrative Domain:
: A set of entities, such as agents, tools, and intermediaries, that are
subject to a common authorization policy and a common authority.

Agent Identity:
: Identity information associated with an AI agent, distinct from the
  identity of the user on whose behalf it acts, and used for
  authentication, authorization, and accountability.

AI Agent:
: A software system that uses AI models to complete a task on behalf of a
  principal - which can be a user or another AI agent.

Delegation:
: The act of an AI agent requesting another AI agent to execute a task on
  its behalf.

Agentic Dialog:
: A session between two parties - a principal whose role is to provide input to, and receive output from, an agent - and the agent, which is running a process which has properties which are agentic in nature and result in the maintenance of context. The agentic properties that are relevant here are (1) conversational contuity, and (2) driven by natural language in each direction. 

Context:
: Context is central to the concept of AI Agents. There are many definitions on the Internet. Anthropic defines it as "the set of tokens included when sampling from a large-language model (LLM)." {{anthropic-context-def}}. The goal of ACP is to exchange the input from the principal to allow the agent to manage context, and the output from inference of that context. As a result, an agentic dialog is associated 1-1 with the context it builds. ACP doesn't define how the agent maintains the context; it merely allows for the establishment of an agentic dialog and exchange of data which allows the agent to do so.

Agentic Dialog Identifier:
: An identifier which identifies an agentic dialog and the context associated with it. This identifier survives network interruption or change.


Principal: 
: An entity whose role is to provide input to, and receive output from, an agent. 
The principal can be a human user (as in the user agent to AI agent case noted above), but the principal can also be an AI agent - as in the A2A case.

User:
: A human being who plays the role of the principal. 


Agent:
: An entity whose role is to receive input from a principal, use it to augment its context, and based on an intelligent processing of that context, generate output to be returned to the principal. Typically, an agent is an AI Agent, but ACP doesn't preclude a human from playing that role. For this reason, the ACP protocol uses the term "agent" instead of "AI Agent". 

Modality:
: A category of data exchanged in an agentic dialog, such as text, audio, image, video or structured data. 

Task:
: A unit of work submitted by a principal to an agent, typically expressed through the input provided to the agent which describes the task to be done. In some agentic dialogs, the task may evolve throughout its lifetime. In other agent dialogs, such as that used for communications between an AI agent and a tool, there may be just one. In other agentic dialogs, there is no real notion of task - its just a conversation. 

Tool:
: A service invoked by an AI agent to retrieve data or perform operations.
  A tool is not necessarily an AI agent. However, in some cases a tool can be session-oriented in nature. 

Trust Boundary:
: A boundary between administrative domains.

Intermediary:
: An entity that relays or processes messages between agentic dialog participants.

Message:
: A discrete unit of communication exchanged between agentic dialog participants.






--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
