# D4: Cost & Observability Analysis

## Langfuse Setup
- Dashboard: https://lang.secruin.cloud
- Project: artemis-analysis
- Org: artemis-tanisha-sivakumar

## Current Status
- Langfuse connected: YES
- Traces captured: 0 (due to API auth failure before LLM calls reached proxy)
- This means: LLM calls failed at network layer before Langfuse could log them

## Expected Metrics (from architecture analysis)

| Metric | Expected Value |
|--------|---------------|
| LLM calls per session | 10-30 (depending on vuln complexity) |
| Avg tokens per call | ~2000-4000 |
| Supervisor model | anthropic/claude-sonnet-4 |
| Subagent model | anthropic/claude-sonnet-4 |
| Est. cost per 15min run | $0.10 - $0.50 |

## Observability Architecture
- All LLM calls routed through LiteLLM proxy
- Proxy auto-logs to Langfuse via LANGFUSE_PUBLIC_KEY/SECRET_KEY
- Traces show: prompt → completion → latency → token count
- Session IDs link supervisor logs to Langfuse traces

## Key Learning
The fact that 0 traces appear in Langfuse while the supervisor was running
confirms that the failure happened BEFORE the LLM proxy — at the HTTP
client level (illegal Bearer header format), so no calls reached LiteLLM.