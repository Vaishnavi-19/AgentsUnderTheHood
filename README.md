# Agents Under The Hood

This repository explains how **Reasoning and Acting (ReAct) Agents** work internally.
ReAct combines:

- **Reasoning**: The model thinks about what to do next.
- **Acting**: The model calls tools (search, calculator, code runner, APIs, etc.) to get missing information.

The agent repeats this pattern in loops until it has enough evidence to produce a final answer.

## What Is a ReAct Agent?

A ReAct agent is a control loop around an LLM:

1. Understand the user goal.
2. Think step-by-step (internal reasoning).
3. Decide whether to call a tool.
4. Observe the tool output.
5. Update the plan.
6. Repeat until ready to answer.

This improves reliability because the model does not guess when it can verify with tools.

## Architecture Diagram

```mermaid
flowchart TD
	U[User Request] --> O[Orchestrator / Agent Runtime]
	O --> C[LLM Core]

	C --> R[Reasoning State]
	R --> D{Need external info?}

	D -- Yes --> A[Action Planner]
	A --> T[Tool Call]
	T --> OBS[Observation / Tool Result]
	OBS --> M[Memory + Context Update]
	M --> C

	D -- No --> F[Answer Composer]
	F --> V[Validation / Safety Check]
	V --> OUT[Final Response]
	OUT --> U
```

## Diagram Explanation

- `User Request`: The problem statement from the user.
- `Orchestrator / Agent Runtime`: Controls the loop and decides when to continue or stop.
- `LLM Core`: Generates thoughts, actions, and responses.
- `Reasoning State`: Tracks current hypothesis, missing facts, and next step.
- `Need external info?`: Decision gate. If knowledge is missing, use tools.
- `Action Planner`: Chooses the best tool and input.
- `Tool Call`: Executes API/query/code/tool action.
- `Observation / Tool Result`: Captures returned data.
- `Memory + Context Update`: Adds trusted observations back into context.
- `Answer Composer`: Builds the final user-facing answer from verified info.
- `Validation / Safety Check`: Ensures quality, policy, and format correctness.

## ReAct Loops

### Loop1: Reasoning Loop (Think -> Decide)

Purpose: decide what is missing and whether action is required.

Typical steps:

1. Parse intent and constraints.
2. Generate a short plan.
3. Identify unknowns.
4. Decide: answer now or gather data.

Output of Loop1:

- Either a direct answer draft, or
- A structured action request for Loop2.

### Loop2: Acting Loop (Act -> Observe -> Update)

Purpose: resolve unknowns via tools.

Typical steps:

1. Select tool (`search`, `db`, `calculator`, `code`, etc.).
2. Execute with precise inputs.
3. Capture observation.
4. Evaluate if observation is enough.
5. If not enough, act again.
6. Send updated context back to Loop1.

Key idea: each action should reduce uncertainty.

### Answer Loop (Anwer Loop): Compose -> Verify -> Deliver

Purpose: produce the final response once confidence is high.

Typical steps:

1. Compose answer using verified observations.
2. Check correctness, clarity, and safety.
3. Ensure response matches user format/style needs.
4. Return final answer.

> Note: "Anwer loop" is commonly intended as "Answer loop".

## End-to-End Flow Summary

1. User asks a question.
2. **Loop1** reasons about what is needed.
3. If needed, **Loop2** gathers evidence using tools.
4. Agent cycles between Loop1 and Loop2 until ready.
5. **Answer loop** formats and validates the final response.

## Why ReAct Works Well

- Reduces hallucination by grounding answers in tool output.
- Improves traceability (you can inspect steps).
- Handles complex tasks through iterative decomposition.
- Supports dynamic planning when new information appears.

## Minimal Pseudocode

```text
state = init(user_query)

while not state.ready_to_answer:
	thought = reason(state)                  # Loop1
	if thought.requires_action:
		obs = act_and_observe(thought)       # Loop2
		state = update(state, obs)
	else:
		state.ready_to_answer = True

final_answer = compose_and_validate(state)   # Answer loop
return final_answer
```

## uv Quickstart Commands

`uv` is a fast Python package and project manager. Use it to initialize projects, manage dependencies, and run code.

### Project setup

```bash
# Initialize a new Python project in the current directory
uv init

# Initialize with a specific project name
uv init my_project
```

### Dependency management

```bash
# Add a runtime dependency
uv add requests

# Add multiple dependencies
uv add fastapi pydantic

# Add a development dependency
uv add --dev pytest ruff

# Remove a dependency
uv remove requests
```

### Run and execute

```bash
# Run a Python file using the project environment
uv run main.py

# Run a module
uv run -m pytest

# Run a command-line tool from dependencies
uv run ruff check .
```

### Environment and sync

```bash
# Create/update environment from lockfile
uv sync

# Rebuild lockfile from pyproject.toml
uv lock

# Show dependency tree
uv tree
```

### Python version management

```bash
# Pin a Python version for the project
uv python pin 3.12

# Install a Python version
uv python install 3.12
```

### Useful workflow

```bash
uv init
uv add openai
uv add --dev pytest
uv run python -c "print('ReAct agent project ready')"
```