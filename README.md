# AgentEval

A unified Python framework for evaluating LLM coding agents via Docker containers.

## Overview

AgentEval provides a consistent interface for calling various LLM coding agents (nanobot, hermes, zeroclaw, openclaw, kilo_code, opencode, codex, claude_code, droid, grok_cli) through Docker containers. It enables reproducible benchmarking of agent capabilities across different LLMs and tasks.

## Features

- Unified Python API for multiple agent types
- Docker-based deployment for consistent environments
- OpenRouter integration for access to 200+ models
- Configurable model selection per agent
- Secure API key management (no hardcoded keys)

## Installation

```bash
# Clone the repository
git clone https://github.com/your-org/agent-eval.git
cd agent-eval

# Install the package
pip install -e .

# Start agent containers
cd containers && docker-compose up -d
```

## Quick Start

```python
from agent_eval import get_caller, DEFAULT_MODEL

# Get a caller for any agent
caller = get_caller('nanobot')

# Call the agent with a task
response = caller.call({
    'task_id': 'example-001',
    'problem_statement': 'What is 2+2? Answer in one number.'
}, timeout=60)

print(f"Success: {response.success}")
print(f"Output: {response.output}")
print(f"Duration: {response.duration:.2f}s")
```

## Supported Agents

| Agent | Description | Default Model |
|-------|-------------|---------------|
| nanobot | Nanobot AI agent | openrouter/free |
| hermes | Hermes self-evolving agent | openrouter/free |
| zeroclaw | ZeroClaw Rust agent | openrouter/free |
| openclaw | OpenClaw coding assistant | openrouter/auto |
| kilo_code | Kilo Code CLI | kilo/openrouter/free |
| opencode | OpenCode by SST | opencode/big-pickle |
| codex | OpenAI Codex CLI | openrouter/free |
| claude_code | Claude Code | anthropic/claude-sonnet-4.6 |
| droid | Factory AI Droid | openrouter/free |
| grok_cli | Grok CLI | openrouter/free |

## API Reference

### get_caller(agent_type: str) -> AgentCaller

Returns a caller instance for the specified agent type.

```python
caller = get_caller('nanobot')
```

### AgentCaller.call(task_input, timeout=300, model=None) -> AgentResponse

Calls the agent with the specified task.

Parameters:
- `task_input` (dict): Task specification containing:
  - `problem_statement` (str): The task to execute
  - `task_id` (str, optional): Identifier for the task
  - `repo` (str, optional): Repository path for code tasks
  - `test_patch` (str, optional): Test code to apply
- `timeout` (int): Timeout in seconds (default: 300)
- `model` (str): Model to use (default: agent-specific default)

Returns:
- `AgentResponse` containing:
  - `success` (bool): Whether the call succeeded
  - `output` (str): The agent's output
  - `error` (str): Error message if failed
  - `duration` (float): Time taken in seconds
  - `task_id` (str): The task identifier

### Example: Benchmarking Multiple Agents

```python
from agent_eval import get_caller

agents = ['nanobot', 'hermes', 'zeroclaw']
task = {'problem_statement': 'Fix the bug in this function'}

results = {}
for agent in agents:
    caller = get_caller(agent)
    response = caller.call(task, timeout=120)
    results[agent] = {
        'success': response.success,
        'output': response.output,
        'duration': response.duration
    }
```

## API Key Management

API keys are read from a `privacy_secret_openrouter_API_key.txt` file in the project root. This file should contain your OpenRouter API key (format: `sk-or-v1-...`).

The file path can be customized by setting the `AGENT_EVAL_PRIVACY_DIR` environment variable:

```bash
export AGENT_EVAL_PRIVACY_DIR=/path/to/your/privacy/files
```

## Docker Containers

All agent containers can be managed via the provided script:

```bash
# Start all containers
./containers/agent_containers.sh start

# Check status
./containers/agent_containers.sh status

# Open shell in a container
./containers/agent_containers.sh exec nanobot

# Stop all containers
./containers/agent_containers.sh stop
```

## Model Selection

Each agent has a default model optimized for general use. You can override this:

```python
# Use a specific model
caller = get_caller('nanobot')
response = caller.call(task, model='openrouter/auto')

# List of recommended free models
from agent_eval.models import OPENROUTER_FREE_MODELS
print(OPENROUTER_FREE_MODELS)
```

## Security

- API keys are never hardcoded in source files
- Keys are read from a privacy file at runtime
- The privacy file is gitignored
- For production, use Docker Secrets or Kubernetes Secrets

See `docs/DOCKER_SECRETS.md` for detailed security best practices.

## Project Structure

```
agent-eval/
├── src/agent_eval/           # Python package
│   ├── __init__.py          # Package entry point
│   ├── callers.py           # Agent caller abstraction
│   ├── api_keys.py           # API key management
│   └── models.py             # Model registry
├── containers/                # Docker configurations
│   ├── docker-compose.yml    # Container orchestration
│   └── agent_containers.sh   # Container management script
├── examples/                  # Usage examples
├── tests/                    # Unit tests
└── docs/                    # Documentation
```

## Contributing

Contributions are welcome. Please ensure:
- All tests pass before submitting PRs
- New agents follow the AgentCaller interface
- Documentation is updated for any API changes

## License

MIT License

## Acknowledgments

This framework was developed as part of the Mobius Injection research project for evaluating LLM coding agents.
