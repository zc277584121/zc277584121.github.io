Agent memory is usually treated as "chat history."

I think that is too narrow.

After spending the past few months building memory tooling for agents, I now think about memory in four layers:

1. Episodic memory: what happened before, including conversations and task history.
2. Semantic memory: facts and knowledge, usually backed by RAG or a knowledge base.
3. Procedural memory: skills, workflows, and repeatable ways of doing work.
4. Context harness: the layer above memory that connects repos, docs, Slack, issues, and other scattered sources.

The third one changed how I think about the whole problem.

If skills are procedural memory, then agent memory is not just about recalling the past. It is also about turning repeated work into reusable capability.

That is the idea behind the memory-to-skill workflow I have been building in MemSearch: distill candidate skills from repeated task history, then let a human review and install them.

The human review step matters. A bad memory is usually just noise. A bad installed skill can actively do the wrong thing.

My current takeaway: memory should probably be written in natural language, kept inspectable, and indexed or structured only as a derived layer.

Curious how others are thinking about this: should agent memory be mostly Markdown, database rows, graphs, or something else?

#AIAgents #RAG #DeveloperTools
