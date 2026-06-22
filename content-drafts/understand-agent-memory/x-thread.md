1/ Agent memory is usually treated as "chat history."

I think that is too narrow.

The useful framing is 4 layers: history, knowledge, skills, and the context harness that decides which piece matters right now.

---

2/ The old split was short-term vs long-term memory.

Context window vs retrieved past conversations.

Useful, but incomplete. It misses that agents also need facts they learned and procedures they can repeat.

---

3/ CoALA's taxonomy aged surprisingly well:

Episodic memory = past interactions
Semantic memory = knowledge / RAG
Procedural memory = skills and workflows

Once I saw skills as memory, a lot clicked.

---

4/ This is why I like Markdown as a memory source of truth.

Real memory is messy: conditions, exceptions, preferences, tone.

Natural language handles those shapes better than a rigid schema.

---

5/ Structured memory still matters.

Tables, graphs, triples, and metadata are useful as views or indexes.

But I do not want them to be the canonical representation of every memory.

---

6/ The open problem for me is memory-to-skill:

Can repeated work patterns become reusable skills?

In MemSearch, I split this into 2 steps: distill candidates from memory, then let a human review and install.

---

7/ The bigger picture: memory is one layer of context.

Repos, docs, Slack, issues, and personal history all matter.

Full writeup:
https://zc277584121.github.io/ai-agent/2026/06/22/understand-agent-memory.html
