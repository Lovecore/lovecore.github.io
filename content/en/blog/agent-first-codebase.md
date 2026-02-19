+++
author = "Nick"
title: "Your Codebase Has a New Reader — And It Doesn't Think Like You"
date = 2026-02-17T18:00:20.000Z
publishDate = 2026-02-17T18:00:20.000Z 
draft = false
slug = "agent-first-codebase"
url = "/agent-first-codebase"
tags = ["agentic-coding", "AI engineering", "developer experience", "security", "AGENTS.md", "Claude Code"]
categories = ["AI Engineering"]
description = "We've spent decades optimizing codebases for human engineers. In the age of agentic AI, it's time to ask who we're really writing code for."
thumbnail = "/images/ai/agent-first.jpg"
+++

# Your Codebase Has a New Reader, And It Doesn't Think Like You

Hey everyone, you like the AI generated title? I promise that the only AI thing written here! Oh.. the blog image is also AI generated... but that's it I swear!

So this one's been sitting in my drafts for a while because, well I just need more time. I've contributed to a lot of codebases over the past year and a trend I've noticed is that many codebases are written or forced even to have humans in mind. Here's another hot take but AI isn't struggling because of its reasoning capability. It's struggling because the codebase wasn't built with it in mind. The more I dug into this, the more I realized many are hitting the same wall. We've been writing code for humans for so long and we haven't really stopped to ask whether that still makes sense.

This ~~post~~ hot take is about that gap. The gap between what makes a codebase readable to a skilled engineer and what makes it readable, and executable, to an AI agent. They are not the same thing. And in 2025/2026, I think that difference is starting to really matter. A lot of this might seem like common sense but I think it's worth calling out and discussing. 

## The World We Designed Code For

For most of software engineering's history, "readable code" meant one thing: a new human could sit down, clone the repo, and within a few days start making meaningful contributions. We optimized and worked for that. Clean naming conventions, thoughtful folder structures, elegant abstractions, and comments that explained why. All the Clean Code stuff. Thoughtfully named PRs with detailed descriptions. Wikis, Confluence pages, Slack threads full of context.

All of this worked because the reader had superpowers we never thought to document. Institutional memory, social context, the ability to ask a colleague, the judgment to smell a bad abstraction before touching it, and the pattern recognition to know that the auth service on a Friday afternoon is a do not touch area.

AI agent's don't have any of that.

## What Agent-Readable Code Actually Means

So what does it look like when a codebase is built with an agent in mind? Based on what teams running genuinely agent-first workflows have learned, a few clear patterns have emerged.

### All Context Lives in the Repo

I think this is the most important shift that we see teams making. Treat the repository as the source of truth for everything the agent needs to know, not just the code but the reasoning behind it. Architecture Decision Records need to live as versioned markdown next to the code they govern, not in a wiki the agent can't access. Domain glossaries, deprecated patterns to avoid, the golden path that guide how the codebase should evolve, all of it needs to move into the repo.

The [AGENTS.md standard](https://agents.md/) has emerged as the canonical format for this supported by Codex, Cursor, GitHub Copilot, and others. Think of it as a README but written for your AI collaborators rather than your human ones. Where `README.md` answers "what is this project and how do I get started?", `AGENTS.md` answers "what do you need to know to work on this codebase without asking anyone anything?"

If you're running Claude Code specifically, the equivalent is `CLAUDE.md`. Same concept, different tooling. Claude reads it automatically at the start of every session, and you can place them in a few different spots depending on scope: `~/.claude/CLAUDE.md` applies globally across all your projects, `./CLAUDE.md` at the project root is the one you commit to git and share with the team, and `CLAUDE.local.md` is your personal override that stays gitignored. For monorepos, both the root file and any subdirectory `CLAUDE.md` get pulled in automatically when Claude is working in that area. You can also import other files directly using `@path/to/file` syntax, so your root file can stay lean and just point outward to more detailed docs.


### Structure is Enforced, Not Emergent

Human readable codebases can tolerate emergent structure. A seasoned engineer can infer data flow through abstractions, trace implicit conventions, and fill in the blanks with judgment. Agents can't do that reliably. Ambiguity that a senior engineer resolves in seconds becomes hallucination fuel for an agent.

The [Harness team at OpenAI](https://openai.com/index/harness-engineering/) learned this firsthand. Their agent-optimized codebase runs on a rigid, mechanically enforced architectural model where each business domain is divided into a fixed set of layers, with strictly validated dependency directions and a limited set of permissible edges. These constraints aren't just documented, they're enforced by custom linters and structural tests. If the agent violates the architecture, the pipeline tells it immediately.

The lesson here is simple. Good human readable code uses conventions. Good agent-readable code enforces invariants.

### Golden Principles Need to Be Mechanical

The Harness team introduced this concept of "golden principles," opinionated and mechanical rules that keep the codebase consistent for future agent runs. Things like: prefer shared utility packages over hand-rolled helpers to keep invariants centralized, or validate data shapes at boundaries with typed SDKs so the agent can't accidentally build on guessed shapes.

The key word is `mechanical`. Agents don't internalize wisdom through exposure the way a human does. You can't write "prefer composition over inheritance" in a doc and expect an agent to apply it with nuance across hundreds of files. You need linters, structural tests, and automated feedback loops that catch violations immediately and tell the agent exactly what's wrong.

Please take a look at the [Harness team's blog post](https://openai.com/index/harness-engineering/) for more details. I really recommend reading it.

### Examples Beat Abstractions

This one is subtle but it matters. Human developers can read a style guide that says "components should be modular and follow single responsibility" and largely extrapolate correctly across our work. Agents learn better from concrete examples. The team at [Builder.io](https://www.builder.io/blog/agents-md) documented this well when building out their AGENTS.md guidance. Instead of writing abstract rules, point to real files in the codebase. "For forms, copy `app/components/DashForm.tsx`", "avoid class-based components like `Admin.tsx`, prefer functional components like `Projects.tsx`". Point to the good. Point to the bad. Let the agent learn by analogy.

### Progressive Disclosure of Context

One of the early mistakes people make is stuffing everything into a single giant `AGENTS.md` or `CLAUDE.md` at the root level. This fills every agent session with context that's only relevant 10% of the time and it actively degrades performance by eating up the context window.

The pattern that has shown to work better is more of aprogressive disclosure. A root-level file gives high level orientation and points to module-specific context, with more detailed agent docs living close to the code they describe. API specific rules live in `src/api/AGENTS.md`. Component guidelines live in `src/components/AGENTS.md`. Agents navigate these hierarchies efficiently and pull in context only when they're actually working in that area of the codebase.

## The Security Angle Nobody's Really Talking About

Now here's where I want to put on my hat for a moment, because there's a dimension to agent readable codebases that most engineering posts aren't covering seriously yet. The security angle.

When you design your codebase to be agent readable, when you encode architectural context, domain knowledge, and system boundaries into versioned artifacts, you're also creating a richer attack surface for prompt injection and context manipulation. An agent that reads `AGENTS.md` or `CLAUDE.md` and acts autonomously based on its contents. However, this is also an agent that could be influenced by malicious content that finds its way into that context chain.

Think about what an agent first codebase looks like from an attacker's perspective. A compromised dependency that also ships a malicious `AGENTS.md` or `CLAUDE.md`. A poisoned third-party documentation reference the agent is instructed to read. A PR that contains subtle modifications to context files that alter agent behavior for anyone who runs sessions against that branch. These aren't hypothetical. They're natural extensions of threats we already understand, just expressed differently.

I think there are a few things worth building in from the start:

Treat `AGENTS.md`, `CLAUDE.md`, and similar files as security sensitive artifacts. They go through code review. Changes are audited. They're not just documentation, they're behavioral configuration for autonomous systems.

Don't give agents access to production credentials, secrets, or write access to infrastructure just because they need to run tests. Scope permissions explicitly. Think least privilege, just like you would for any service account.

If your agents are reading external docs as part of their workflow, apply the same scrutiny you'd give to any external input. Prompt injection through documentation is real, and it's something we've already covered in the [AI Attacker Reference](https://rootflag.io/ai-attacker-reference/) series.

As agentic coding matures, threat modeling for agent readable codebases is going to become its own discipline. I've got a post brewing on exactly that, so stay tuned.

## This Isn't an Either/Or

Before this comes across as "throw away everything and rebuild for agents", I want to push back on that framing. Agent readability and human readability are not opposites.

The Venn diagram has shifted, not collapsed. Things that have always been good engineering practice, clear naming, well defined module boundaries, documented intent, consistent patterns, are good for both audiences. The shift is in *where* context lives and *how* structure gets enforced, not in whether you should have context and structure at all.

What agent readability is really asking us to do is be more disciplined about things we've always known were important but often let slide. Putting decisions in durable versioned artifacts instead of people's heads. Enforcing architectural conventions mechanically instead of hoping everyone internalizes them. Keeping documentation close to the code it describes so it doesn't drift.

The teams doing this well aren't sacrificing human ergonomics. They're building codebases that are better for both audiences. The agent is just forcing them to be honest about work they probably should have been doing anyway.

## Where to Start

If you want to move in this direction without blowing up your existing workflow, here's a reasonable starting point.

Audit where your architectural context currently lives. Notion, Confluence, Slack threads, people's heads? That's your gap. Even picking the three most important architectural decisions that live outside the repo and writing them into a `docs/architecture/` folder as markdown is a meaningful first step.

Then create your `AGENTS.md` or `CLAUDE.md`. Keep it lean at first. High-level orientation, key commands, a few "this, not that" examples pointing at real files, and pointers to where the detailed docs live. Treat it as a living document, not a one-time setup.

Finally, start thinking about invariant enforcement. What are the rules in your codebase that matter enough to break a build if violated? Are any of them currently enforced only by convention? Pick one and automate it. That's the beginning of a codebase that an agent can work in without drifting.


## Where This Is Going

Here's my honest prediction. Within the next couple of years (months?), code review will include reviewing how well a PR updates the agent context artifacts, not just the code itself. The `AGENTS.md` will be as load bearing as your CI/CD configuration. Teams that neglect it will accumulate what I'd call agent drift, a growing gap between what the codebase actually is and what agents think it is, with compounding errors on every autonomous task.

Many of you that know me know that I'm a big fan of [spec-driven development](https://github.com/github/spec-kit). I've done many webinars and projects with it. The [spec-driven development movement](https://redmonk.com/kholterhoff/2025/12/22/10-things-developers-want-from-their-agentic-ides-in-2025/) being pushed by tools like AWS Kiro and GitHub's Spec Kit is already pointing in this direction. Structured `requirements.md`, `design.md`, and `tasks.md` files that function as a contract both humans and agents can reference. It's a new layer of artifact that didn't really exist in traditional codebases, somewhere between code and documentation, and teams that embrace it early are going to compound their velocity advantage over teams that are still bolting AI tools on top of codebases designed for humans alone.

We're at the beginning of figuring out what it means to design software for a world where agents are first class contributors. The programming languages haven't changed. The audience has.

That's it for this one. Next up I want to get into the security side of this more deeply, specifically what a threat model looks like when the "user" of your codebase context is an autonomous agent. That should be a fun one.

As always, I would love to know how others are thinking about this.

Thanks for reading!