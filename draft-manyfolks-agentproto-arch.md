---
title: "Architecture for AI Agent Communications on the Internet"
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
    organization: Alibaba
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

  RFC8693:

  RFC9420:

  I-D.ietf-wimse-aims:

  I-D.ietf-wimse-workload-creds:

  I-D.ietf-oauth-identity-chaining:

  I-D.ietf-oauth-transaction-tokens:


--- abstract

This document defines the terminology and architecture for AI agent communications protocols over the Internet. It provides the foundational architectural elements for the Agent Communications Protocol (ACP) - also known as agentproto - a new protocol to be developed in the IETF. ACP enables communication between user agent clients and AI agents, between AI Agents and other AI Agents, or between AI Agents and tools. All three of these interaction patterns - and the many use cases therein - require a session-type protocol which handles lifecycle management, including creation, modification, deletion and so on - along with the exchange of input and output. These sessions can be extremely long lived - running for hours or even days - and span multiple IP hosts used by the both the principal and by the agent. They can also be highly intermittent with asynchronous responses. All of these mean a new type of session protocol is needed.



--- middle

# Introduction {#intro}

The widespread usage of AI Agents on the Internet has introduced new forms of networked communications. User agent clients connect to AI agents running elswehere, AI agents communicate with other AI agents, and AI agents communicate with tools. All three of these interaction patterns - and the many use cases therein - require a session-type protocol which handles lifecycle management, including creation, modification, deletion and so on - along with the exchange of input and output. These sessions can be extremely long lived - running for hours or even days - and span multiple IP hosts used by the both the principal and by the agent. They can also be highly intermittent with asynchronous responses. All of these mean a new type of session protocol is needed.


These sessions differ from the type of sessions created by the Session Initiation Protocol (SIP) [RFC3261], which are shorter lived and work between a single IP host for each participant.

This document introduces concepts, terminology, reference architecture and protocol functional blocks for ACP, which manages agentic dialogs. An agentic dialog is defined as a session between two parties - a principal whose role is to provide input to, and receive output from, an agent - and the agent, which is running a process which has properties which are agentic in nature. The agentic properties that are relevant here are (1) conversational contuity, and (2) driven by natural language in each direction. These concepts are discussed in more detail in this document.

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


AI Agent:
: A software system that uses AI models to complete a task on behalf of a
  principal - which can be a user or another AI agent.

Principal:
: An entity whose role is to provide input to, and receive output from, an agent.
The principal can be a human user (as in the user agent to AI agent case noted above), but the principal can also be an AI agent - as in the A2A case.

User:
: A human being who plays the role of the principal. In this document, the word "user" always implies a human being.


Agent:
: An entity whose role is to receive input from a principal, use it to augment its context, and based on an intelligent processing of that context, generate output to be returned to the principal. Typically, an agent is an AI Agent, but ACP doesn't preclude a human from playing that role. For this reason, the ACP protocol uses the term "agent" instead of "AI Agent".

Task:
: A unit of work submitted by a principal to an agent, typically expressed through the input provided to the agent which describes the task to be done. In some agentic dialogs, the task may evolve throughout its lifetime. In other agent dialogs, such as that used for communications between an AI agent and a tool, there may be just one. In other agentic dialogs, there is no real notion of task - its just a conversation.


Context:
: Context is central to the concept of AI Agents. There are many definitions on the Internet. Anthropic defines it as "the set of tokens included when sampling from a large-language model (LLM)." {{anthropic-context-def}}. The goal of ACP is to exchange the input from the principal to allow the agent to manage context, and the output from inference of that context.

LLM Context:
: Same as context. We used the adjective "LLM" in front of context to help differentiate LLM context from dialog context.

Agentic Dialog:
: A session between two parties - a principal whose role is to provide input to, and receive output from, an agent - and the agent, which is running a process which has properties which are agentic in nature and result in the maintenance of LLM context. The agentic properties that are relevant here are (1) conversational contuity, and (2) driven by natural language in each direction. An agentic dialog is associated 1-1 with the LLM context it builds. ACP doesn't define how the agent maintains the LLM context; it merely allows for the establishment of an agentic dialog and exchange of data which allows the agent to do so.

Input:
: Information provided by the principal, which is used to drive the operation of the agent. In simplest terms, this is the stuff that the principal types or speaks to the AI agent, e.g., "I want to book a trip to Maui" or "That sounds like a good idea but what is the weather like that time of year?".

Output:
: The information produced by the agent which are meant to be conveyed to the principal. There are three distinct forms of output - response, information, and progress.

Response Output:
: This is the portion of the output which is meant to be shown to the user, and is what the user needs to know to decide their next input. For example, "I've booked that trip for you. Do you need a car too?". The output can of course constitute any modality.

Information Output:
: This is the portion of the output which helps the principal understand what the agent has done, but is not necessarily required for them to compose their input. This includes tool call summaries, reasoning, sub-agent invocations, and so on. Information output is content that the agent used to build its local context but is not necessary for the principal to know.

Progress:
: This is the portion of the output that provides updates on work to the principal. This includes token counts, time elapsed, usage progress, and so on. Progress output is informational in nature, like information output, but it has the important distinction that it is not data the agent used to build its LLM context. As a result, it is almost aways generated by the agent harness and not the LLM itself.


Dialog Context:
: Dialog context refers to the state maintained by, and exchanged via, the ACP protocol. Each agentic dialog is associated 1-1 with an instance of dialog context. The dialog context has two primary elements - meta-data and the log. Meta-data is the set of information that is required for the operation of the protocol itself, and includes protocol identifiers (such as the Agentic Dialog Identifier (ADID)), participants along with their identifiers and roles, supported modalities, start time, and so on. The log is the append-only collection of the inputs provided by the principal, the outputs provided by the agent (inclusive of response, information and progress output). The log includes data across all modalities, including real-time voice and video, along with text, images and structured data.

Log:
: The portion of the dialog context that is the append-only sequence of input from the principal and output (response, information and progress) from the agent.

Agentic Dialog Identifier (ADID):
: An identifier which identifies an agentic dialog and the context associated with it. This identifier survives network interruption or change.


Modality:
: A category of data exchanged in an agentic dialog, such as text, audio, image, video or structured data.

Progress Signaling:
: The act of providing progress output from the agent to the principal.


Administrative Domain:
: A set of entities, such as agents, tools, and principals, that are
subject to a common authorization policy and a common authority.

Agent Identity:
: Identity information associated with an AI agent, distinct from the
  identity of the user on whose behalf it acts, and used for
  authentication, authorization, and accountability.


Delegation:
: The act of an AI agent requesting another AI agent to execute a task on
  its behalf.

Tool:
: A service invoked by an AI agent to retrieve data or perform operations.
  A tool is not necessarily an AI agent. However, in some cases a tool can be session-oriented in nature.

Trust Boundary:
: A boundary between administrative domains.

Message:
: A discrete unit of communication exchanged between agentic dialog participants.





# Interaction Pattern

ACP targets three distinct interaction patterns:

~~~
+-------------+
| User /      |
| Principal   |
+------+------+
 (ACP) |
       v
+-------------+        +--------+
|  Agent (A)  |------->|  Tool  |
+------+------+ (ACP.  +--------+
  (ACP)|         other)
       V
+-------------+        +--------+
|  Agent (A)  |------->|  Tool  |
+------+------+ (ACP,  +--------+
                 other)
~~~

## User Agent to AI Agent

In this interaction pattern, a human user (which we'll just refer to as a user) is sitting in front of a desktop application, mobile application, or web application providing a user interface for engaging with an AI agent. The AI agent runs elsewhere - on a server - different from the user agent. In this pattern, the user agent is really a "thin client" of sorts -  passing information from the user to the AI Agent, and passing information from the AI Agent to the user. This communications protocol would pass user messages, AI agent responses, meta-data like token usage, sub-agent spawns and completions, and so on. If the AI agent supports voice input and spoken output, the communications protocol would pass real-time voice information back and forth as well.  Basically - all of the information needed to render the UI.

If the user agent is a thick desktop application, the communications protocol is from the user's desktop to the AI agent on the server. In a mobile app, it is from their mobile app to the AI agent on the server. If the user agent is a web application, it is most likely from the web server to the the server on which the AI agent is running.

This interaction pattern is different from how desktop AI agents like Claude work today. Those have the AI agent running co-resident with the user interface. There is communications with a server elsewhere, but it takes the form of model invocation and tool invocation.

When the vendor of the AI agent is the same as the vendor of the user agent, there is not necessarily a need for standardized protocols between them. However, standardized protocols can be helpful in cases when the user agent is an embedded device made by a different vendor than the one making AI agents. For example, consider a case where a user has a smart speaker (similar to Amazon Alexa). However, the smart speaker is made by one vendor (say, Bose, who makes such devices), and the desire is to allow the smart speaker to connect with AI agents made by many vendors - Anthropic, OpenAI, Google, or even B2B AI agents. In this kind of case, when the cardinality of AI agent vendors is high, there is a strong need for a standardized protocol here.

## AI Agent to a Session Oriented Tool

In this interaction pattern, an AI agent is running somewhere - a user's desktop, or perhaps on a server. The AI agent wishes to invoke a tool. However, this tool is not just a simple one that performs a request/response transaction. Examples of a simple input and output tool would be a traditional REST API, or a tool that is exposed through the A2T (Agent to Tool) protocol [I-D.rosenberg-aiproto-a2t]. These are transactional, stateless API calls. There is no session - nothing long lived.

However, there are cases where the tool itself is session oriented. The primary example is the Model Context Protocol (MCP) which is session oriented. The tool might also be agentic in nature - taking structured data as input, providing it ultimately as output, but requiring interaction with the user back and forth in order to fulfill the request.

## AI Agent to AI Agent

In this interaction pattern, two AI agents are running in different places. Perhaps they are made by different vendors but running in the same domain. An example might be an enterprise running AI agents from a CRM vendor, which communicate with AI agents made by a provisioning system vendor. They may be in different domains. Or they may be distinct agents made by the same vendor, split up into separate agents on different servers for reasons of scale or isolation.

This is the core interaction pattern for A2A {A2A}.

# Core Concept: Agentic Dialog

When we look at all of the interaction patterns, there is a common theme which runs through all of them - the notion of a long-lived session between two entities, with a growing context window as the interaction progresses, with a defined start and eventual finish. That's a mouthful, so we propose a specific name for this - an agentic dialog.

An agentic dialog is defined as a session between two parties - a principal whose role is to provide input to, and receive output from, an agent - and the agent, which is running a process which has properties which are agentic in nature. The agentic properties that are relevant here are (1) conversational contuity, and (2) driven by natural language in each direction.

The principal can be a human user (as in the user agent to AI agent case noted above), but the principal can also be an AI agent - as in the A2A case. The agent is typically an AI Agent, but it could be another human,  with that human being receiving the input and providing output. In this way, the term "agent" defined here refers strictly to the role in the protocol, which can be fulfilled by a human agent, an AI agent, or even traditional software. Indeed, in the A2T case, the agent might be a piece of software which supports only a small amount of conversational capability.

The agentic dialog is facilitated by a protocol, which runs between a client (acting on behalf of the principal) and a server (acting on behalf of the agent). These distinctions are important, because the protocol needs to communicate information which is sometimes relevant only to the client (such as an identifier for the agentic dialog), and sometimes information relevant only to the principal - such as the text that is communicated. This layering is similar to how other Internet protocols operate, e.g, email - which has distinct concepts of envelope (meant for the mail agent) and the body (relevant for the recipient).


## Definitional Properties

An agentic dialog is distinct from other types of sessions (like the SIP dialog) because of two key properties.

### Conversational Continuity

Conversational continuity means that, as far as the principal is concerned, the output generated by the agent appears as if any output has propertly taken into account all of the prior inputs and outputs shared between them. Typically, this is accomplished through LLM context that is built on the agent. The agent initializes its LLM context at the start of the dialog, and as the dialog progresses, the LLM context evolves as the principal provides input, and the LLM performs inference. The LLM context may be compacted if it gets too long, but the compaction serves the primary purpose of preserving continuity - retaining what is important so that, to the principal, it is almost as if there was no compaction or loss of information.

The LLM context is used by the agent to determine how to satisfy the needs of the principal as specified via input. Those needs may evolve over time.

Conversational continuity means that the LLM context is associated 1-1 with each agentic dialog that feeds it.

### Driven by Natural Language

In an agentic dialog, the input from the principal to the agent, and output from the agent back to the principal, contain natural language, in written, audio, video, or image form, or even structured data. We use the term "natural language" pretty loosely here. If a user is talking to an AI agent to modify an image, the user might send the image they want modified. This image is a form of "natural language" in our definition. The notion here is that it is processed according to a human or human-like understanding of why the image is being sent. The same is true for structured data - for example a piece of software. The agent may generate a Java file, and send it back to the principal. The information being shared is structured - it is source code - but it is understood as code for a specific purpose based on the context of the conversation.

This means that a protocol for managing agentic dialogs needs to be able to convey these various forms of content in both directions.

## Additional Properties

Agentic dialogs have other properties which are impactful on the protocol.

### It has a Start

Perhaps obvious but worth stating. The agentic dialog has a start. This start is the point at which the dialog context and LLM context are both initialized. The LLM context may not be initialized as empty; the agent might use the identity of the principal to add personalization. Or, it may be a coding agent which reads an agents.md file to initialize what it is doing. It might read system instructions. For the agent to behave properly, it needs to be told that a particular agentic dialog is starting. The log portion of the dialog context is, however, initialized as empty.

### It may be Long Lived

The agentic dialog may last a really long time. By "last" we mean that the dialog context which is its defining property, may continue to exist and be accessible by the principal for long periods of time.

One example is an agentic dialog between a principal and agent that is used for a therapy session. A human user is connected to a client on their desktop; the server is running an AI agent which acts as a virtual therapist. The agentic dialog continues for months.

### Input can be Intermittent

The input from the principal can be intermittent. As an example, consider a user that is using a coding agent, and they have a period of activity where they are typing at it and getting responses, and then they head to bed at some point. The user comes back in the morning to continue the agentic dialog.

In this case, if the user's the client was on their desktop, and the agent was an AI agent on a server in a cloud somewhere, there is no user input (and potentially no AI Agent output) throughout the entire evening, and no data transmitted from the users' home network towards the Internet through the evening.

This means the protocol needs to account for alternating periods of activity and quiesence. And, because the agentic dialog can be long-lived, the actual IP host which is acting as the client or the server might change over the lifecycle of the agentic dialog.

### Output can be Delayed

The output from the agent could arrive a long time after the input has been sent. Anyone who has used a coding agent is familiar with this - it can sometimes take many minutes before the first response is returned.

### It Might or Might Not End

Agentic dialogs can have a formal end, in which case it is not possible for the principal to contribute further to it. If an agentic dialog ends but is actually resumable, it hasnt really ended. Since the agent takes ownership for continued maintenance of both the dialog and LLM context, it needs the ability to declare that it is no longer supporting this context, and ending the dialog. Similarly, a principal should be able to say "I'm done" giving permission for the agent to immediately delete the context associated with the agentic dialog.

### Output can be Asynchronous

In early chatbots, each user input created a single user output, resulting in a simple turn-by-turn model of a conversation.

Modern AI Agents can invoke tools that include background activities. As a result, a single principal input might generate any number of agent outputs, and some of those outputs can come seconds, minutes, or even hours after the initial input.

This complicates the protocol significantly. If the agent needs to generate a response, and that response comes long after the request which triggered it, the server cannot assume the client is still running on the same IP host. Historically, IP protocols have solved for these use cases with webhooks, requiring the client to run its own HTTP server.

### Agents are Named

When a principal establishes an agentic dialog with an agent, it is attempting to do so to a particular named instance of the agent. This named instance is, at the very least, associated with the vendor of the agent. "I want to connect to Claude" or "I want to talk to the Walmart AI Agent". However, there can be additional levels of naming too. One can imagine a vendor having different agents with different personas - a contact center AI agent which handles sales vs. support, or a virtual knowledge worker from Salesforce that handles accounting vs. one that handles legal issues. Generally the differing named agents vary by LLM model, but also through the instructions, prompts, guardrails and other aspects of the harness which define its behavior.

When the principal connects to the agent through its client to initiate an agentic dialog, the client needs to provide to the name to the server, so the server can run the right named instance of the agent.


### Client vs Principal and Server vs. Agent

There is a distinction between the principal and the client. The principal exists outside of the protocol; the role of the protocol is to collect input from the principal, and to provide output back to the principal. In traditional protocols we'd use the word "user" to represent the human driving the protocol, and not "principal". However, that terminology doesn't work here because the principal can be an AI agent which is not a human being. The word "principal" is agnostic to whether it is a human being or an AI Agent. The software element - and it is a piece of traditional software implementing ACP - is called the client.

In the same way, the "agent" exists outside of the protocol as well. It is an AI agent, though it could be a human being. The server is the piece of software - also traditional software - which implements ACP and is responsible for funneling information to and from the AI Agent.

This distinction is important because it helps us understand how to properly layer the system.


### ACP Manages Dialog Context - not LLM Context

The ACP protocol is not directly transmitting LLM context. The naive way to think about LLM context is that it is just a growing log that concatenates each piece of input sent from principal to agent, and each output from agent to principal. If that were true, there would be no distinction between the dialog context and LLM context. But, this is not the case.

Firstly, the LLM context is usually initialized with system instructions, which are never seen by the principal and never need to be seen by them. These are never transmitted on the wire by ACP, and do not form part of the dialog context. Similarly, if an agent loads a skill, the skill details are added to the context. However, those details are not transmitted over the wire by ACP, nor are they added to the dialog context. The dialog context would just indicate that the skill was loaded. Indeed, skills might be considered proprietary and it is an explicit goal of the vendor of the agent to not send them to anyone who uses the agent!

What ACP does exchange, is input and output. Input is information the principal wishes to send to the agent, and output is information the agent wishes to send to the principal. The input and output are appended to form the log, the main part of the dialog context.

The distinction between LLM context and dialog context also aligns with the distinction between server and agent. The LLM context is a core property of the agent, NOT of the server, whereas the dialog context is a property of the server, and not its associated agent. In ACP,  we are conveying information to and from the agent on the server, but the LLM context lives with the agent. The protocol will concern itself with technical conveyance issues necessary to make sure both client and server have an aligned view of the dialog context. But, how that dialog context is related to the LLM context is a matter of local implementation.


### Output Isnt Just the Response

The output has three distinct elements:

* Responses: This is the output of the agent which needs to be rendered to the principal for them to make a decision on what to do next. Examples include text responses, like "I've booked that trip for you, do you need a car?", images - such as a photo of a hotel or a rendered image from an image-generation agent, real-time voice - used by live agents that interact with principals over real-time voice, common in contact center applications for example, real-time video - an emerging output modality used by agents that are represented by talking-heads, able to move their mouths, make facial expressions and so on. Of course, it also includes structured data - for example a JSON object or HTML page.

* Information: This is the output of the agent which is useful for the principal to know, and is used by the agent as part of its LLM context - but is not necesarily essential for the principal to observe in order to determine their next input. Examples of this include tool calls, sub-agent invocations, and reasoning.

* Progress: Whereas information output is generated by the LLM and fed back into the agents internal context, progress is data generated by the harness and is useful to understand status. This includes token counts, turn durations, and costs.

It is important to note that a piece of output in the dialog context is not necessarily exactly the same as what the agent adds to LLM context. For example, the agent may apply obfuscation to sensitive tool call results. Or, it may simplify the tool call output to facilitate comprehension by the principal, by omitting unimportant fields (a trace ID for example).



# Network Architecture Diagram {#network-arch}

From these properties, we can draw a network architecture diagram which describes the entities between which ACP works.

~~~
{::include arch.md}
~~~

The principal accesses the agent through one or more distinct clients over the lifetime of a specific agentic dialog. The notion of "distinct" here is that they can reside on different IP hosts. As noted in the examples above, in an A2A use case, the client might be on a virtual compute node A with a specific IP today, and then tomorrow the principal (an AI agent in this case) resumes the dialog from a different virtual compute node B with a different IP.

In the same way, a particular named agent - and any instance of an agentic dialog created with that agent - can be accessed through a multiplicity of distinct servers - each of which is on a distinct host.

ACP, as an on-the-wire protocol, runs over an IP network between the clients and servers.

The ACP protocol is fundamentally two-party - it is between a principal and an agent. More complex topologies are supported by cascading together multiple independent agentic dialogs. For example, a user might connect to a an agent for banking, and that agent connects to a second agent for a mortgage quote. In this case, there are two distinct agentic dialogs, two unrelated instances of ACP. They are connected only through delegated identities and the chain of dialog context identifiers that are passed from one to the other.

The clients and servers exist on IP hosts. The agent in the picture is conceptual in nature, and is not an IP host. It might be a human, but most often is an AI agent composed of a harness and LLM, building and using context fed by the input received from the server.


# Functional Decomposition

We can think about the ACP protocol as have three core building blocks.

~~~
+----------------------------------------------------------------+
|                Agent Communication Protocol                    |
|                                                                |
| +------------------+ +------------------+ +------------------+ |
| | Agentic Dialog   | | Authorization    | | Identity         | |
| | Management       | | and Delegation   | | Management       | |
| |                  | |                  | |                  | |
| | - Identifier     | | - Delegation     | | - Roles          | |
| | - Lifecycle      | | - Authorization  | | - Participant    | |
| | - Exchange       | | - Decision point | |   identity       | |
| | - Recovery       | |                  | | - Credential     | |
| |                  | |                  | |   reference      | |
| +--------+---------+ +---------+--------+ +---------+--------+ |
|                         Dialog Context                         |
+----------------------------------------------------------------+
                                |
                                v
+----------------------------------------------------------------+
|                          Transport                             |
|  - Multiplexed Streams                                         |
|  - Per-stream Semantics                                        |
|  - Connection continuity (Connection ID, TLS resumption)       |
|  - Protocols: HTTP, QUIC, WebTransport, WebRTC, MoQT           |
+----------------------------------------------------------------+
                                 |
                                 v
+----------------------------------------------------------------+
|                       Security                                 |
|  - Channel Protection                                          |
|  - Mutual Authentication                                       |
|  - Protocol: TLS 1.3, WIT/WPT                                  |
+----------------------------------------------------------------+
~~~

## Agentic Dialog Management {#dialog}

Agentic dialog management is the central role of the protocol.

Its role is to identify an agentic dialog, to provide lifecycle management for it, to exchange input and output used to grow dialog context, and to allow the agentic dialog to continue through the failure of clients, servers and network throughout its lifecycle. It can work within a single trust domain, or across trust domains.

It is the building block that allows a dialog context to remain recognizable over multiple hops, over time,
and across transport connections.

### Agentic Dialog Identifier and Correlation {#dialog-id}

Every agentic dialog is required to have an agentic dialog identifier (ADID). The identifier is required to:

- correlate all messages that belong to the agentic dialog, in each direction
- survive the loss or replacement of a transport connection, so that the
  agentic dialog is not tied to a TCP connection, QUIC connection, or TLS
  association;
- be usable by an entity to link to distinct agentic dialogs, where one is triggered based on the other - for example, in sub-agent delegation.


The dialog identifier should be a UUID, unique in space and time, and unguessable. Knowledge of an ADID provides no information whatsoever about the agentic dialog. Knowledge of he ADID is also insufficient for a client to connect to the server and access the agentic dialog in any way - the client must also authenticate itself, and be authorized for such access.

### Lifecyle

ACP needs to perform lifecycle management, covering:

* Initiation: beginning a new agentic dialog

* Termination: ending an agentic dialog such that it cannot be resumed

* Suspension: Pausing input or output, so that there no longer remains any active transport level connection for exchaning that input or output. Suspension could be time based, or explicitly triggered.

* Resumption: Resuming input or output, requiring (if needed) creation or modification of transport connections to convey the input or output. Resumption also requires protocol agreement on whether the client needs to replay prior input, or whether the server had already processed it.

* Cancellation: cancellation of prior input


Agent interactions are often long-lived, interruption-prone, and delegated across multiple hops. Each agentic dialog is required to have an ADID that survives network interruption or change. If a transport connection is interrupted, an agent uses the ADID to re-associate the existing agentic dialog with the newly formed connection, instead of initiating a new agentic dialog. Of course, proper authentication is also required - knowledge of the ADID is insufficient for the newly formed connection to be associated to the agentic dialog.

Similarly, an agent performing delegation uses the ADID to correlate the agentic dialog it terminates, with the one it initiates.

Agentic Dialogs are defined above any transport protocol connection (TCP connection, QUIC connection) or TLS association. QUIC Connection ID and TLS resumption provide connection-level continuity; they preserve transport and cryptographic state, not agentic dialog state.

### Exchange

ACP allows the principal, through the ACP client, to provide input to the agent, delivered via a server.

The input constitutes many modalities, including voice, video, text (i.e., chat), images, and structured data. These modalities encompass both real-time (video and voice), and non-real-time (text, images, and structured data). In the case of A2A for example, the input might be a piece of text which describes the task to perform - a request to recognize an image, along with the image to recognize.

ACP allows the principal to contribute input at any time, and the agent to provide output at any time.

Output, like input, can constant all modalities. If a principal is providing input via real-time voice, the output might be real-time voice too. However, it can also include progress signaling, such as tool calls or skill loading, to be rendered on a UI.

Not all agents and principals can handle all modalities. For this reason, ACP requires traditional negotiation capabilities, allowing them to be exchanged at the beginning of the agentic dialog.

### Recovery

As noted in {#network-arch}, ACP needs to handle the fact that the agentic dialog can work even when the client and the server move between IP hosts through the lifetime of the dialog.

Similarly, ACP needs to provide recovery in case any of these hosts should fail during the agentic dialog. This can happen during the exchange of real-time media (e.g., a server fails while receiving the user's voice input), or during the transmission of non-real-time input (the client sends a text message, and the server failed while processing it, but before completion of that processing).

ACP also needs to provide rehydration. If a new client instance starts up in the middle of the dialog, it may need to hydrate any input or output which had been sent or received but not yet made it to any local databases used for storage.


### Structured Error and Progress Signaling {#error-signaling}

ACP is required to carry more than the final output of the agent in each turn. It is required to also convey incremental output, progress updates, checkpoint states, cancellation states, and structured errors usable by an orchestrating peer. Error signaling is required to distinguish transport failure from application failure, authorization failure, policy rejection, timeout, validation failure, and interoperability problems. Progress and liveness notifications are required to be representable independently of final completion so that a peer can tell whether an agent is active, blocked on external input, or waiting in a recoverable state. It is required to ensure that terminal error or cancellation signals are not silently lost if an ordinary data stream fails.

### Cancellation and Interruption {#cancellation}

Agent execution may be interrupted by user/principal action, policy enforcement, higher-priority work, loss of authorization, or dependent-subtask failure. The protocol is required to define explicit application-layer cancellation and interruption signaling, rather than rely on connection teardown at transport protocol layer. Cancellation is required to identify the affected scope, including a single invocation, delegated subtree, or entire agentic dialog. Receiving agents are required to treat cancellation as idempotent and report whether execution stopped, had already completed, or could not be fully rolled back due to an irreversible action. In multi-hop deployments, interruption semantics are required to propagate predictably so that dependent subtasks do not continue after withdrawal of the parent task.


## Authorization and Delegation

Authorization and Delegation determines whether a participant is permitted to
act within an agentic dialog and on whose behalf. This sub-block does not define an
authorization mechanism. It identifies the functions and the information that
the Agent Communication Protocol needs to associate with a dialog so that
existing authorization mechanisms can be applied at the points where they
matter.

ACP relies on objects defined by other standards to convey authorization and identities. In this way, it is extensible and modular to evolving work in this area in the OAuth working group.

### Delegation {#delegation}

Delegation is an act between participants, not a property of a
transport connection. The architecture distinguishes three roles:

- the principal, such as a user or an organization, whose authority is
  originally exercised;
- the delegating agent, which requests another agent to execute a task;
- the delegated agent, which executes the task.

In multi-hop delegation, an agent can be the delegated agent for one hop and
the delegating agent for the next. A delegating agent can re-delegate authority
derived from the original principal. A delegation can be revoked or terminated
independently of the agentic dialog in which it was used.

In delegated use cases, there are two distinct agentic dialogs, each with a distinct dialog context and a distinct ADID. The first is between the principal and the delegating agent, and the second between the delegating agent and the delegated agent. We refer to the second of these as the delegated dialog context. The delegated dialog context has two additional pieces of meta-data - the ADID of the first dialog context, and the identity of the principal in the first context. The delegating agent is responsible for communicating these over ACP. Conceptually, it looks like this:

~~~

Principal --------------> Delegating Agent --------------> Delegated Agent
              ADID:167                         ADID:293
                                               ADID-Delegation-Chain: 167
                                               Delegated-From: Principal

~~~

ADID 167 refers to the agentic dialog between the principal and delegating agent. There is a separate agentic dialog, with ADID 293, between the delegating agent and the delegated agent. However, when the second agentic dialog is formed, the delegating agent provides the chain of prior ADIDs. In this case, there is just one - 167 - which refers to the agentic dialog which triggered it. THe delegating agent also conveys the identity of the principal on whose behalf they are operating, in this case it is the original principal.

The constraints in {{intent-execution}} and {{delegation-chain}} apply: the
authority passed at each hop is limited to the delegated task and is not
broadened along the chain.

## Identity Management

Identity Management establishes who the participants of a dialog are. It
builds on the agent identity requirements in {{agent-identity}} and on the
Agent Identity Management System model in {{I-D.ietf-wimse-aims}}. The Agent
Communication Protocol does not define identity credentials or authentication
protocols. It defines how participant identities are associated with the
agentic dialog and how that association is preserved.

### Identity Roles {#identity-roles}

The architecture keeps the following identities distinct:

- the principal identity, which is the identity of the user or organization on
  whose behalf an agent acts;
- the agent identity, which is the identity of the agent itself and is
  distinct from that of the principal;


Keeping these identities distinct is required so that a participant can
distinguish the principal, the delegating agent, and the delegated agent, and so that accountability can be
assigned to the right entity.

In multi-party use cases, such as when there is a "conference call" between two users and an AI agent, there will be two distinct agentic dialogs, each with a distinct dialog context. However, the LLM context is shared, so that the agent is "seeing" the inputs from both principals, and both principals see the output from the agent. Each principal also needs to see the other's input. This introduces the need to associate the identity of the principal with each input, and the need for ACP to communicate input from the server, back to the client, that wasn't sent by the principal on that dialog.

# Security  {#security}

Security for agent communication spans four interdependent concerns: verifiable agent identity, channel protection, binding of authorized intent to agent execution, and delegation chain integrity. The use cases and protocol requirements that motivate this architecture are discussed in {{I-D.agentic-ai-usecases-requirements}}. Some of the mechanisms for satisfying these requirements using existing IETF standards are described in {{I-D.ietf-wimse-aims}}.

## Agent Identity {#agent-identity}

Agent identity encompasses two concerns: a stable unique identifier and cryptographic credentials bound to that identifier.

Every agent is required to be assigned a unique, stable identifier that remains consistent across network reconnections and agentic dialog resumptions. Credentials are required to be bound to the agent's identity. The credential lifecycle, including provisioning, rotation, and revocation, is required to be supported without manual intervention. {{I-D.ietf-wimse-aims}} defines the term Agent Identity Management System (AIMS) as a conceptual model describing the set of functions required to establish, maintain, and evaluate the identity and permissions of an agent workload.

These identity requirements are satisfied by using WIMSE Workload Credentials {{I-D.ietf-wimse-workload-creds}}: a Workload Identity Token (WIT) at the application layer and a Workload Identity Certificate (WIC) at the transport layer. Both bind a public key to the agent's workload identity.

## Channel Protection {#channel-protection}

All agent communication is required to be encrypted and mutually authenticated. This architecture does not define a fallback to cleartext or unauthenticated operation.
Authentication may operate at the transport layer, the application layer, or both. Transport-layer authentication works well when TLS connections are not terminated by intermediary IP hosts. In architectures involving proxies, application-layer authentication is required to maintain identity continuity across TLS termination points. The channel protection mechanism is required to be negotiated during transport connection establishment, with both endpoints authenticating before any message is exchanged.

## Intent-Execution Separation {#intent-execution}

AI agents dynamically generate execution plans and issue sub-requests based on their own reasoning, which may diverge from the scope of the original user authorization. This architecture requires that authorization granted to an agent be limited to the scope of the delegated task, so that an agent cannot use it to authorize actions outside that scope. This depends on delegation mechanics in which an agent acts on behalf of the original user or principal and the authority passed at each hop is narrowed to the delegated task rather than broadened. Standardizing these delegation mechanics for AI agents is an active area of work in the OAuth WG. This document will reference the relevant outcomes as this work matures.

## Delegation Chain Integrity {#delegation-chain}

When one agent delegates a sub-task to another, the receiving agent is authorized only to the extent explicitly permitted by the delegating agent. In multi-hop scenarios, the delegation chain may span several agents before reaching the resource or tool ultimately invoked. At each hop, the receiving party is required to verify that the authority presented was derived from the human or system that originally authorized the task, has not been expanded beyond what was originally granted, and has not been forged or modified in transit. A delegating agent is not permitted to grant authority beyond what the original authorization permits for the delegated task. This draft will be updated with more details as the work progresses in the OAuth WG.

## Audit and Non-Repudiation {#audit}

Agents act on behalf of users/principals or other agents across multiple hops and
administrative domains. Audit requires that each action be traceable to the
principal that authorized it and the agent that executed it. For Non-repudiation,
this traceability is required to be cryptographically verifiable, so that
no party can later deny its role. The mechanism for recording and
verifying it is outside the scope of this document.


# Transport Aspects {#transport}

Transport for agent-to-agent communication spans several interdependent concerns: agentic dialog continuity across long-running tasks, heterogeneous delivery semantics, explicit agentic dialog and stream correlation, efficient transfer of large data, signaling for priority and cancellation, structured error propagation, and negotiation of modalities and group communication topologies. The use cases and protocol requirements that motivate these transport properties are discussed in {{I-D.agentic-ai-usecases-requirements}}. The transport substrate is not required merely to deliver bytes between endpoints; it is required to preserve the correctness, efficiency, and recoverability of delegated agent execution across administrative domains and under changing network conditions.

## Delivery Semantics {#delivery-semantics}

ACP requires heterogeneous transport semantics rather than a single uniform model.

Agent communication patterns fall into three broad delivery semantic classes, each with distinct transport properties:

Reliable ordered delivery:
: Used for control messages, progess signaling, authorization checkpoints, non-real-time user input, and non-realtime user output. The transport substrate is required to guarantee in-order, lossless delivery for these message classes, and is required to isolate them from other traffic classes to avoid head-of-line blocking.

Low-latency, loss-tolerant delivery:
: Used for real-time audio, video, and high-frequency sensor or telemetry streams. The transport substrate is required to minimize latency for these streams and is not required to guarantee delivery or ordering.

High-throughput reliable delivery:
: Used for bulk transfer of large data between dialog participants. The transport substrate is required to support high-throughput reliable transfer with flow-control isolation from other traffic classes [OPEN ISSUE].

## Message Exchange Patterns {#message-exchange}

Agentic dialogs are not limited to simple request-response exchange. An agent may delegate work asynchronously, receive an acknowledgement before completion, stream partial results, emit progress updates, request additional authorization, and later return a final result or cancellation status. The transport is required to take these patterns explicitly into account, including correlation of messages to the relevant agentic dialog. Multiplexing is required so that multiple delegated tasks or tool invocations can proceed concurrently without unrelated head-of-line blocking. The transport is also required to permit out-of-order completion of concurrent subtasks while preserving per-task ordering for incremental output and terminal states.

## Priority and Scheduling {#priority}

Delegated agent work varies in urgency and consequence. Some exchanges are on the critical path of a user-visible task, while others are background enrichment, speculative reasoning, or deferred synchronization. The protocol is required to signal task priority and delivery urgency so that transport and scheduling layers can differentiate service. Such signaling is required to be advisory and policy-controlled; a sender is not permitted to raise priority beyond the authority or policy of the delegated task. Priority signaling is also required for coordination messages such as authorization responses, cancellation acknowledgements, and error notifications, because delaying them can break an otherwise correct workflow.



## Multiple Communication topologies {#topologies}

Some agent interactions may use one-to-many or many-to-many communication among a group of agents whose membership may change during an exchange. The transport is required to support these delivery patterns and to handle agents joining or leaving the group while the exchange is active. An agent's authorization to participate is required to be verified when it joins, and the keying material is required to be updated whenever an agent joins or leaves, so that an agent can decrypt group traffic only while it is a member. MLS {{RFC9420}} provides group key establishment with these properties.


# Applicability of Existing IETF Work {#existingworks}
## Reuse As-Is

* TLS 1.3 {{!RFC9846}} provides mutual authentication and channel protection. Agents
  are required to negotiate TLS 1.3 or higher.
* QUIC {{!RFC9000}} and QUIC-TLS {{!RFC9001}} provide multiplexed streams and 0-RTT
  session resumption.
* OAuth 2.0 Token Exchange {{RFC8693}} provides the base token exchange mechanism.
* WIMSE Workload Credentials {{I-D.ietf-wimse-workload-creds}} are reused for agent
  identity; see {{agent-identity}}.

## Profile or Extend

The following existing protocols may require profiling or extension; this list is expected to evolve as new IETF and OAuth WG specifications emerge:

- The OAuth WG is actively discussing how existing and new mechanisms apply to AI agent authorization. {{I-D.ietf-oauth-identity-chaining}} addresses cross-domain authorization, and {{I-D.ietf-oauth-transaction-tokens}} addresses intra-domain token exchange between workloads. This document will track this work and profile the relevant outcomes once the OAuth WG reaches consensus.

- MoQT (Media over QUIC Transport): acts as a unified transport substrate for distributed agent state synchronization and real-time multimodal communications. Work needs to happen to provide a common mapping of request-response and streaming patterns onto the pub/sub model of MoQT in order to enable interoperability across diverse agent ecosystems.


# Security Considerations {#secconsiderations}

AI agents enlarge the attack surface: they act autonomously, delegate across administrative domains, and operate over long-lived dialogs that rely on long-standing delegated authority. This section highlights threats specific to the building blocks in this document; detailed mitigations are expected in the specifications of those building blocks.

Delegated authority is the primary risk. A rogue or compromised agent may attempt to use delegated authority beyond the task it was granted, or to broaden it as it delegates onward. As required in {{intent-execution}} and {{delegation-chain}}, authority granted to an agent is scoped to the delegated task and cannot be broadened along the chain, and the delegation chain is verifiable at each hop so an agent on the chain cannot forge or escalate it.

Valid credentials do not show that an agent runs on an uncompromised platform. Attestation allows a peer to assess the integrity of the agent's execution environment.

QUIC and TLS 1.3 0-RTT data can be replayed by an attacker (Section 8 of {{RFC9846}}). Messages that trigger non-idempotent actions are not permitted in 0-RTT data.

# Privacy Considerations {#privacy}

The ADID is used to correlate together distinct agentic dialogs, when one of them triggers the next. This means entities which know the ADID could attempt to access dialog contexts they were not privvy to. The Agent Communication Protocol is expected to analyze
these risks and specify mitigations, such as limiting the scope and
lifetime of dialog context identifiers.

Agent identity information is sensitive, particularly in multi-domain
deployments. Persistent agent identifiers used across dialogs and domains
enable tracking and correlation of agent activity. Pseudonymous or
temporary identifiers reduce linkability while preserving the ability to
audit and enforce accountability where required.

Because agents exchange rich, potentially sensitive multimodal task context, both message content and metadata require protection. Even with content encrypted, analysis of message sizes, timing, and stream patterns can leak the nature of an agent's tasks; mitigations such as padding may be warranted where that exposure matters.


# IANA Considerations {#iana}

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
