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
# Lang_Graph_Tutorial-