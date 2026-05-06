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
  "model": "gpt-4o",
  "ttft_ms": 312,
  "latency_ms": 2100,
  "tokens_per_sec": 68.4,
  "token_count": 20,
  "status": "ok",
  "error": "",
  "timestamp": "2026-05-06T14:00:00Z"
}
```

| Field | Type | Description |
|-------|------|-------------|
| provider | string | API provider (openai, anthropic, google, deepseek, xai) |
| model | string | Model identifier as sent to the provider API |
| ttft_ms | float | Time to first token in milliseconds |
| latency_ms | float | Total request duration in milliseconds |
| tokens_per_sec | float | Generation throughput (tokens after first / generation time) |
| token_count | int | Total output tokens |
| status | string | "ok" or "error" |
| error | string | Error message if status is "error", empty otherwise |
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
latency rather than model generation time. Probes run hourly against the
provider's native API endpoint (not through aggregators).

## Usage

```bash
# clone the dataset
git clone https://github.com/Jwrede/llm-bench-data.git

# query with jq: average TTFT for GPT-4o over a day
cat data/2026-05/2026-05-06.jsonl | jq -s '[.[] | select(.model == "gpt-4o")] | (map(.ttft_ms) | add / length)'
```

## License

This data is released under [CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/).
Use it however you want.
