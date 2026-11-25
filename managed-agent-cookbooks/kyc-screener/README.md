# KYC Screener — managed-agent template

> Verification status (2026-11-25): this is a workflow specification/template. Manifest references passed structural checks; no business-output or hosted-agent acceptance run was performed. Linux deployment dry runs are blocked by the deployment script's GNU `mktemp` incompatibility. Worker schemas/permission descriptions below express intended requirements: `deploy-managed-agent.sh` does not currently wire `validate.py` into worker responses. See the [repository feature audit](../../README.md#feature-verification--2026-11-25) and [roadmap](../../ROADMAP.md).

## Overview

Parses onboarding docs, runs the rules engine, screens sanctions/PEP, flags gaps. Same source as the [`kyc-screener`](../../plugins/agent-plugins/kyc-screener) Cowork plugin — this directory is the Managed Agent cookbook for `POST /v1/agents`.

## Deploy

```bash
export ANTHROPIC_API_KEY=sk-ant-...
export SCREENING_MCP_URL=...
../../scripts/deploy-managed-agent.sh kyc-screener
```

## Steering events

See [`steering-examples.json`](./steering-examples.json).

## Security & handoffs

Onboarding documents are untrusted. Three-tier isolation:

| Tier | Touches untrusted docs? | Tools | Connectors |
|---|---|---|---|
| **`doc-reader`** | **Yes** | `Read`, `Grep` only | None |
| `rules-engine` / Orchestrator | No | `Read`, `Grep`, `Glob`, `Agent` | screening (read-only) |
| **`escalator`** (Write-holder) | No | `Read`, `Write`, `Edit` | None |

`doc-reader` is specified to return length-capped, schema-valid JSON; runtime validation is not wired into deployment. `escalator` produces `./out/escalation-<packet>.xlsx`.

**Not guaranteed:** this agent recommends a risk rating; the compliance officer decides.
