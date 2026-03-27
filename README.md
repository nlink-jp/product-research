# product-research

A CLI agent tool that autonomously researches a specified product or service on the web and outputs a structured report covering overview, terms of service, privacy practices, and data security.

Two backends are available:

| Backend | Script | Search engine |
|---|---|---|
| Anthropic (Claude) | `research_agent.py` | Anthropic `web_search` tool |
| Google (Gemini) | `research_agent_gemini.py` | Google Search Grounding (Vertex AI) |

[日本語版 README はこちら](README.ja.md)

## Features

- **Autonomous web research** — Generates multiple search queries automatically to collect information including official documentation, ToS, and privacy policies
- **Structured JSON output** — Type-safe extraction via Pydantic schemas; directly usable by downstream tools
- **Data handling and security fields** — Dedicated fields for data collection, usage, sharing, encryption, authentication, and restrictions on sensitive data
- **Risk level assessment** — Three-tier overall risk rating: `low / medium / high`
- **Pipe-friendly** — `--json-only --no-save` outputs JSON to stdout for use with `jq` and other tools

## Installation

**Prerequisites:** Python 3.11+ and [uv](https://docs.astral.sh/uv/).

```bash
git clone https://github.com/nlink-jp/product-research.git
cd product-research
uv sync
```

## Configuration

### Anthropic backend

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

### Google Gemini backend (Vertex AI)

Requires a Google Cloud project and [gcloud CLI](https://cloud.google.com/sdk/docs/install) with Vertex AI API enabled.

```bash
gcloud auth application-default login

export GOOGLE_CLOUD_PROJECT="your-project-id"
export GOOGLE_CLOUD_LOCATION="us-central1"  # optional, defaults to us-central1
```

## Usage

### Anthropic backend

```bash
# Basic research
uv run research_agent.py "Slack"

# Specify output directory
uv run research_agent.py "ChatGPT" --output-dir ./reports

# Show verbose log (search queries, progress)
uv run research_agent.py "Notion" --verbose

# Output JSON only to stdout (no file save)
uv run research_agent.py "Dropbox" --json-only --no-save

# Combine with jq
uv run research_agent.py "GitHub Copilot" --json-only | jq '.data_security'
```

### Google Gemini backend

```bash
uv run research_agent_gemini.py "Slack"
uv run research_agent_gemini.py "ChatGPT" --output-dir ./reports
uv run research_agent_gemini.py "Notion" --verbose
uv run research_agent_gemini.py "Dropbox Business" --json-only --no-save
```

### Options (both backends)

| Option | Short | Default | Description |
|---|---|---|---|
| `--output-dir` | `-o` | `./reports` | Report output directory |
| `--verbose` | `-v` | off | Show search queries and reference URLs |
| `--json-only` | — | off | Output JSON only to stdout |
| `--no-save` | — | off | Do not save files |

### Output format

Reports are saved under `./reports/`:

```
reports/
├── Slack_20260314_120000.md          # Anthropic Markdown
├── Slack_20260314_120000.json        # Anthropic JSON
├── Slack_gemini_20260314_120000.md   # Gemini Markdown
└── Slack_gemini_20260314_120000.json # Gemini JSON
```

The JSON schema includes: `overview`, `terms_of_service`, `user_data_handling`, `data_security`, `overall_risk_level`, `cautions`, and `sources`.

### Notes

- Research results are based on publicly available web information. Always verify the latest ToS and privacy policy on official sites.
- Fields where no information is found are reported as `"unknown"` — no guessing.
- **Anthropic backend:** API usage costs apply (Claude Opus 4.6). Expect tens of thousands to ~100k tokens per research run.
- **Gemini backend:** Vertex AI costs apply (Gemini 2.5 Pro). Check your Google Cloud project billing.

## Building

```bash
# Type checking
uv run pyright
```

## Documentation

- [Development rules](./AGENTS.md)
