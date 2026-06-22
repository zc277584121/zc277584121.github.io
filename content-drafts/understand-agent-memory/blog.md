---
layout: post
title: "Understanding Agent Memory: Four Layers from Chat History to Skills"
categories: AI-Agent
---

Agent memory has been everywhere lately.

I have been working on this problem from the engineering side for the past few months, and the more I build, the more I feel the field is messy for one simple reason: people are using the same word, "memory", to mean very different things.

Sometimes memory means chat history. Sometimes it means a vector database. Sometimes it means a Markdown file. Sometimes it means a skill the agent can use later. And sometimes it means the whole context layer around the agent.

That sounds confusing, but I think the confusion mostly comes from one missing question:

**What exactly counts as memory for an agent?**

Once that question is clearer, the taxonomy, architecture choices, and trade-offs become much easier to reason about.

Here is the four-layer model I use.

## Layer 1: The Early View of Memory

The earliest and simplest split was short-term memory vs. long-term memory.

Short-term memory is whatever still fits in the model context window, plus small local notes that capture temporary state or user preferences. Many coding assistants have some version of this: a Markdown file in the workspace, a project instruction file, or a lightweight memory page that gets loaded into context.

Long-term memory usually means storing a much larger history of conversations and retrieving relevant pieces on demand.

![A memory page in a coding assistant](https://zc277584121.github.io/images/understand-agent-memory/claude-memory-page.png)

This view is easy to understand, but it is narrow. In this framing, memory is mostly "what you and the agent talked about before."

That was useful as a starting point, but it does not explain what modern agents are becoming.

## Layer 2: The CoALA Taxonomy

In September 2023, the CoALA paper, [Cognitive Architectures for Language Agents](https://arxiv.org/abs/2309.02427), proposed a much broader way to classify memory. LangChain later used the same framing in its [Memory for Agents](https://www.langchain.com/blog/memory-for-agents) post.

![CoALA paper overview](https://zc277584121.github.io/images/understand-agent-memory/coala-paper-overview.png)

CoALA splits long-term memory into three important types:

| Memory type | What it means for agents | Common implementation |
|-------------|--------------------------|-----------------------|
| Procedural memory | What the agent knows how to do | Skills, workflows, tools, SOPs |
| Semantic memory | Facts and knowledge the agent can use | RAG, knowledge bases, documents |
| Episodic memory | What the agent has experienced | Conversation history, task history, event logs |

There is also working memory, which maps roughly to the short-term context we already talked about.

At first, these terms can feel academic. But looking back from 2026, they map surprisingly well to how agents actually evolved.

Procedural memory is skills. It is not just "knowledge about a task"; it is the repeatable procedure for getting the task done.

Semantic memory is RAG. It is the agent's access to facts, documents, manuals, product knowledge, codebase knowledge, or any external source of truth.

Episodic memory is the original "memory" people talked about: past conversations, previous sessions, and what happened during earlier work.

![Three major types of agent memory](https://zc277584121.github.io/images/understand-agent-memory/agent-memory-types.png)

This is the first shift I think matters: agent memory is not only chat history. It includes skills and knowledge too.

The human analogy helps. What is a person's memory?

It is partly what they have experienced. That is episodic memory.

It is partly what they have learned from books, documents, and the world. That is semantic memory.

And it is partly what they know how to do: cooking, debugging, negotiating, writing, shipping software. That is procedural memory.

Agents are starting to need the same three categories.

## Layer 3: Should Memory Live in Markdown?

OpenClaw made one design choice that I find especially important: memory is stored in plain Markdown files. The Markdown file is the source of truth. Any index built on top of it, such as a vector index, is only a derived structure.

![OpenClaw stores memory in plain Markdown files](https://zc277584121.github.io/images/understand-agent-memory/openclaw-memory-markdown.png)

This has obvious benefits.

Markdown is easy to read. It is easy to edit. It can be versioned with Git. It is transparent to the user. If the user wants to delete or correct a memory, they can do it directly instead of digging through a database table or a hidden service.

That was also one of the core ideas behind [MemSearch](https://github.com/zilliztech/memsearch): extracted memories should be visible, editable, portable, and searchable.

But I think there is a deeper reason Markdown works well here:

**The natural shape of memory is language.**

Most real memories are not clean rows in a table. They are conditional, messy, exception-heavy, and context-sensitive.

For example:

> The user prefers shipping on Friday afternoon, but if the release is large, delay it to Monday unless the PM is pushing.

Try putting that into a rigid schema. What is the release day? Friday? Monday? Where do the conditions go? What about the exception?

This is what most useful memory looks like. It carries assumptions, edge cases, tone, and context. If you force it into a pre-designed structure too early, you often lose the part that made it useful.

Natural language is powerful precisely because it can hold these irregular shapes.

I like Geoffrey Hinton's analogy that language is like Lego. Tokens are the small blocks. With the same blocks, you can build a bridge, a spiral, a tower, or something strange that only makes sense in one specific situation.

![Language shapes are memory shapes](https://zc277584121.github.io/images/understand-agent-memory/language-memory-shapes.png)

That is why Markdown as a memory substrate feels so natural. It gives language a simple, inspectable container.

This does not mean structured memory is useless. I see at least three cases where structure helps:

1. **Domain-specific workflows.** Sales records, support tickets, medical events, and legal facts often benefit from explicit fields and labels.
2. **Human-facing presentation.** Tables, timelines, and dashboards can make memory easier to inspect.
3. **General extraction layers.** Triples, graphs, and entity relations can be useful derived views, especially in domains with complex dependencies.

So I do not think the answer is "Markdown only" or "database only." The better framing is:

Markdown is a good source of truth for memory content. Structured systems are often useful indexes, projections, or views.

## Layer 4: Memory and Skills

The next layer is where things get more interesting.

If skills are procedural memory, then the agent memory problem is not only about recalling facts or conversations. It is also about how an agent learns reusable ways of doing work.

Current agent systems have mostly solved skill discovery and skill use.

A skill is usually described by a small metadata block: name, description, trigger conditions, and instructions. The agent harness can use progressive disclosure: first read the metadata, then load the full skill only when it seems relevant.

That means the lookup path is mostly there.

The missing part is skill writing and skill evolution.

![Skill discovery is solved, but creation and evolution still need a mechanism](https://zc277584121.github.io/images/understand-agent-memory/skill-discovery-gap.png)

Creating a good skill by hand is still clunky. You have to explain a lot of background, describe when it should trigger, list files to inspect, define verification steps, and then test it repeatedly.

![Manually describing a skill can require a long prompt](https://zc277584121.github.io/images/understand-agent-memory/skill-creator.png)

Skill evolution is also hard. Some approaches try to create evaluation datasets, run skill variants, score them, and select the better version. That can work, but it is heavy. You need datasets, environments, tasks, and evaluation signals.

There is another path:

**Use the agent's existing memory to create and improve skills.**

If an agent has access to your task history, it can look for repeated patterns. Maybe you keep releasing the same package. Maybe you keep debugging the same service. Maybe you keep asking for the same publishing workflow.

Those repeated operations are not just history. They are raw material for procedural memory.

This is the idea behind MemSearch's `memory-to-skill` workflow.

### How MemSearch Turns Memory into Skills

The design has two steps.

First, the system distills candidate skills from memory. It scans memory journals, finds repeated reusable workflows, and writes candidate skills into a separate candidate directory.

The candidate can keep evolving as more related memories arrive. If your release process changes, the candidate skill can be revised before it is ever installed.

Second, a human reviews and installs the skill.

![Reviewing and installing a distilled skill](https://zc277584121.github.io/images/understand-agent-memory/memsearch-review-standard.gif)

This boundary matters. A bad note is usually just noise. A bad installed skill is worse: it can actively trigger at the wrong time and do the wrong thing.

So the system proposes, but the human approves.

Here is a real example from my own workflow. While developing MemSearch, I repeatedly went through the same release process. Eventually, the system recognized that this was reusable and distilled a release skill candidate.

![A distilled release skill candidate](https://zc277584121.github.io/images/understand-agent-memory/memsearch-candidate-skill.png)

Once installed, I no longer needed to explain the release process from scratch. I could invoke the skill directly.

That is the moment where memory and skills form a loop:

Daily agent use creates memory. Memory distills into skills. Skills improve future agent use. Future use produces better memory.

![The memory-to-skill loop](https://zc277584121.github.io/images/understand-agent-memory/memory-to-skill-loop.png)

This is a small form of bootstrapping. The agent starts turning what happened before into reusable capability.

## Layer 4.5: Memory Needs a Context Harness

After thinking through these four layers, I started to feel that memory itself is only one part of a larger context problem.

Conversation history is memory. Skills are memory. Knowledge bases are memory.

But what about repositories, design docs, Slack messages, issue trackers, tickets, meeting notes, and project specs?

They are also part of the working memory of a team. They are just scattered across many systems, formats, and access patterns.

For any real task, the hard part is rarely "does the information exist?" The hard part is:

**Which tiny slice of all available context matters right now?**

That is why I think agent systems need a context harness above memory: a unified layer that connects scattered sources and lets the agent search and browse them through one interface.

This is the problem we are exploring with [MFS](https://github.com/zilliztech/mfs).

![MFS architecture](https://zc277584121.github.io/images/understand-agent-memory/mfs-architecture.png)

MFS represents each source as a file-like tree mounted at a stable address. Then agents can use skills to connect new sources, search across them, and inspect the relevant pieces.

For example, suppose you want to understand:

> What happens when our webhook retry fails?

The answer may be split across three places:

- the code repository, where the retry logic lives
- an old Slack thread, where someone explained the design trade-off
- your personal memory, where you recorded a previous incident

Without a context harness, you search these systems one by one. With one, the agent can search across code, messages, and memory in the same task flow.

At that point, memory stops being a standalone feature. It becomes one layer inside a broader context architecture.

## Final Thoughts

Agent memory is still evolving. There may be a fifth layer, a sixth layer, and a better vocabulary than the one I am using here.

But the direction feels clear to me.

Memory is not just a chat log. It is a thread that connects conversations, knowledge, skills, context, and the agent's own ability to improve over time.

And the foundation is surprisingly simple:

Write down what happened. Write down what is true. Write down how to do the thing next time.

The rest is indexing, retrieval, review, and iteration.

## References

1. CoALA paper, [Cognitive Architectures for Language Agents](https://arxiv.org/abs/2309.02427)
2. LangChain, [Memory for Agents](https://www.langchain.com/blog/memory-for-agents)
3. [MemSearch](https://github.com/zilliztech/memsearch)
4. [MFS](https://github.com/zilliztech/mfs)
