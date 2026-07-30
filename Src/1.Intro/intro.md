# Intro to LangChain & LangChain vs LangGraph

*Beginner-friendly notes with analogies*

---

## 1. What is LangChain?

LangChain is a framework that helps you build applications powered by large language models (LLMs) — like chatbots, document Q&A tools, or research assistants — without reinventing the wheel every time.

On its own, an LLM (like GPT) is just a "text-in, text-out" brain. It doesn't know how to search the web, read your files, remember earlier messages, or call other tools. LangChain provides the plumbing to connect an LLM to all of that.

> **Analogy:** Think of an LLM as a brilliant chef who's really good at cooking (generating text) but has no kitchen, no ingredients, and no memory of what the customer ordered five minutes ago. LangChain is the *restaurant* built around that chef — it brings in ingredients (your data), remembers orders (conversation memory), and passes dishes between stations (chaining steps together).

### What LangChain gives you
- **Prompt templates** – reusable, fill-in-the-blank prompts instead of retyping instructions every time.
- **Chains** – a way to link steps together (e.g., "take user input → search a database → summarize the result").
- **Memory** – lets a chatbot remember earlier parts of a conversation.
- **Document loaders & retrievers** – pull in your own data (PDFs, websites, notes) so the LLM can answer questions about it (this is the basis of RAG — Retrieval-Augmented Generation).
- **Tool/agent integration** – lets the LLM call external tools, like a calculator, a search engine, or an API.

### A tiny example (generic, not tied to any real project)

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

llm = ChatOpenAI(model="gpt-4o-mini")

prompt = ChatPromptTemplate.from_template(
    "Explain {topic} to a {audience} in 3 simple sentences."
)

chain = prompt | llm

response = chain.invoke({"topic": "black holes", "audience": "10-year-old"})
print(response.content)
```

Here, the prompt template is the "recipe card," the LLM is the "chef," and the `|` (pipe) connects them into one simple chain — ask a question, get a tailored answer.

---

## 2. Why Not Just Use the LLM Directly?

You *can* call an LLM directly for simple one-off questions. But real applications usually need:

- Multiple steps in sequence (search → summarize → answer)
- Access to outside data or tools
- Memory across a conversation
- Structured, repeatable prompts instead of hardcoded strings

LangChain packages all of this into reusable building blocks, so you're not writing the same plumbing code from scratch in every project.

---

## 3. What is LangGraph?

LangGraph is a separate framework (built by the same team behind LangChain) designed for building applications where the flow of steps isn't a straight line — it can branch, loop, or make decisions.

> **Analogy:** If a LangChain "chain" is like a factory assembly line — item goes in one end, comes out the other, one step after another — LangGraph is more like a flowchart or a board game. You might loop back a few steps, take a different path depending on a condition, or have several "players" (agents) working and checking in on each other.

### Why loops and branches matter
Many real AI agents don't just do Step 1 → Step 2 → Step 3 and stop. They need to:
- Check their own work and retry if it's wrong ("self-correction loop")
- Decide between multiple possible next actions based on the situation
- Have multiple specialized agents hand off tasks to each other

A simple chain struggles to express "go back and try again" or "if X, do this; otherwise, do that." LangGraph represents your application as a **graph** — nodes (steps) connected by edges (possible paths) — which naturally supports branching and looping.

### A tiny example (conceptual)

```python
from langgraph.graph import StateGraph, END

def generate_answer(state):
    state["answer"] = f"Draft answer for: {state['question']}"
    return state

def check_quality(state):
    state["is_good"] = "good" in state["answer"].lower() or len(state["answer"]) > 10
    return state

def route(state):
    return END if state["is_good"] else "generate_answer"

graph = StateGraph(dict)
graph.add_node("generate_answer", generate_answer)
graph.add_node("check_quality", check_quality)
graph.set_entry_point("generate_answer")
graph.add_edge("generate_answer", "check_quality")
graph.add_conditional_edges("check_quality", route)

app = graph.compile()
result = app.invoke({"question": "What is gravity?"})
print(result)
```

Notice the graph can loop back to `generate_answer` if the quality check fails — something a plain straight-line chain can't easily do.

---

## 4. LangChain vs LangGraph — Cheat Sheet

| | **LangChain** | **LangGraph** |
|---|---|---|
| **Best for** | Straightforward, mostly linear pipelines (prompt → LLM → output, RAG Q&A) | Complex, branching, or looping workflows (multi-step agents, self-correcting agents) |
| **Flow shape** | Mostly a straight line (a "chain") | A graph — nodes and edges, can branch and loop |
| **Analogy** | Assembly line | Flowchart / board game |
| **Memory & state** | Built-in conversation memory helpers | Explicit, structured state passed between nodes |
| **Decision-making** | Limited — usually needs an "agent" bolted on | Native — conditional edges route the flow |
| **Complexity to set up** | Simpler for basic use cases | More setup, but scales better for complex logic |
| **Relationship** | Often used *underneath* LangGraph for prompts, LLM calls, and retrieval | Built by the same team; can use LangChain components inside its nodes |

**In short:** LangChain gives you the individual building blocks (prompts, LLM calls, retrievers, tools). LangGraph gives you a way to *orchestrate* those blocks into workflows that can think, branch, retry, and loop — not just move forward in one direction.

---

## 5. One Mental Model to Keep

**LangChain = the ingredients and recipes. LangGraph = the kitchen workflow that decides which recipe to use next, whether to redo a dish that came out wrong, and how multiple cooks coordinate.**

If your app is a simple, predictable pipeline — reach for LangChain alone. If your app needs to make decisions, retry itself, or coordinate multiple agents — reach for LangGraph, using LangChain's building blocks inside it.