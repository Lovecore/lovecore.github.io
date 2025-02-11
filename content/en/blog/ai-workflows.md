+++ author = "Nick" categories = ["AI", "Reference", "LLM"] date = 2025-02-05T11:00:00.000Z publishDate = 2025-02-11T11:00:00.000Z description = "This is a breakdown of AI oriented workflows." summary = "This is a breakdown of AI oriented workflows.." draft = false slug = "ai-workflows" tags = ["ai", "reference guide", "Design Patterns", "Prompt Chaining", "Router", "Parallelization", "orchestration-worker", "evaluation-optimizer"] title = "AI Workflows" url = "/ai-workflows" thumbnail = "/images/ai/ai-header-1.png" +++

Welcome back to my small corner of the world. There has been a lot of hype around AI Agents and Agentic Models. In the security space, this is an even more hot topic, because these 'systems' while not exactly new, do require a new methodology of securing them. I have a blog post brewing that is oriented directly to securing AI systems and agents, so stay tuned for that! In the meanwhile, let's talk about the purpose of this post - AI systems, both Agentic and LLM augmented workflows. Now this post isn't going to be a very technical post, the goal of this post is to simplify the AI centric design patterns down, for better understanding. In another post, I'll look to outline Agentic workflows and systems and how they can fit into the larger picture.

I think the first thing to get out of the way is - What is an AI Agent? 

Here's my take:
>An AI Agent is a system or component that LLMs can direct that allows for their own processes and tool usage.

Where as an AI Workflow is system where LLMs and tools are put together via pre-defined code paths.

For any of us that have spent time in the AI space, I think we would agree that a truly Agentic system is hard to build - and often not the correct choice.

Let's take the most basic breakdown of what is probably standard scenario of LLM usage.

>`INPUT` => `LLM` => `OUTPUT`

What we have sort of abstracted away here is the LLM can explicitly use additional resources:

- RAG
- Tools
- Memory

What we end up with is something more like this:

>`INPUT` => `LLM` <==> `Tools/RAG/Memory` => `Output`

This basic design works well for 85% of things. It's when we want to further enrich our output that we need to start looking at better patterns. We also need to decide if the system in question would benefit more from a well designed AI Workflow or an Agentic system.

###  AI Workflows
This is where the fun starts, we're going to quickly cover some of the more popular basic AI workflows.

#### Prompt Chaining
Prompt Chaining can be defined as chaining multiple LLM calls, using the output of one call as input for the next.

![](/images/ai/ai-prompt-chain.png)

We can see in this diagram, that we take the LLM output and feed it again to another LLM call, building on our previous instructions / results. One of the items here is a Control Check. This is a place where the LLM output is verified to ensure that it's still supplying proper data as defined by the system. This can be at any data flow point in a Prompt Chaining system.

####  Parallelization
Parallelization is very similar to the above Prompt Chaining in that we are still leveraging basic LLM calls but we do them asynchronously and aggregate the data at the end. There are actually a few subset version of this - voting, sections or groupings, pattern checked and others.

![](/images/ai/ai-parallelization.png)

This workflow is ideal for when the subtasks can be run parallel to each other for increased speed. The aggregator service then takes your defined method of determining the correct answer and completes the expected data structure. This is a good use case for systems that need different output from each LLM call, rather than one larger topic / output.

#### Routing
Routing is a method for when we are taking the input and directing it to a specific path. Often giving the output to a component that is more specialized in the task.

![](/images/ai/ai-routing.png)

Routers are often very good for tasks that have been well defined. This is often what you tend to see in sales chat bots. You ask for some billing info, the LLM routes that to the billing call and returns some data for you.

#### Orchestrator-workers
Here's a more 'advanced' version of routing - Orchestraction-workers model. In the orchestrator-workers model, a central LLM dynamically breaks down tasks, delegates them to worker LLMs, and synthesizes their results. Note these 'LLM workers' are not explicitly agents, they are coded paths for the work, but the can also be agents!

![](/images/ai/ai-orchestration-worker.png)

Now our Synthesizer is a component that takes the individual outputs from different workers, combines them, and generates a final, cohesive output that is used as input / output further down the line. In the above case, it just gives us the final output. This workflow is great if you do not know what the subtasks of an LLM call could be. An example is having an LLM make complex bulk edits to files in a codebase.

#### Evaluator-optimizer
The Evaluator-optimizer flow is really basic at it's core. We supply an input to the LLM call system, it provides a solution to the LLM evaluator. The evaluator either supplies feedback or rejects the solution. This loop continues until a valid criteria has been met.

![](/images/ai/ai-eval-optomize.png)

This workflow is ideal if you have a clear set of evaluation criteria for a system. It works best when items can be clearly measured. An example cloud be to ask if the results meet the criteria of having a word count or a specific amount of bibliographic sources.

### Key Takeaway
- I think that MANY solution can be achieved with the above patterns - so start simple.
- Starting with a more 'standard' workflow and slowly moving into the Agentic workflows is often the best choice.
- Understanding the scaling of these systems is something that needs to be investigated for any system.
- While building these workflows, look to understand and build guardrails along with them.
