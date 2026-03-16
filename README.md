# Lang_Graph_Tutorial

This project is a small, practical introduction to LangGraph using a ReAct-style agent.

## About LangGraph

LangGraph is a framework for building stateful, multi-step AI workflows as graphs.
Instead of writing one long chain, you define nodes (steps) and edges (transitions) that control how the agent reasons, uses tools, and loops until completion.

In this tutorial, LangGraph is used to:

- Keep conversation state in `MessagesState`
- Run an LLM reasoning step
- Call tools when the model requests them
- Loop between reasoning and acting until no more tool calls are needed

## How This Repo Uses LangGraph

The graph in `main.py` has two nodes:

- `agent_reason`: runs model reasoning
- `act`: executes requested tools via `ToolNode`

Flow logic:

1. Start at `agent_reason`
2. If the latest response contains tool calls, route to `act`
3. Return to `agent_reason` after tool execution
4. End when no tool calls remain

This is a clean example of the ReAct loop implemented with LangGraph control flow.

## Core Concepts: Nodes, Edges, and State

LangGraph applications are built from three core building blocks.

### 1. Nodes

A node is a step in the workflow that performs work and returns updates to state.

In this project:

- `agent_reason` node runs `run_agent_reasoning` from `react.py`
- `act` node runs tools through `ToolNode(tools)` in `react.py`

You can think of nodes as small, focused functions in the agent loop:

- Reason node: decide what to do next
- Act node: execute tool calls

### 2. Edges

An edge defines how execution moves between nodes.

In `main.py`:

- A conditional edge from `agent_reason` decides between `act` and `END`
- A normal edge from `act` returns execution to `agent_reason`

This creates the ReAct cycle:

1. Reason
2. Act (if needed)
3. Reason again
4. Stop when no tool calls remain

### 3. State

State is the shared data that flows through the graph.

This tutorial uses `MessagesState`, which holds the chat message history.

- Each node reads from `state["messages"]`
- Nodes append new messages (model responses or tool results)
- `should_continue` checks the latest message to route the next edge

So state is what gives continuity to the workflow. Without state, each node would be isolated.

### Why This Pattern Works

- Clear separation: reasoning and tool execution are independent
- Easy to extend: add memory, validation, or guardrail nodes
- Easy to debug: transitions are explicit and visualized in `flow.png`

## Legacy LangChain Executor vs LangGraph

Both can run LLM + tools workflows, but they are designed for different levels of control.

| Topic | Legacy LangChain Executor | LangGraph |
|---|---|---|
| Control flow | Mostly predefined loop behavior | Explicit graph control with nodes and edges |
| State handling | Hidden or implicit in executor internals | First-class state object (like `MessagesState`) |
| Branching | Limited and harder to customize | Native conditional routing |
| Multi-step workflows | Works for simple agent loops | Better for complex, long-running workflows |
| Debuggability | Harder to inspect exact transitions | Easy to reason about transitions and graph paths |
| Extensibility | Add tools/prompts, but loop internals are less flexible | Add custom nodes (memory, validation, guardrails, human approval) |
| Production readiness | Good for quick prototypes | Better for deterministic and maintainable agent systems |

### Quick Rule of Thumb

- Use Legacy LangChain Executor when you want a fast, simple agent with minimal orchestration.
- Use LangGraph when you need explicit routing, durable state, and workflow-level control.

For this tutorial, LangGraph is the better fit because the ReAct loop is modeled directly as:

`agent_reason -> act -> agent_reason -> ... -> END`

## Files

- `main.py`: graph definition, routing logic, and app execution
- `node.py`: tool definitions and LLM binding
- `react.py`: reasoning node and `ToolNode` setup

## Run

Make sure your environment variables are set (for OpenAI and Tavily), then run:

```bash
python main.py
```

The script also exports a graph image to `flow.png`.

## Evolution of ReAct Agents

ReAct agents have evolved from simple prompt loops to structured workflow systems.

### Phase 1: Prompt-Only Reasoning

- The model reasons in plain text without external tools.
- Good for basic Q&A, but limited by model memory and knowledge cutoff.

### Phase 2: ReAct with Tool Calling

- The model alternates between reasoning and tool use.
- This enables web search, calculators, and custom functions.
- Most early implementations used fixed loop executors.

### Phase 3: Executor-Based Agents

- Framework executors made ReAct patterns easier to use.
- Fast to prototype, but harder to customize deeply.
- Complex branching, retries, and human-in-the-loop steps can become difficult.

### Phase 4: Graph-Based ReAct (LangGraph)

- ReAct becomes an explicit graph of nodes, edges, and state.
- Control flow is transparent and deterministic.
- Easy to add memory, guardrails, validation nodes, and approval steps.
- Better suited for production AI systems.

### Phase 5: Durable and Multi-Agent Workflows

- Modern systems combine multiple specialized agents.
- Workflows support persistence, resumability, and richer orchestration.
- ReAct remains the core pattern, but now inside robust graph architectures.

### Key Takeaway

ReAct is still the foundation, but the implementation has matured from "single loop" agents to "stateful graph" systems. This tutorial reflects that shift by implementing ReAct with LangGraph.