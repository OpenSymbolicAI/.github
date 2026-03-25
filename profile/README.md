<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/OpenSymbolicAI/.github/main/profile/opensymbolicai-horizontal-dark.svg">
    <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/OpenSymbolicAI/.github/main/profile/opensymbolicai-horizontal.svg">
    <img alt="OpenSymbolicAI" src="https://raw.githubusercontent.com/OpenSymbolicAI/.github/main/profile/opensymbolicai-horizontal.svg" height="48">
  </picture>
</p>

**Make AI a software engineering discipline.**

On the [TravelPlanner benchmark](https://www.opensymbolic.ai/blog/travelplanner-benchmark) (ICML 2024), LangChain passes 77.8% of tasks and CrewAI passes 73.3%. They burn [3–6× more tokens](https://www.opensymbolic.ai/blog/illustration-part-2-token-economics), cost [4–8× more](https://www.opensymbolic.ai/blog/illustration-part-3-cost-and-reliability) per passing result, and [lose track of instructions](https://www.opensymbolic.ai/blog/illustration-part-1-the-attention-loss-problem) as context grows. GPT-4 alone scores 0.6%.

OpenSymbolicAI scores **97.9%** on 1,000 tasks by splitting the LLM's job in two:

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

### Runtimes

| Language | Repo | Description |
|----------|------|-------------|
| Python | [core-py](https://github.com/OpenSymbolicAI/core-py) | Primitives, blueprints (PlanExecute, DesignExecute, GoalSeeking), multi-provider LLM abstraction |
| TypeScript | [core-ts](https://github.com/OpenSymbolicAI/core-ts) | TypeScript core SDK |
| Go | [core-go](https://github.com/OpenSymbolicAI/core-go) | Go runtime with AST-based plan execution |
| C# / .NET | [core-dotnet](https://github.com/OpenSymbolicAI/core-dotnet) | .NET runtime |

### Examples & Tools

| Repo | Description |
|------|-------------|
| [examples-py](https://github.com/OpenSymbolicAI/examples-py) | Python examples: RAG, multi-hop QA, deep research, unit converter, date calculator |
| [examples-ts](https://github.com/OpenSymbolicAI/examples-ts) | TypeScript examples: RAG Agent, Date Agent, Unit Converter |
| [cli-py](https://github.com/OpenSymbolicAI/cli-py) | Interactive TUI for discovering and running agents |
| [claude-skills](https://github.com/OpenSymbolicAI/claude-skills) | Claude Code skills for scaffolding agents, adding primitives/decompositions/evaluators, and debugging traces |

## Benchmarks

| Benchmark | Result | What it shows |
|-----------|--------|---------------|
| [TravelPlanner](https://github.com/OpenSymbolicAI/benchmark-py-TravelPlanner) | **97.9%** on 1,000 tasks — GPT-4 gets 0.6% | GoalSeeking two-stage. 100% hard constraint pass rate, 3.1× fewer tokens than LangChain. [Blog post](https://www.opensymbolic.ai/blog/travelplanner-benchmark) |
| [MultiHopRAG](https://github.com/OpenSymbolicAI/benchmark-py-MultiHopRAG) | **82.9%** — +7.9pp over previous best | GoalSeeking, 609 documents, 2,556 queries. Same result in [Python](https://github.com/OpenSymbolicAI/benchmark-py-MultiHopRAG), [C#](https://github.com/OpenSymbolicAI/benchmark-cs-MultiHopRAG) (83.8%), and [Go](https://github.com/OpenSymbolicAI/benchmark-go-MultiHopRAG) (81.6%). [Blog post](https://www.opensymbolic.ai/blog/multihop-rag-cross-language) |
| [LegalBench](https://github.com/OpenSymbolicAI/benchmark-py-legalbench) | **93.1%** across 162 legal reasoning tasks | GoalSeeking agent. 835 items, 0 errors, $1.88 total cost |
| [FOLIO](https://github.com/OpenSymbolicAI/benchmark-py-folio) | **89.2%** — outperforms GPT-4 CoT (78.1%) | PlanExecute + Z3 theorem prover. First-order logic reasoning |

### Framework Comparison (TravelPlanner)

Same model (`gpt-oss-120b`), same tools, same evaluation — only the framework differs:

```
                Pass Rate        Tokens/Task       Cost/Passing Task    LLM Calls/Task
                ─────────        ───────────       ─────────────────    ──────────────
OpenSymbolicAI  ████████████ 100%  ██░░░░░░░  13,936   █░░░░░░░  $0.013    ██░░░░░░░  2.3
LangChain       █████████░░░ 77.8% █████░░░░  43,801   ████░░░░  $0.051    ████████░  13.5
CrewAI          ████████░░░░ 73.3% █████████  81,331   ████████  $0.100    █████████  39.6
```

7 models hit 100% pass rate — including **Llama 3.3 70B at $0.006/task and 4.3s latency** on Groq. The framework matters more than the model. See the [full model landscape](https://github.com/OpenSymbolicAI/benchmark-py-TravelPlanner/blob/main/MODEL-LANDSCAPE.md).

## Deep Dives

- [Getting Started](https://www.opensymbolic.ai/blog/getting-started-with-opensymbolicai) - Build your first agent in 5 minutes
- [The OpenSymbolicAI Manifesto](https://www.opensymbolic.ai/blog/the-opensymbolicai-manifesto) - The philosophy behind the architecture
- [The Anatomy of PlanExecute](https://www.opensymbolic.ai/blog/plan-execute-anatomy) - Why the core blueprint is what it is
- [Behaviour Programming vs. Tool Calling](https://www.opensymbolic.ai/blog/behaviour-programming-vs-tool-calling) - Why executable examples beat massive prompts
- [English, Spec, or Code](https://www.opensymbolic.ai/blog/the-prompt-spectrum) - How you talk to the LLM decides how far you get
- [LLM Attention Is Precious](https://www.opensymbolic.ai/blog/llm-attention-is-precious) - A visual breakdown of token waste
- [Secure by Design](https://www.opensymbolic.ai/blog/security-by-design) - How the Symbolic Firewall prevents prompt injection
- [The Missing Flywheel in Agent Building](https://www.opensymbolic.ai/blog/the-missing-flywheel-in-agent-building) - Why agents stay brittle and how to fix it
- [Closing the Flywheel in Practice](https://www.opensymbolic.ai/blog/closing-the-flywheel-in-practice) - Practical implementation of the flywheel
- [Agent-to-Agent Is Just Function Calls](https://www.opensymbolic.ai/blog/agent-to-agent) - Multi-agent systems need typed interfaces, not new infrastructure
- [Change Everything, Change Nothing](https://www.opensymbolic.ai/blog/multihop-rag-cross-language) - MultiHopRAG in Python and C# — accuracy moves by 0.9pp
- [Third Language, Same Result](https://www.opensymbolic.ai/blog/multihop-rag-go) - MultiHopRAG in Go — the framework holds across three languages

## License

MIT
