# Market Researcher — managed-agent template

> Verification status (2026-11-25): this is a workflow specification/template. Manifest references passed structural checks; no business-output or hosted-agent acceptance run was performed. Linux deployment dry runs are blocked by the deployment script's GNU `mktemp` incompatibility. Worker schemas/permission descriptions below express intended requirements: `deploy-managed-agent.sh` does not currently wire `validate.py` into worker responses. See the [repository feature audit](../../README.md#feature-verification--2026-11-25) and [roadmap](../../ROADMAP.md).

## Overview

Sector or theme → industry overview → competitive landscape → peer comps → ideas shortlist → research note. Same source as the [`market-researcher`](../../plugins/agent-plugins/market-researcher) Cowork plugin — this directory is the Managed Agent cookbook for `POST /v1/agents`.

## Deploy

```bash
export ANTHROPIC_API_KEY=sk-ant-...
export CAPIQ_MCP_URL=... FACTSET_MCP_URL=...
../../scripts/deploy-managed-agent.sh market-researcher
```

## Steering events

See [`steering-examples.json`](./steering-examples.json). Kick from a research-queue event or fan out across a coverage map.

## Security & handoffs

Third-party reports and issuer materials are untrusted. Three-tier isolation:

| Tier | Touches untrusted docs? | Tools | Connectors |
|---|---|---|---|
| **`sector-reader`** | **Yes** | `Read`, `Grep` only | None |
| `comps-spreader` / Orchestrator | No | `Read`, `Grep`, `Glob`, `Agent` | CapIQ, FactSet (read-only) |
| **`note-writer`** (Write-holder) | No | `Read`, `Write`, `Edit` | None |

`sector-reader` is specified to return length-capped, schema-valid JSON; runtime validation is not wired into deployment. `note-writer` produces `./out/primer-<sector>.docx` (and `.pptx` if slides requested).

**Handoff:** to model a single name surfaced in the ideas shortlist, emit a `handoff_request` for `model-builder`; `scripts/orchestrate.py` routes it as a new steering event.
