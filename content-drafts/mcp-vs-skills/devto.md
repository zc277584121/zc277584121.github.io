---
title: "MCP Is Being Abandoned: How Fast Can a 'Standard' Die?"
published: false
tags: ai, mcp, agents, skills
series:
canonical_url: https://zc277584121.github.io/ai-coding/2026/03/26/mcp-abandoned-skills-as-better-agent-tool-interface.html
cover_image: https://zc277584121.github.io/images/mcp-vs-skills/context-window-mcp-vs-skills.png
---

In mid-March, Perplexity's CTO Denis Yarats casually dropped a bombshell at the Ask 2026 conference: the company is moving away from MCP internally, going back to REST APIs and CLIs.

![Morgan relaying Perplexity CTO's announcement to drop MCP](https://zc277584121.github.io/images/mcp-vs-skills/morgan-perplexity-drops-mcp.png)

The audience barely reacted, but the statement exploded on social media. YC CEO Garry Tan retweeted it with a blunt "MCP sucks honestly" — it eats too much context window, authentication is broken, and he wrote a CLI wrapper in 30 minutes to replace it.

![Garry Tan publicly criticizing MCP on X](https://zc277584121.github.io/images/mcp-vs-skills/garry-tan-mcp-sucks-tweet.png)

A year ago, this kind of pushback would have been unthinkable. MCP was hailed as the ultimate standard for AI tool integration, ecosystem growth was explosive, and server counts doubled weekly. Now it's been hyped, overused, and rejected.

So what actually went wrong with MCP?

## MCP Is Too Heavy: Context Windows Can't Handle It

A standard MCP setup consumes roughly 72% of the context window. Someone measured it: three servers (GitHub, Playwright, and an IDE integration) burned through 143K tokens of tool definitions on a 200K-token model. Before the agent even starts working, less than 30% of the space remains.

The cost isn't just financial. The more noise packed into the context, the worse the model focuses on what matters. With 100 tool schemas sitting there, the agent has to wade through all of them for every single decision.

![MCP vs Skills context window usage comparison](https://zc277584121.github.io/images/mcp-vs-skills/context-window-mcp-vs-skills.png)

Researchers call this "context rot." The data is stark: tool selection accuracy drops from 43% to below 14%. The more tools you add, the worse the agent gets at picking the right one.

The root cause is MCP's loading strategy — it dumps all tool descriptions into the session at the start, regardless of whether they'll be used. This is a protocol-level design choice, not a bug, but the cost scales with the number of tools.

Skills take a different approach: **progressive disclosure**. At session start, the agent only sees metadata for each Skill — name, one-line description, and trigger conditions. A few dozen tokens total. Only when the agent determines a Skill is relevant does it load the full content.

MCP lines up every tool at the door and asks you to pick. Skills hand you an index and let you look things up as needed.

## MCP Is Too Dumb: It Just Waits to Be Called

MCP is essentially a tool-calling protocol: how to discover tools, how to invoke them, how to get results. Clean for small-scale use, but the "cleanness" is also the limitation — it's too flat.

### No Hierarchy, No Subcommands

In MCP, a tool is a function signature. No subcommands, no awareness of session lifecycle, no knowledge of where the agent is in its workflow.

CLI tools are fundamentally different. A CLI naturally has subcommands — `git commit`, `git push`, `git log` are completely different behavior paths. An agent can run `--help` first, explore what's available, and expand as needed.

![MCP flat tool space vs CLI + Skills hierarchical structure](https://zc277584121.github.io/images/mcp-vs-skills/flat-tools-vs-cli-subcommands.png)

### No SOPs, No Guidance for the Agent

A Skill is essentially a markdown file containing an SOP: what to do first, what to do next, how to retry on failure, when to notify the user. The agent doesn't get an isolated tool — it gets a complete operational playbook.

MCP tools passively wait. Skills actively participate in the agent's workflow.

![Passive tools vs active skills](https://zc277584121.github.io/images/mcp-vs-skills/flat-mcp-vs-hierarchical-skills.png)

### Can't Reuse the Agent's LLM

This one hits close to home. Six months ago, we built the claude-context MCP project to provide contextual code retrieval for Claude Code. When a user asked a question, MCP retrieved relevant conversation fragments from a Milvus vector database and fed them back into the context.

The problem: out of top 10 retrieved results, maybe 3 were useful. The other 7 were noise. I tried several approaches — dumping everything to the agent (too noisy), adding reranking inside the MCP server (small models weren't accurate enough), using a large model (but the MCP server is a separate process that can't access the outer agent's LLM).

![MCP needs separate LLM vs Skills reuse the agent's own LLM](https://zc277584121.github.io/images/mcp-vs-skills/llm-reuse-mcp-vs-skills.png)

Skills solve this. A retrieval Skill can be structured as: run vector search for top 10, then let the agent's own LLM judge relevance and filter noise. No extra model, no extra API key.

Our later project memsearch used this approach — three-layer progressive retrieval where the agent's LLM participates throughout the decision-making process, keeping only curated results visible to the main conversation.

![memsearch three-layer progressive recall flow](https://zc277584121.github.io/images/mcp-vs-skills/memsearch-progressive-recall-flow.png)

## But Does MCP Deserve to Die?

MCP has been donated to the Linux Foundation, has over 10,000 active servers, and SDK downloads hit 97 million per month. That ecosystem doesn't just die overnight.

Some developers take a moderate view: Skills are like a detailed recipe; MCP is the tool. Both have their place.

![Abhay Bhargav: Skills are recipes, MCP is tools](https://zc277584121.github.io/images/mcp-vs-skills/abhay-skills-recipe-mcp-tools.png)

It depends on the scenario. MCP over stdio — the mode most developers run locally — has the most issues. But MCP over HTTP is different: enterprise tool platforms need unified permission management, centralized OAuth, standardized telemetry — things scattered CLI tools genuinely struggle to provide.

![MCP's two faces: local stdio mode vs enterprise HTTP mode](https://zc277584121.github.io/images/mcp-vs-skills/mcp-stdio-vs-http-comparison.png)

MCP probably won't disappear. It will retreat to where it fits — enterprise tool platforms, scenarios requiring centralized governance. But the claim that "every agent should use MCP" no longer holds up.

## What's Next

The most pragmatic combination right now is **CLI + RESTful + Skills**: CLIs for everyday operations, RESTful APIs for external system integration, Skills for agent behavioral knowledge. MCP hasn't vanished, but its position has shifted — from "the standard" to "one option among several."

MCP left the industry with a valuable question: what kind of tool interface do agents actually need? The answer is slowly taking shape — it's just no longer something a single protocol can provide.
