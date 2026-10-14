---
title: "AgentProto Requirements, Use Cases, Concepts and Terminology"
abbrev: "AgentProto Reqs"
category: info

docname: draft-rosenberg-agentproto-requirements-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "Agent Communication Protocols"
keyword:
 - next generation
 - unicorn
 - AI-native
venue:
  group: "Agent Communication Protocols"
  type: "Working Group"
  mail: "agentproto@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/agentproto"
  github: "jdrosen/draft-rosenberg-agentproto-requirements"
  latest: "https://jdrosen.github.io/draft-rosenberg-agentproto-requirements/draft-rosenberg-agentproto-requirements.html"

author:
 -
    fullname: "Jonathan Rosenberg"
    organization: jdrosen.net
    email: "jdrosen@jdrosen.net"

normative:

informative:

  RFC3261:

  I-D.rosenberg-aiproto-framework:

  I-D.rosenberg-aiproto-a2t:


--- abstract

The Agent Communications Protocol (ACP) - also known as agentproto - is a new protocol to be developed in the IETF. It is meant to be used between user agent clients and AI agents, or between AI Agents and other AI Agents, or between AI Agents and tools. All three of these use cases - and the many variations therein - all require a session-type protocol which handles lifecycle management, including creation, modification, deletion, and so on. These sessions differ from the type of sessions created by the Session Initiation Protocol (SIP) in several important ways. These sessions can be extremely long lived - running for hours or even days. An example is a lengthy coding session. They can also be highly intermittent with asynchronous responses. All of these mean a new type of session protocol is needed. This document introduces requirements, use cases, concepts and terminology for this protocol.



--- middle

# Introduction {#intro}

The Agent Communications Protocol (ACP) - also known as agentproto - is a new protocol to be developed in the IETF. It is meant to be used between user agent clients and AI agents, or between AI Agents and other AI Agents, or between AI Agents and tools. All three of these use cases - and the many variations therein - all require a session-type protocol which handles lifecycle management, including creation, modification, deletion, and so on. These sessions differ from the type of sessions created by the Session Initiation Protocol (SIP) [RFC3261] in several important ways. These sessions can be extremely long lived - running for hours or even days. An example is a lengthy coding session. They can also be highly intermittent with asynchronous responses. All of these mean a new type of session protocol is needed.

Prior work [I-D.rosenberg-aiproto-framework] discussed these use cases and covered a wide range of requirements. This document builds upon that work, but focuses on aspects specific to the protocol that is to be developed. It introduces requirements, use cases, concepts and terminology for this protocol.

# Use Cases

We consider three core use cases, with sub-cases for each one.

## User Agent to AI Agent

In this case, a human user (which we'll just refer to as a user) is sitting in front of a desktop application, mobile application, or web application providing a user interface for engaging with an AI agent. The AI agent runs elsewhere - on a server - different from the user agent. In this use case, the user agent is really a "thin client" of sorts -  passing information from the user to the AI Agent, and passing information from the AI Agent to the user. This communications protocol would pass user messages, AI agent responses, meta-data like token usage, sub-agent spawns and completions, and so on. If the AI agent supports voice input and spoken output, the communications protocol would pass real-time voice information back and forth as well.  Basically - all of the information needed to render the UI. This use case is very similar to the way in which a SIP user agent talks to a SIP server.

If the user agent is a thick desktop application, the communications protocol is from the user's desktop to the AI agent on the server. In a mobile app, it is from their mobile app to the AI agent on the server. If the user agent is a web application, it is most likely from the web server to the the server on which the AI agent is running.

This use case is different from how desktop AI agents like Claude work today. Those have the AI agent running co-resident with the user interface. There is communications with a server elsewhere, but it takes the form of model invocation and tool invocation.

When the vendor of the AI agent is the same as the vendor of the user agent, there is not necessarily a need for standardized protocols. However, standardized protocols can be helpful in cases when the user agent is an embedded device made by a different vendor than the one making AI agents. For example, consider a case where a user has a smart speaker (similar to Amazon Alexa). However, the smart speaker is made by one vendor (say, Bose, who makes such devices), and the desire is to allow the smart speaker to connect with AI agents made by many vendors - Anthropic, OpenAI, Google, or even B2B AI agents. In this kind of case, when the cardinality of AI agent vendors is high, there is a strong need for a standardized protocol here.

## AI Agent to a Session Oriented Tool

In this use case, an AI agent is running somewhere - a user's desktop, or perhaps on a server. The AI agent wishes to invoke a tool. However, this tool is not just a simple API with an input and output. Examples of a simple input and output tool would be a traditional REST API, or a tool that is exposed through the A2T (Agent to Tool) protocol [I-D.rosenberg-aiproto-a2t]. These are transactional, stateless API calls. There is no session - nothing long lived.

However, there are cases where the tool itself is session oriented. The primary example is the Model Context Protocol (MCP) which is session oriented. The tool might also be agentic in nature - taking structured data as input, providing it ultimately as output, but requiring interaction with the user back and forth in order to fulfill the request.

## AI Agent to AI Agent

In this use case, two AI agents are running in different places. Perhaps they are made by different vendors but running in the same domain. An example might be an enterprise running AI agents from a CRM vendor, which communicate with AI agents made by a provisioning system vendor. They may be in different domains. Or they may be distinct agents made by the same vendor, split up into separate agents on different servers for reasons of scale or isolation.

This is the core use case for A2A.

# Core Concept: Agentic Dialog

When we look at all of the use cases, there is a common theme which runs through all of them - the notion of a long-lived session between two entities which has a growing context window as the interaction progresses, with a defined start and eventual finish. That's a mouthful, so we propose a specific name for this - an agentic dialog.

An agentic dialog is defined as a session between two parties - a principal whose role is to provide input to, and receive output from, an agent - and the agent, which is running a process which has properties which are agentic in nature. The agentic properties that are relevant here are (1) conversational contuity, and (2) driven by natural language in each direction.

The principal can be a human user (as in the user agent to AI agent case noted above), but the principal can also be an AI agent - as in the A2A case. The agent is typically an AI Agent, but it could be another human,  with that human being receiving the input and providing output. In this way, the term "agent" defined here refers strictly to the role in the protocol, which can be fulfilled by a human agent, an AI agent, or even traditional software. Indeed, in the A2T case, the agent might be a piece of software which supports only a small amount of capabilities.

The agentic dialog is facilitated by a protocol, which runs between a client (acting on behalf of the principal) and a server (acting on behalf of the agent). These distinctions are important, because the protocol needs to communicate information which is sometimes relevant only to the client (such as an identifier for the agentic dialog), and sometimes information relevant only to the principal - such as the text that is communicated. This layering is similar to how other Internet protocols operate, e.g, email - which has distinct concepts of envelope (meant for the mail agent) and the body (relevant for the recipient).


## Definitional Properties

An agentic dialog is distinct from other types of sessions (like the SIP dialog) because of two key properties.

### Conversational Continuity

Conversational continuity means that, as far as the principal is concerned, the output generated by the agent appears as if any output has propertly taken into account all of the prior inputs and outputs shared between them. Typically, this is accomplished through context that is built on the agent. The agent initializes context at the start of the dialog, and as the dialog progresses, the input and output are appended to that context. The context may be compacted if it gets too long, but the compaction serves the primary purpose of preserving continuity - retaining what is important so that, to the principal, it is almost as if there was no compaction or loss of information.

The context is used by the agent to determine how to satisfy the needs of the principal as specified via input. Those needs may evolve over time.

Conversational continuity means that the context is - in essence - a piece of state which must be referenced in all aspects of the protocol.

### Driven by Natural Language

In an agentic dialog, the input from the principal to the agent, and output from the agent back to the principal, contain natural language, in written, audio, video, or image form, or even structured data. We use the term "natural language" pretty loosely here. If a user is talking to an AI agent to modify an image, the user might send the image they want modified. This image is a form of "natural language" in our definition. The notion here is that it is processed according to a human or human-like understanding of why the image is being sent. The same is true for structured data - for example a piece of software. The agent may generate a Java file, and send it back to the principal. The information being shared is structured - it is source code - but it is understood as code for a specific purpose based on the context of the conversation.

This means that a protocol for managing agentic dialogs needs to be able to convey these various forms of content in both directions.

## Implied Properties

These two core properties come with other properties that are implied.

## It has a Start

Perhaps obvious but worth stating. The agentic dialog has a start. This start is the point at which the context is initialized. It may not be initialized as empty; the agent might use the identity of the principal to add personalization. Or, it may be a coding agent which reads an agents.md file to initialize what it is doing. It might read system instructions. For the agent to behave properly, it needs to be told that a particular agentic dialog is starting.

## It may be Long Lived

The agentic dialog may last a really long time. By "last" we mean that the context which is its defining property, may continue to exist and be accessible by the principal for long periods of time.

One example is an agentic dialog between a principal and agent that is used for a therapy session. A human user is connected to a client on their desktop; the server is running an AI agent which acts as a virtual therapist. The agentic dialog continues for months.

## Input can be Intermittent

The input from the principal can be intermittent. As an example, consider a user that is using a coding agent, and they have a period of activity where they are typing at it and getting responses, and then they head to bed at some point. THe user comes back in the morning to continue the agentic dialog.

In this case, if the user's the client was on their desktop, and the agent was an AI agent on a server in a cloud somewhere, there is no user input (and potentially no AI Agent output) throughout the entire evening, and no data transmitted from the users' home network towards the Internet through the evening.

This means the protocol needs to account for alternating periods of activity and quiesence. And, because the agentic dialog can be long-lived, the actual IP host which is acting as the client or the server might change over the lifecycle of the agentic dialog.

## Output can be Delayed

The output from the agent could arrive a long time after the input has been sent. Anyone who has used a coding agent is familiar with this - it can sometimes take many minutes before the first response is returned.

## It Might or Might Not End

Agentic dialogs can have a formal end, in which case it is not possible for the client to contribute further to it. If an agentic dialog ends but is actually resumable, it hasnt really ended. Since the agent takes ownership for continued maintenance of context, it needs the ability to declare that it is no longer supporting this context, and ending the dialog. Similarly, a client should be able to say "I'm done" giving permission for the agent to immediately delete the context associated with the agentic dialog.

## Output can be Asynchronous

In early chatbots, each user input created a single user output, resulting in a simple turn-by-turn model of a conversation.

Modern AI Agents can invoke tools that include background activities. As a result, a single user input might generate any number of server outputs, and some of those outputs can come seconds, minutes, or even hours after the initial input.

This complicates the protocol significantly. If the agent needs to generate a response, and that response comes long after the request which triggered it, the server cannot assume the client is still running on the same IP host. Historically, IP protocols have solved for these use cases with webhooks, requiring the client to run its own HTTP server.

## Agents are Named

When a principal establishes an agentic dialog with an agent, it is attempting to do so to a particular named instance of the agent. This named instance is, at the very least, associated with the vendor of the agent. "I want to connect to Claude" or "I want to talk to the Walmart AI Agent". However, there can be additional levels of naming too. One can imagine a vendor having different agents with different personas - a contact center AI agent which handles sales vs. support, or a virtual knowledge worker from Salesforce that handles accounting vs. one that handles legal issues. Generally the differing named agents vary by LLM model, but also through the instructions, prompts, guardrails and other aspects of the harness which define its behavior.

When the principal connects to the agent through its client to initiate an agentic dialog, the client needs to provide to the name to the server, so the server can run the right named instance of the agent.


## Client vs Principal and Server vs. Agent

There is a distinction between the principal and the client. The principal exists outside of the protocol; the role of the protocol is to collect input from the principal, and to provide output back to the principal. In traditional protocols we'd use the word "user" to represent the human driving the protocol, and not "principal". However, that terminology doesn't work here because the principal can be an AI agent which is not a human being. The word "principal" is agnostic to whether it is a human being or an AI Agent. The software element - and it is a piece of traditional software implementing the agent protocol - is called the client.

In the same way, the "agent" exists outside of the protocol as well. It is an AI agent, though it could be a human being. The server is the piece of software - also traditional software - which implements the protocol and is responsible for funneling information to and from the AI Agent.

This distinction is important because it helps us understand how to properly layer the system. The context is a core property of the agent, NOT of the server. In the agentic dialog protocol, we are conveying information to and from the agent on the server, but the context lives with the agent. The protocol will concern itself with technical conveyance issues -  such as retransmissions and reconnection - but it is not a database system and the context itself is not directly accessed by this protocol.


# Architecture Diagram

From these properties, we can draw a picture that defines the operating model of the agentic dialog protocol.

~~~
{::include arch.md}
~~~

The principal accesses the agent through one or more distinct clients over the lifetime of a specific agentic dialog. The notion of "distinct" here is that they can reside on different IP hosts. As noted in the examples above, in an A2A use case, the client might be on a virtual compute node A with a specific IP today, and then tomorrow the principal (an AI agent in this case) resumes the dialog from a different virtual compute node B with a different IP.

In the same way, a particular named agent - and any instance of an agentic dialog created with that agent - can be accessed through a multiplicity of distinct servers - each of which is on a distinct host.



# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations {#security}

TODO Security


# IANA Considerations {#iana}

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
