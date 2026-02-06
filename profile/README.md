# OpenSymbolicAI

**Make AI a software engineering discipline.**

## The Problem

LLMs are stochastic, change without notice, and drift as context grows. Most agent frameworks dump instructions and data together, then let the LLM loop freely - creating injection risks and unpredictable behavior.

## Our Approach

OpenSymbolicAI separates planning from execution:

1. **LLM plans** - Given a query and primitive signatures (not your data), the LLM outputs a deterministic execution plan
2. **Runtime executes** - The plan runs without the LLM in the loop. Data never gets tokenized. Side effects are explicit.

This gives you version control, testing, debugging, and reproducibility - the things software engineers expect.

## Repositories

| Repo | Description |
|------|-------------|
| [core-py](https://github.com/OpenSymbolicAI/core-py) | Python runtime - primitives, decompositions, and the PlanExecute blueprint |
| [cli-py](https://github.com/OpenSymbolicAI/cli-py) | Interactive TUI for discovering and running agents |

## Quick Example

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

The LLM learns from examples to plan new queries using your primitives.

## Learn More

- [Security by Design](https://www.opensymbolic.ai/blog/security-by-design) - How the Symbolic Firewall prevents prompt injection
- [Website](https://www.opensymbolic.ai)

## License

MIT
