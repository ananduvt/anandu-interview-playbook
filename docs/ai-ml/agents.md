# AI Agent Building Blocks

An **AI agent** is a system where an LLM is given autonomy to decide, call tools and act, instead of only returning text. It runs a loop: observe, plan, act, observe the result, repeat until the goal is met.

## Core Concepts

| Concept | Meaning | Analogy |
|---------|---------|---------|
| **Agent** | LLM-driven loop that pursues a goal | The employee |
| **Instructions** | System prompt: role, rules, tone, boundaries | Employee handbook |
| **Skills** | Packaged capabilities (guidance plus resources) the agent loads when a request needs them, e.g. research or document generation | Role / expertise |
| **Tools** | Individual callable functions: web search, code interpreter, DB query, API call | Software access |
| **MCP server** | Server exposing tools, resources and prompts over the open **Model Context Protocol**, a standard way to connect AI apps to external data and systems | Standard IT integration layer |

Tools are single actions; skills bundle know-how for a kind of task; MCP is the standard that lets any compatible agent use external tools without custom glue per integration.

## Workflow Example

Request: "Find today's top tech news and compile a PDF summary."

1. The agent reads its **instructions** (format, tone, constraints).
2. It decides which **skills** apply (research, document generation).
3. It calls **tools**: a search API, then a code executor to build the PDF.
4. If it must save to a workspace or corporate system, it calls an **MCP server** that controls access to that resource.
5. It checks the result and returns it, or loops again on failure.

## Interview Tips

- Agent vs plain LLM call: an agent chooses actions and iterates; a plain call is one prompt, one response.
- Guardrails: least-privilege tools, human approval for destructive actions, step/iteration limits, logging of every tool call.
- Risks: prompt injection through tool outputs, runaway loops and cost, hallucinated tool arguments (validate inputs).

**See also:** [LLMs & Transformers](llms-transformers.md), [RAG & Generative AI](rag-generative.md), [AI in Java](ai-java.md)
