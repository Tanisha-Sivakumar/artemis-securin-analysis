# D1: ARTEMIS Agent Architecture & Communication Flow

## Overview
ARTEMIS (Automated Red Teaming Engine with Multi-agent Intelligent Supervision)
is a two-layer autonomous security testing system built by Stanford Trinity.

## Components

### 1. Supervisor (Python)
- Entry point: `supervisor/supervisor.py`
- Orchestrates the entire attack pipeline
- Reads target configuration from YAML files (configs/)
- Makes all high-level decisions via LLM (claude-sonnet-4)
- Manages session state and logging

### 2. Orchestrator
- File: `supervisor/orchestration/orchestrator.py`
- Sits between supervisor and subagents
- Manages turn-by-turn LLM conversation
- Handles context window, summarization
- Routes tool calls to appropriate handlers

### 3. Codex Subagents (Rust binary - codex.exe)
- Built from: `codex-rs/`
- Spawned by supervisor as child processes
- Execute actual commands: scanning, exploitation, recon
- Report results back via stdout/stderr
- Stateless — each spawn is fresh

## Communication Flow
```
YAML Config
    ↓
Supervisor (Python)
    ↓ reads target info
Orchestrator
    ↓ constructs prompt
LLM (claude-sonnet-4 via LiteLLM proxy)
    ↓ returns action decision
Orchestrator
    ↓ parses tool call
Codex Subagent (Rust binary)
    ↓ executes command on target
Result returned to Orchestrator
    ↓
LLM decides next action
    ↓ (loop continues until time limit or vuln found)
Final Report
```

## Key Design Decisions
- Supervisor uses BENCHMARK MODE to skip triage in CTF scenarios
- Codex binary handles sandboxed execution with workspace-write mode
- LLM context is managed turn-by-turn by orchestrator
- Sessions are logged to logs/supervisor_session_<timestamp>.log

## Tech Stack
- Python 3.11+ (supervisor layer)
- Rust 1.94 (codex agent binary)
- anthropic/claude-sonnet-4 (LLM)
- LiteLLM proxy (API routing)
- Langfuse (observability)
- Docker (target sandboxing)