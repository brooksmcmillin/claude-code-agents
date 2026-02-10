# Claude Code Agents

A collection of specialized agents for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that perform automated code quality, security, and maintenance analysis on your codebase.

## What Are Claude Code Agents?

Claude Code agents are reusable, specialized prompts that give Claude a defined role, toolset, and methodology for a specific task. Each agent is a Markdown file with YAML frontmatter that configures its name, model, available tools, and behavior. When invoked, the agent follows its instructions to analyze your code and produce a structured report.

Agents are useful because they encode expert-level analysis procedures into a repeatable format. Instead of writing ad-hoc prompts each time, you get consistent, thorough results.

## Agents in This Collection

| Agent | Description | Best Used |
|-------|-------------|-----------|
| **code-optimizer** | Analyzes code for complexity, duplication, dead code, and maintainability issues | After feature work, during refactoring, periodic health checks |
| **dependency-auditor** | Scans dependencies for CVEs, outdated packages, and license problems | Before releases, on a schedule, when adding dependencies |
| **doc-auditor** | Finds stale, inconsistent, duplicate, or missing documentation | After major refactors, before releases |
| **security-code-reviewer** | Reviews code for vulnerabilities, misconfigurations, and insecure patterns | After code changes, before commits/pushes |
| **test-coverage-checker** | Identifies untested code paths, especially in critical areas like auth and error handling | After new features, before refactoring |

All agents use the Claude Sonnet model and have access to **Read**, **Grep**, **Glob**, and **Bash** tools for analyzing your codebase.

## Getting Started

### Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed and authenticated
- A codebase you want to analyze

### Installation

Clone this repository (or copy the agent `.md` files) into a location on your machine:

```bash
git clone <repo-url> claude-code-agents
```

### Running an Agent

Use the `claude` CLI with the `--agent` flag, pointing to the agent file and the project you want to analyze:

```bash
# Run from your target project directory
claude --agent /path/to/claude-code-agents/security-code-reviewer.md

# Or specify a prompt to focus the analysis
claude --agent /path/to/claude-code-agents/code-optimizer.md "Focus on the src/api directory"
```

You can also run agents against a specific directory:

```bash
claude --agent /path/to/claude-code-agents/test-coverage-checker.md --cwd /path/to/your/project
```

### Running Agents from Inside Claude Code

You don't have to start a new session to use an agent. If you already have Claude Code running, you can ask it to spin up one or more agents mid-conversation. Claude Code will launch each agent as a sub-process with its own context, run the analysis, and return the results back to you.

Just tell Claude Code to run the agent and point it at the file:

```
> Run the security-code-reviewer agent from /path/to/claude-code-agents/security-code-reviewer.md against this project
```

You can also ask for multiple agents at once. Claude Code will run them in parallel and report back with each agent's findings:

```
> Run the security-code-reviewer and test-coverage-checker agents from /path/to/claude-code-agents/ on this repo
```

This is especially useful when you're already in the middle of a coding session and want a quick check without switching context. For example, after finishing a feature you might ask:

```
> I just finished the new payment flow. Run the security-code-reviewer and code-optimizer agents on src/payments/
```

Claude Code handles the orchestration -- it reads the agent file, spawns the agent with the right tools and model, and folds the results back into your conversation.

### Example Workflows

**Pre-commit security check:**

```bash
claude --agent /path/to/claude-code-agents/security-code-reviewer.md "Review the changes in my current git diff"
```

**Dependency audit before a release:**

```bash
claude --agent /path/to/claude-code-agents/dependency-auditor.md "Full audit for the upcoming v2.0 release"
```

**Check test coverage for a specific module:**

```bash
claude --agent /path/to/claude-code-agents/test-coverage-checker.md "Analyze coverage for the authentication module"
```

**Optimize a specific area of the codebase:**

```bash
claude --agent /path/to/claude-code-agents/code-optimizer.md "Check complexity and duplication in src/services"
```

**Audit docs after a refactor:**

```bash
claude --agent /path/to/claude-code-agents/doc-auditor.md "Check if any docs reference the old API endpoints we removed"
```

## Language Support

These agents support multiple languages and ecosystems:

- **Python** -- radon, vulture, pylint, ruff, bandit, semgrep, pip-audit, safety, pytest-cov, coverage.py
- **JavaScript/TypeScript** -- eslint, knip, jscpd, semgrep, npm audit, Jest, Mocha, Vitest
- **Rust** -- cargo-audit, cargo-tarpaulin
- **Go** -- govulncheck, go test

Agents will detect which tools are installed and use them when available. If a tool isn't installed, agents fall back to pattern-based analysis using Grep and Bash.

## Optional Analysis Tools

The agents work out of the box, but they produce better results when specialized analysis tools are installed in your project environment. Install the ones relevant to your stack:

**Python:**

```bash
pip install radon vulture ruff bandit semgrep pip-audit safety pytest-cov
```

**JavaScript/TypeScript:**

```bash
npm install -g jscpd
# eslint, jest, vitest, etc. are typically already in your project
```

**Rust:**

```bash
cargo install cargo-audit cargo-tarpaulin
```

**Go:**

```bash
go install golang.org/x/vuln/cmd/govulncheck@latest
```

## How Agent Files Work

Each agent is a Markdown file with YAML frontmatter:

```yaml
---
name: my-agent            # Identifier used to reference the agent
description: What it does # Shown in agent listings
tools: Read, Grep, Glob, Bash  # Tools the agent can use
model: sonnet             # Claude model to use
color: blue               # Color for UI display
---

The rest of the file is the system prompt that defines the agent's
behavior, methodology, and output format.
```

You can customize these agents by editing the Markdown files, or create your own by following the same format. For more details on authoring agents, see the [Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code).

## Tips

- **Run agents from your project root** so they can discover config files, lock files, and project structure automatically.
- **Be specific with prompts** when you want to focus on a particular directory, module, or concern.
- **Combine agents** for a comprehensive review -- run the security reviewer, then the test coverage checker to verify security-critical paths are tested.
- **Install the recommended tools** for your language to get the most accurate results. The agents will tell you what's missing.
