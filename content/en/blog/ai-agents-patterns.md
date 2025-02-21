+++
author = "Nick"
categories = ["AI", "Reference", "LLM", "agentic", "agentic design", "multi agent", "tool-use agent", "reflection agent", "planning agent", "tools", "planning", "Multitool", "reflection"]
date = 2025-02-11T11:00:00.000Z
publishDate = 2025-02-11T11:00:00.000Z
description = "This is a breakdown of AI and Agentic Design Patterns."
summary = "This is a breakdown of AI and Agentic Design Patterns."
draft = true
slug = "ai-agents-patterns"
tags = ["AI", "Reference", "LLM", "agentic", "agentic design", "multi agent", "tool-use agent", "reflection agent", "planning agent", "tools", "planning", "Multitool", "reflection"]
title = "AI Workflows"
url = "/ai-agents-patterns"
thumbnail = "/images/ai/ai-agents-header.png"
+++

We're back! In the [first blog post](https://rootflag.io/ai-workflows/) in this series, we talked about some LLM powered workflows and how they aren't agentic solutions or design patterns, but they can be. In this entry I want to touch on the hype of Agentic Design Patterns.

We all know everyone is all about agents and agentic solutions right now. The goal of agentic systems is to really make a system more autonomous. Hopefully, after reading this post, you'll be able to look at a design pattern and draw a parallel to how many humans breakdown problems as well.

### Agent based design patterns
Let's go over some of the more common design patterns we're seeing currently. 

#### Tool Based
The Tool Based pattern is one of the most common agentic design patterns currently. It has been largely adopted due to how much it can significantly increase the usefulness of a system. Being able to give an Agent the ability to query a dataset, search the web for an answer, have memory and additional functionality is a very strong system expander.

![](/images/ai/agent-tools.png)


#### Multi Agent
Multi Agent systems work on the idea of delegation to different agents. When something is passed off to an agent, it is assume that the particular agent has a specific LLM based purpose and will return a result within the expected structured response.

![](/images/ai/agent-multi.png)

There are a few types of multi agent patterns:
- Hierarchical Agent systems: A system that has structured agents with 'sub' agents that have larger conditional decision making.
- Supervised Agent systems: A system that has a agent supervisor that coordinates agent tasks, results and verification.
- Collaborative Agent Systems: A system that uses agents to build on each other for a singular final results.

#### Planning Breakdown
The Planning Breakdown pattern is based on breaking down a larger problem into smaller and more consumable chunks. This pattern is easiest to think about in terms of a roadmap. The system will break down the request into a form of roadmap. The agent and/or system can then iterate on the results of the roadmap as it completes them. 

![](/images/ai/agent-planning.png)

Just like multi agent systems - planning systems have additional types as well.

#### Reflection
The Reflection pattern is often considered in this grey area of agentic design. It doesn't necessarily need an agent to work. This pattern is based on evaluating and refining the output until a particular validation scenario is met.

![](/images/ai/agent-reflection.png)

A very common use case for this pattern is RAG - namely [SELF-RAG](https://arxiv.org/abs/2310.11511).

### Patterns, now what?
Great, so we have a quick breakdown of some of the common agentic design patterns above - but what now? Do we just go out and build things using them? Well, yes. I think that to have a better understanding of security in the LLM space, you should really be building something and understanding it from the inside out. Which leads to the main purpose of the first post and this post...

### When a Complex Workflow May Not Justify an Agentic Pattern

#### Over-Engineering Risks 
A dynamic, multi-step process doesn't automatically mean it needs an agentic system. If the complexity of your workflow can be managed through simpler, well-established techniques—such as a series of predefined tasks, retrieval-augmented generation, or in-context examples then introducing agentic patterns might result in unnecessary over-engineering. The additional overhead could lead to a system that is harder to maintain and debug without delivering benefits.

#### Increased System Complexity and Maintenance  
Agentic systems inherently add layers of complexity. They require careful orchestration of autonomous decision making processes, which can complicate error handling and system monitoring. If the problem can be solved with a straightforward solution, the extra complexity might introduce more points of failure. This makes the system less predictable and increases the effort required for ongoing maintenance and a potential security nightmare.

#### Resource and Performance Considerations 
Deploying an agentic pattern may demand additional computational resources due to its iterative nature and the need for continuous context evaluation. For applications that don't truly benefit from adaptive behavior, these extra resource demands will increased operational costs.

#### Uncertainty in Decision-Making  
While agentic systems excel in environments where adaptability is key, they can also produce unpredictable behavior if the problem isn’t sufficiently defined to warrant their use. Simpler, linear workflow approaches are often more transparent, making it easier to trace decision paths and ensure consistent outcomes. In contrast, the iterative, and 'closed' decision loops in an agentic system can make it challenging to pinpoint issues when things go awry.

### What's next? 
Well, now that these items have been covered, we can now explore the security implications of these systems. What does securing them look like? Do we have new security controls in this space? How are these new security vulnerabilities weighted against the 'classic' CVE, CWE, ExPRT ect ratings? Do agents need their own scoring system? 

I hope to answer many of these questions in the entries to come! Thanks for the read!
