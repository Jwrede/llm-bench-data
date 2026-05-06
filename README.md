# llm-bench-data

> Open dataset of hourly LLM API performance measurements.

This repository contains raw JSONL data from continuous probing of major LLM
API endpoints. Updated hourly by
[llm-bench](https://github.com/Jwrede/llm-bench).

## Schema

Each line in the JSONL files is a single probe result:

```json
{
  "provider": "openai",
  "model": "anthropic/claude-sonnet-4.6",
  "status": "healthy",
  "ttft_ms": 312,
  "latency_ms": 2100,
  "tokens_per_sec": 68.4,
  "token_count": 20,
  "timestamp": "2026-05-06T14:00:00Z"
}
```

| Field | Type | Description |
|-------|------|-------------|
| provider | string | Protocol used for probing (always "openai" when routed via OpenRouter) |
| model | string | Model slug in provider/name format (e.g. "anthropic/claude-sonnet-4.6") |
| status | string | "healthy" or "error" |
| ttft_ms | float | Time to first token in milliseconds |
| latency_ms | float | Total request duration in milliseconds |
| tokens_per_sec | float | Generation throughput (tokens after first / generation time) |
| token_count | int | Total output tokens |
| timestamp | string | ISO 8601 UTC timestamp of the probe |

## File organization

```
data/
  2026-05/
    2026-05-06.jsonl
    2026-05-07.jsonl
    ...
  2026-06/
    ...
```

Files are organized by month and day. Each file contains all probe results
for that UTC day.

## Probe configuration

Each probe sends "Hi" with max 20 output tokens to measure infrastructure
latency rather than model generation time. Probes run hourly and are
currently routed through [OpenRouter](https://openrouter.ai), so latency
numbers include proxy overhead. Models are selected automatically from
OpenRouter's weekly popularity rankings (top 3 per provider).

## Usage

```bash
# clone the dataset
git clone https://github.com/Jwrede/llm-bench-data.git

# average TTFT for a model over a day
cat data/2026-05/2026-05-06.jsonl \
  | jq -s '[.[] | select(.model == "openai/gpt-5.5")] | (map(.ttft_ms) | add / length)'

# all errors for a given day
cat data/2026-05/2026-05-06.jsonl | jq -c 'select(.status == "error")'

# throughput comparison across all models
cat data/2026-05/2026-05-06.jsonl \
  | jq -s 'group_by(.model) | map({model: .[0].model, median_tps: (map(.tokens_per_sec) | sort | .[length/2])})'
```

## License

This data is released under [CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/).
Use it however you want.
