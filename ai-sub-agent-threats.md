+++
author = "Nick"
categories = ["AI", "Reference", "LLM", "agentic", "agentic design", "emergent behavior", "STRIDE", "Threat Model", "AI Agent Threats"]
date = 2025-06-17T11:00:00.000Z
publishDate = 2025-06-17T11:00:00.000Z
description = "This is a post on vulnerabilities in sub agent systems."
summary = "This is a post on vulnerabilities in sub agent systems."
draft = false
slug = "ai-sub-agent-threats"
tags = ["AI", "Reference", "LLM", "agentic", "agentic design", "emergent behavior", "STRIDE", "Threat Model"]
title = "Security vulnerabilities in agent-subagent design patterns"
url = "/ai-sub-agent-threats"
thumbnail = "/images/ai/ai-header-4.jpeg"
+++

Welcome back! I've had a bit less time to write anything recently - but now I'm on a vacation with family, so while they're all sleeping in, I have a chance to get some of the things rattling around in my head down! I had recently created a repo and a post on leveraging LLM-Guard as both a prompt linter as well as a robust LLM security input layer. It was a bit too light and really wasn't anything that you couldn't already dig up anywhere else if you set to it. 

So for this post, I wanted to talk about vulnerabilities in agents and sub-agent systems. As we move into a more agent centric phase of software and tooling, I think it's important that we realize that these design patterns have the potential to create a different vulnerabilities that are quite different than your standard security concerns.

Modern AI agent frameworks like LangChain, Auto-GPT, LlamaIndex, Microsoft Autogen and others have made it easy to build these multi-component systems. As we start to adopt these frameworks in applications, it’s critical for security engineers and architects to understand how agent–subagent patterns work, what could go wrong, and how to design these systems more safely. Now I'm no expert, but here are my thoughts.

### Agents you say?

As I talked about in previous blog posts, there are multiple types of agentic systems out there. As a quick refresher, let's recap what some of these are:

We have single agent systems, which are just like they sound, one agent that completes it's workflow from end to end.

We have multi agent systems, which - you guessed it - are systems that have multiple agents working collectively, sharing resources and are generally coordinated to complete a given goal.

Then we've got Hierarchal multi agent systems. These are the newer system that essentially organizes agents into layers. Higher level agents, usually the supervisors/orchestrators delegating tasks to and managing lower level agents - our sub-agents/workers.

I should note that within each of the above, there are even sub patterns. Patterns like decentralized agent system and meta planning / tiered planning.
###  Pattern comparisons?

Let's now takes these and just apply basic infosec knowledge and lens to them and understand (or remember) some of the common attack patterns in the traditional domain and look at how they apply.
#### Hierarchical vs. flat agent architectures

**Hierarchical architectures** introduce delegation chain vulnerabilities where each delegation step can weaken security controls. Authority dilution occurs as the first security constraints are lost through multiple layers. Circular delegation creates can create logical paradoxes and security bypasses. However, they offer better task specialization and scalability for complex operations.

**Flat architectures** reduce delegation based attack vectors but can suffer from coordination challenges and limited scalability. While they eliminate hierarchical trust issues, they create new problems with peer-to-peer trust establishment and consensus mechanisms.

#### Synchronous vs. asynchronous communication patterns

**Synchronous patterns** enable immediate attack detection and response but create blocking vulnerabilities where malicious agents can stall entire systems. 

**Asynchronous patterns** improve resilience against DoS attacks but complicate attack tracking and enable time-based exploitation.

#### Centralized vs. distributed control

**Centralized control** creates single points of failure but simplifies security monitoring and policy enforcement. 

**Distributed control** eliminates central attack targets but introduces complex trust relationship management and makes coordinated security responses challenging.

### But what's the vulnerability here?

At this point, I hope that most of the world is familiar with OWASP Top 10 - in all aspects. Each of the OWASP Top 10 can be applied to each agent within a system to help get clarity as to where an agent can fail. Let's go over a few basic example:

Let's use everyone's favorite 'unsolvable' problem, [Prompt Injection]() and apply it to our Hierarchal design pattern where this attack becomes exponentially more dangerous. Direct prompt injection now propagates through delegation chains, with parent agents unknowingly passing malicious instructions to subagents. More sophisticated indirect prompt injection attacks embed harmful instructions in external or even internal data sources like emails, documents, and webpages. Agents consume these resources during legitimate tasks. 

We can easily see how this is a problem, but what if we took this example a step further? What if the attackers craft initial injections targeting parent agents with instructions to manipulate subagent communications? The attack chain can propagate through multiple hierarchical levels, with each layer believing it received legitimate instructions from trusted sources. This creates a cascading failure scenario where a single compromised agent can corrupt an entire network.

The basic idea above is becoming known as a Prompt Infection. Infected agents modify their outputs to include infection vectors, spreading the attack silently through agent networks via normal communication channels. With persistence achieved through agent memory and communication logs, these attacks can achieve very dangerous outcomes.

The core issue here is the breakdown of the data/instruction boundary as information flows between agents and from external sources. One agent’s output becomes another agent’s input, and if that input contains instruction-like text, the second agent's LLM might prioritize it over its original system prompt or task.

Let's go over another example, this one we'll talk about confidentiality. Exposure of sensitive information is quite common in agentic patterns. This can happen when a supervisor passing too much data to a sub-agent, or a sub-agent improperly accessing/returning data. This could even happen from an agent to an unauthorized external entity.

**Excessive Data Sharing:** A supervisor agent might pass a complete user profile to a sub-agent that only needs the user's ID for a specific task.

**Sub-agent Overreach:** A sub-agent, due to overly broad permissions or being tricked by a prompt, accesses or returns more data than it was authorized for. For instance, a sub-agent asked for an order status might return the customer's entire purchase history and payment details.

**Insecure Communication Channels:** If the communication paths between the supervisor and sub-agents (or between peer sub-agents) are not encrypted or are otherwise vulnerable to interception.

**Leaky Tool Outputs**: Tools used by sub-agents might return verbose outputs containing sensitive details, which are then propagated through the agent system if not properly filtered.

**Intellectual Property (IP) Leakage**: Adversarial queries can propagate through multi-agent systems, extracting sensitive information like system prompts, task instructions, tool specifications, number of agents, and even the system topology.

The fundamental issue is often a lack of granularity in data access and transfer. Combine this with insufficient sanitization of data as it moves between components and we have a potential data nightmare.

This list can go on and on, there's no shortage of threats from the 'traditional information security' domain that can be applied in this manner. The key here is understanding that applying this method has benefits but can often require thinking a bit differently about the system.

### It gets worse?!

Yep, it does - sorry. These are the 'basic' attacks, nothing extra fancy happening here. As we know, when we wrench up complexity, we also dial up the risk. In complex agentic systems, mostly those involving multiple agents and sub-agents, an initial security failure in one component can trigger a chain reaction, leading to a far more significant compromise. This cascade should be a critical concern for designers and security engineers.

Some more examples of this could be:

**Prompt Injection => Data Exfiltration:** A successful prompt injection might trick a sub-agent into misusing one of its tools. An agent designed to query a database for non-sensitive information might be manipulated into executing a query that extracts sensitive user credentials. These credentials could then be used by the attacker to access the database directly, bypassing the agentic system entirely.

**Data Poisoning => Malicious Actions:** Consider a scenario where a sub-agent responsible for ingesting and processing external news feeds (data retrieval) encounters a "poisoned" news article. This article contains subtle misinformation or even embedded commands. The sub-agent, now operating on tainted data, might generate flawed analyses or recommendations. Now if a supervisor agent trusts this sub-agent's output without independent verification, it could trigger false action, disseminate misinformation, or initiate other harmful actions based on the poisoned intelligence.

**Weak Tool Authentication => System Control:** If a sub-agent uses a tool that requires authentication (like an API key for an external service) but this authentication is weak or the key is poorly managed (overly permissive or even hard coded), compromising the sub-agent could grant the attacker access to this key. The attacker could then use the tool directly with the sub-agent's (or system's) privileges, potentially modifying data, disrupting services, or accessing other connected systems.

I think the attack above highlight the threat of chained attacks. I also think they highlight the fact that impact of these are greater than the initial vulnerability. I still think there are a few hidden considerations here. Often these chained threats, fail silently. By that I mean the governing system doesn't crash like 'traditional' systems might. The system still continues to work, but it can still work in and towards the wrong direction. An initial compromise, like as a subtle prompt injection or memory poisoning, might not cause immediate or obvious errors. Instead, the affected agent subtly alters its behavior, data processing, or decision-making. This altered behavior then influences other agents or processes down the chain. The overall system continues to operate, appearing functional, but its integrity is compromised, and it may be unknowingly working towards an attacker's objectives.

Then there's the problem of agency in these systems. When an agent is granted overly broad permissions or too much autonomy, and an attacker manages to influence this agent, the excessive agency provides the attacker with a wide range of potential pivot points and actions. A compromised agent with extensive file system access, could be directed to read sensitive files, delete critical data, or write code. All because it was overly permissioned. 

### How do we fix it?

This is the question, right? How do we keep agents true to intent? Identifying vulnerabilities is only the first step. Implementing robust mitigation strategies is crucial for building secure and trustworthy agentic systems and even then, it's still not solved and can be bypassed with enough effort and time. But we can get back to the basics to help highlight a path forward.

#### Robust Input Validation and Sanitization

This is a foundational security measure. Strict validation and sanitization of all inputs that an agent or its components process. This includes data directly provided by users, information retrieved from external sources by tools (web pages, database results, external sources), and data passed between agents in a multi-agent system. Effective input validation is the primary defense against various forms of prompt injection and data poisoning. 

Some standard techniques here are, allowing listing, block listing, regex, ensure that data input is properly structured, output validation and context aware filtering.

#### **Principle of Least Privilege for Agents and Tools**

The Principle of Least Privilege dictates that every component in a system, each agent (supervisor or sub-agent) and each tool it uses, should only be granted the absolute minimum set of permissions, access rights, and capabilities necessary to perform its specific, intended function and nothing more.

**Sub-agent Scoping:** Sub-agents must have narrowly defined roles. They should only be given access to the specific tools, data sources, and API endpoints relevant to fulfilling that role. A WeatherReportSubAgent should only have access to a weather API, not a customer database.

**Supervisor Delegation Control:** Supervisors should not have unrestricted ability to command sub-agents to perform any action. Their delegation capabilities should also be scoped. A supervisor might be allowed to ask a sub-agent to read a file but not delete it.

**Tool Permissions:** Tools themselves must operate with least privilege. If a tool interacts with an external API, the API key it uses should be scoped to only the necessary permissions (read-only access if write access is not required for the agent's task). Credentials should be managed securely through a dedicated secrets manager.

#### Secure Sandboxing and Environment Isolation

This one is key to any group leveraging agents to write code. Agent processes, especially those that execute untrusted code, interact with the file system, or make arbitrary network requests, must be run in isolated, restricted environments. Sandboxing aims to contain the potential damage if an agent is compromised or behaves maliciously.

#### Secure Tool Design, Integration, and Usage Policies

Tools are the primary way for agents to interact with the external world and are a significant potential attack surface. They must be designed with security in mind, integrated carefully, and used according to strict policies.

Some mitigations here are probably familiar: input/output validation, parameterize tool calls, authentication for your tools, rate limiting and error handling, make sure that your tool usage is audited.

#### Comprehensive Monitoring, Logging, Auditing, and Anomaly Detection

Back to another core security principle here. Implement robust systems to continuously monitor agent behavior, log all significant actions and decisions, conduct regular audits of these logs, and employ anomaly detection techniques to identify patterns that might indicate a compromise, malfunction, or deviation from intended behavior.

Comprehensive logging in agentic systems should include full prompts sent to LLMs and their responses, details of tool calls including the agent, tool, parameters, and output, the content of inter-agent messages, significant changes to shared state in systems like LangGraph, all errors and exceptions, and records of Human-in-the-Loop interventions, specifying what was reviewed, who reviewed it, and the final decision.

#### Leverage Penetration testing

Designing a secure agentic system is a complex, multi-faceted challenge. We can do all of these things above, and more, and still be at the mercy of the attacker. This is where penetration testing, moves from a best practice to an absolute necessity.

It's important to understand that AI centric testing can take time and can be very costly. This should be considered when putting out the product roadmap. 

### The End?

Not likely. I compiled quite a few resources for this blogpost. I'm still wrapping up the code to accompany these threats, risks and mitigations as well, so we can see what they look like in the LangChain / LangGraph environment. 

The larger take aways here are: 

Architectural design of an agentic system significantly defines its attack surface. It even creates sub-surfaces within it's agent systems.

Effective mitigation also relies on continuous oversight. Comprehensive monitoring and logging of all agent decisions, tool calls, and communications are crucial for detecting anomalies and performing forensic analysis. Observability tools like LangSmith are invaluable for this purpose. 

For critical or irreversible operations, implementing a Human in the Loop approval step is a vital safety net to prevent catastrophic errors. Frameworks like LangGraph are explicitly designed to facilitate such interruptions for human review. 

Ultimately, building trustworthy agentic AI requires a security-first mindset, where the potential for failure is anticipated and a layered defense is integrated into the system's design from the very beginning.
