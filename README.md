# CallSaaS Tools Library

Universal library of ElevenLabs voice agent tools. Every tool here is pre-built,
pre-tested, and ready to deploy to any client with only tenant_id and webhook_url
filled in.

## Structure

```
core/          — Every client gets these (reserve, update, finalize)
booking/       — Appointment and calendar tools
communication/ — SMS reminders, confirmations, callback requests
escalation/    — Live transfer, emergency routing
information/   — Read-only lookups (hours, services)
industry/      — Niche-specific tools
```

## Each tool contains

```
[tool_name]/
├── definition.json         ← ElevenLabs tool schema — deploy this
├── server_spec.md          ← What the server handler does
├── test_payload.json       ← Simulated ElevenLabs call input
├── expected_response.json  ← What server should return
└── meta.json               ← Tags, version, status, usage history
```

## Status values

- `draft`         — Ash wrote definition + spec, cc not yet involved
- `tested`        — cc built handler and ran test suite
- `library-grade` — Validated, reusable, production ready
- `deprecated`    — Superseded by newer version

## Versioning

- Integer versions: v1, v2, v3
- Minor fixes (descriptions, typos): same version, no re-deployment
- Any parameter or payload change: new version required
- Old versions kept so existing clients are never broken

## Adding a new tool

1. Ash searches library by name/tags — if not found, she builds it
2. Ash writes: definition.json, server_spec.md, meta.json (status: draft)
3. Ash alerts cc: "New tool ready for handler: [name]"
4. cc builds server handler, writes test_payload + expected_response
5. cc runs tests, sets status to library-grade
6. Tool available for all future clients
