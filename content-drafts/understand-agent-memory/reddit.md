Suggested subreddit: r/OpenClaw

Title: I think "agent memory" is too often used to mean only chat history

I have been working on agent memory tooling for a while, and I keep running into the same confusion: people say "memory", but they are often talking about completely different things.

Sometimes they mean chat history. Sometimes they mean a vector database. Sometimes they mean a Markdown file. Sometimes they mean a skill the agent can reuse later.

The framing that finally helped me was the CoALA taxonomy:

- Episodic memory: what happened before, like conversations and task history.
- Semantic memory: facts and knowledge, usually RAG or a knowledge base.
- Procedural memory: skills, workflows, and repeatable ways of doing work.

That last one is the part I think is underrated.

If skills are procedural memory, then memory is not only about recalling previous context. It is also about turning repeated work into reusable capability.

For example, I kept going through the same release workflow while building MemSearch. Eventually it became obvious that the release process was not just "history"; it was a candidate skill waiting to be extracted.

The tricky part is trust. I do not think a system should silently install skills just because it found a pattern. A bad memory is usually just noise, but a bad installed skill can actively trigger at the wrong time and do the wrong thing.

So the flow I like is:

1. Distill candidate skills from memory.
2. Keep them in a candidate directory where no agent loads them automatically.
3. Let a human review, edit, and install the useful ones.

I also think Markdown is a surprisingly good source of truth for memory. Real memory is full of conditions, exceptions, and fuzzy context. Natural language is often better at preserving that than a rigid schema. Structured data is still useful, but more as an index or view than the canonical memory itself.

Curious how others here are handling this. Do you keep agent memory in Markdown, a database, a graph, a vector store, or some combination?

Full writeup if useful: https://zc277584121.github.io/ai-agent/2026/06/22/understand-agent-memory.html
