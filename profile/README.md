# OpenSymbolicAI

**Make AI a software engineering discipline.**

Tool-calling agents cost [2.3x more](https://www.opensymbolic.ai/blog/illustration-part-3-cost-and-reliability), [fail ~20% of the time](https://www.opensymbolic.ai/blog/illustration-part-3-cost-and-reliability), and [ignore your instructions](https://www.opensymbolic.ai/blog/illustration-part-1-the-attention-loss-problem) as context grows. Every ReAct loop re-tokenizes the entire conversation, burning tokens and losing accuracy with each step.

OpenSymbolicAI fixes this by splitting the LLM's job in two:

```
┌─────────────────────────────────────┐
│  Traditional Agent (ReAct)          │
│                                     │
│  User ─→ LLM ─→ Tool ─→ LLM ─→      │
│          Tool ─→ LLM ─→ Tool ─→     │
│          LLM ─→ ... (loop forever)  │
│                                     │
│  ⚠ Data in prompt = injection risk  │
│  ⚠ Context bloats every iteration   │
│  ⚠ LLM makes unplanned tool calls   │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│  OpenSymbolicAI (Plan + Execute)    │
│                                     │
│  User ─→ LLM ─→ Plan                │
│                    ↓                │
│          Runtime executes plan      │
│          deterministically          │
│                                     │
│  ✓ Data never enters LLM context    │
│  ✓ Fewer tokens, fewer LLM calls    │
│  ✓ Every side effect is explicit    │
└─────────────────────────────────────┘
```

The LLM plans. The runtime executes. Data stays in application memory and never gets tokenized.

**Three blueprints** for different problem shapes:

| Blueprint | Pattern | Use when |
|-----------|---------|----------|
| **PlanExecute** | Plan once, execute deterministically | Fixed sequence of steps (calculators, converters, simple QA) |
| **DesignExecute** | Plan with loops and conditionals | Dynamic-length data (shopping carts, batch processing) |
| **GoalSeeking** | Plan → execute → evaluate → repeat | Iterative problems (optimization, multi-hop research, deep research) |

```bash
pip install opensymbolicai-core
```

## How It Works

Define **primitives** (what your agent can do) and **decompositions** (examples of how to use them). The LLM learns from your examples to plan new queries:

```python
from opensymbolicai import PlanExecute, primitive, decomposition

class Calculator(PlanExecute):

    @primitive(read_only=True)
    def add(self, a: float, b: float) -> float:
        return a + b

    @decomposition(
        intent="What is 2 + 3?",
        expanded_intent="Add the two numbers",
    )
    def _example(self) -> float:
        return self.add(a=2, b=3)
```

Every decomposition you add makes the agent better. This is [the flywheel](https://www.opensymbolic.ai/blog/the-missing-flywheel-in-agent-building) that prompt engineering doesn't have.

## Why This Matters

| Problem | How OpenSymbolicAI solves it |
|---|---|
| **Prompt injection** | [Symbolic Firewall](https://www.opensymbolic.ai/blog/security-by-design) keeps data out of LLM context. Nothing to inject into |
| **Unpredictable behavior** | Execution is deterministic and fully traced. Even iterative agents (GoalSeeking) produce inspectable plans each step — no runaway tool-calling |
| **High costs** | Fewer LLM calls to plan, then pure code execution. No re-tokenizing on every step |
| **Can't test or debug** | Full execution traces, typed outputs (Pydantic), version-controlled behavior |
| **Model lock-in** | Model-agnostic. Swap providers without rewriting your agent |

## Repositories

| Repo | Description |
|------|-------------|
| [core-py](https://github.com/OpenSymbolicAI/core-py) | Python runtime: primitives, blueprints (PlanExecute, DesignExecute, GoalSeeking), multi-provider LLM abstraction |
| [examples-py](https://github.com/OpenSymbolicAI/examples-py) | Example agents: RAG, multi-hop QA, deep research, unit converter, date calculator |
| [cli-py](https://github.com/OpenSymbolicAI/cli-py) | Interactive TUI for discovering and running agents |
| [claude-skills](https://github.com/OpenSymbolicAI/claude-skills) | Claude Code skills for scaffolding agents, adding primitives/decompositions/evaluators, and debugging traces |

## Benchmarks

| Benchmark | Result | What it shows |
|-----------|--------|---------------|
| [FOLIO](https://github.com/OpenSymbolicAI/benchmark-py-folio) | **89.2%** — outperforms GPT-4 CoT (78.1%) | PlanExecute + Z3 theorem prover. `gpt-oss-120b` (Fireworks) translates to first-order logic, Z3 proves it |
| [MultiHopRAG](https://github.com/OpenSymbolicAI/benchmark-py-MultiHopRAG) | **82.9%** — +7.9pp over previous best | GoalSeeking iterates over 609 documents, 2,556 queries. `gpt-oss-120b` (Fireworks) |

## Deep Dives

- [Getting Started](https://www.opensymbolic.ai/blog/getting-started-with-opensymbolicai) - Build your first agent in 5 minutes
- [The OpenSymbolicAI Manifesto](https://www.opensymbolic.ai/blog/the-opensymbolicai-manifesto) - The philosophy behind the architecture
- [Behaviour Programming vs. Tool Calling](https://www.opensymbolic.ai/blog/behaviour-programming-vs-tool-calling) - Why executable examples beat massive prompts
- [LLM Attention Is Precious: Why ReAct Wastes It](https://www.opensymbolic.ai/blog/llm-attention-is-precious) - A visual breakdown of token waste
- [Secure by Design](https://www.opensymbolic.ai/blog/security-by-design) - How the Symbolic Firewall prevents prompt injection
- [The Missing Flywheel in Agent Building](https://www.opensymbolic.ai/blog/the-missing-flywheel-in-agent-building) - Why agents stay brittle and how to fix it

## License

MIT
