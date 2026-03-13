# D2: Runtime Failure & Success Analysis

## Environment
- OS: Windows 11
- ARTEMIS version: main branch (Stanford-Trinity/ARTEMIS)
- Model: anthropic/claude-sonnet-4
- LiteLLM Proxy: https://llmproxy.secruin.cloud/v1
- Test config: configs/tests/ctf_easy.yaml

## Observed Failures

### Failure 1: Codex Binary Path (Windows-specific)
- **Error**: `Codex binary not found: codex-rs/target/release/codex`
- **Root Cause**: Supervisor hardcodes Unix path without `.exe` extension
- **Impact**: Agents cannot be spawned, pipeline halts after LLM decision
- **Fix Applied**: Pass `--codex-binary` flag with full `.exe` path
- **Learning**: ARTEMIS is designed for Linux — Windows requires manual path fixes

### Failure 2: Authentication Error (401)
- **Error**: `openai.AuthenticationError: Missing Authentication header`
- **Root Cause**: LiteLLM proxy requires `/v1` suffix in base URL
- **Impact**: All LLM calls fail — supervisor cannot make decisions
- **Fix Applied**: Set `OPENAI_BASE_URL=https://llmproxy.secruin.cloud/v1`

### Failure 3: Unicode Encoding Error
- **Error**: `UnicodeEncodeError: charmap codec can't encode character \u2705`
- **Root Cause**: Windows CMD uses cp1252 encoding, not UTF-8
- **Impact**: Cosmetic only — logs show garbled emoji but logic is unaffected
- **Fix Applied**: `set PYTHONIOENCODING=utf-8` before running

### Failure 4: Windows Defender Quarantine
- **Error**: `cannot find file shelling.md`
- **Root Cause**: Windows Defender flagged shellcode-related files as malware
- **Impact**: Build failure — codex binary could not compile
- **Fix Applied**: Added ARTEMIS folder to Defender exclusions, restored files

## Successes Observed
- Supervisor session initialization: SUCCESS
- Config loading from YAML: SUCCESS
- LLM model selection (claude-sonnet-4): SUCCESS
- Benchmark mode activation: SUCCESS
- Session logging to logs/: SUCCESS
- Codex binary compilation (27 min build): SUCCESS

## Key Observations
- ARTEMIS is tightly coupled to Linux/Unix environments
- The supervisor gracefully shuts down on errors (no crashes)
- Each run creates a unique session log with timestamp
- Benchmark mode skips triage — useful for CTF scenarios