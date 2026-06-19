# Postmortem — Evals From Prod Failures

> Your evals are green. Production is 56% reliable. The difference is the failures you never turned into tests.

**Postmortem** automatically turns real production traces of agent failures into reusable regression test cases — so the same failure never ships twice.

## The Loop

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│  Production  │      │   Cluster    │      │   Distill    │      │   Eval Case  │
│   Failure    │─────▶│  Similar     │─────▶│  Minimal     │─────▶│  + Assertion │
│   (trace)    │      │  Failures    │      │  Repro       │      │              │
└──────────────┘      └──────────────┘      └──────────────┘      └──────┬───────┘
                                                                         │
                                                                         ▼
┌──────────────┐      ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│   Ship it    │◀─────│  No          │◀─────│   Runner     │◀─────│  Regression  │
│  🚀         │      │  Regressions │      │  Replay      │      │    Gate       │
└──────────────┘      └──────────────┘      └──────────────┘      └──────────────┘
```

Prod failure → eval case → regression gate → ship with confidence.

## Why Benchmarks Lie

Your agent scores 92% on your eval suite. In production it fails on 44% of real requests. This isn't a mystery — it's a measurement gap. Benchmarks test what you imagined could go wrong. Production shows you what actually does.

Research from Berkeley's LMSYS shows that static benchmarks diverge wildly from real-world task completion. The fix isn't better benchmarks — it's evals grounded in production traces.

## Quickstart

```bash
pip install -e .

# 1. Ingest a production trace
postmortem add examples/traces/failing_trace.jsonl --label "tool-loop"

# 2. Build an eval suite from labeled traces
postmortem build --out examples/suite

# 3. Run the suite against your agent
postmortem run examples/suite --agent my_agent.py

# 4. Cluster unlabeled traces to find patterns
postmortem cluster examples/traces
```

## Python API

```python
from postmortem import from_trace, run_suite

# Single trace → eval case
case = from_trace("trace.jsonl")

# Run a full suite
report = run_suite("suite/", agent=my_agent)
assert report.no_regressions
```

## Trace Formats

### JSONL (one trace per file or multiple)

```json
{"id": "trace-1", "messages": [
  {"role": "user", "content": "List files in /tmp"},
  {"role": "assistant", "content": null, "tool_calls": [{"name": "list_files", "args": {"path": "/tmp"}}]},
  {"role": "tool", "content": "Error: permission denied"}
], "outcome": "error", "metadata": {"model": "gpt-4o", "tokens": 150}}
```

### Eval Case (YAML)

```yaml
id: case-trace-1
label: tool-loop-on-empty-result
input:
  messages:
    - role: user
      content: 'List files in /tmp'
assertions:
  - type: not_contains
    expected: 'Error: permission denied'
    description: 'Should not return permission error'
  - type: tool_called
    expected: 'list_files'
    description: 'Should call list_files tool'
```

## Architecture

| Component | Purpose |
|-----------|---------|
| **Ingest** | Parse traces from JSONL, OpenTelemetry, or LangChain |
| **Cluster** | Group similar failures with Jaccard similarity |
| **Distill** | Extract minimal reproducing input + assertion |
| **Case** | Portable eval-case format (JSON/YAML) |
| **Runner** | Replay suite against agent, track regressions |

## License

MIT
