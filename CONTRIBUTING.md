# Contributing to Postmortem

Thank you for your interest in contributing!

## Development Setup

```bash
git clone <repo>
cd postmortem
python -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"
```

## Running Tests

```bash
pytest tests/ -v
```

## Pull Request Process

1. Fork the repo and create a branch from `main`.
2. Add tests for any new functionality.
3. Ensure all tests pass.
4. Submit a PR with a clear description of changes.

## Code Style

- Follow PEP 8.
- Use type hints.
- Write docstrings for public APIs.

## Reporting Issues

Open an issue with:
- Steps to reproduce
- Expected vs actual behavior
- Trace file if applicable (sanitized)
