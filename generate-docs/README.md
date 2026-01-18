# generate-docs

`generate-docs` is an Agent Skill that analyzes a codebase and generates a small set of technical documents suitable for onboarding and handover.

## What it generates

By default, it writes to `docs/`:

- `index.md` — quick start + links to the other docs
- `overview.md` — project overview, architecture, key flows, data model
- `api-reference.md` — endpoint inventory with request/response examples (when applicable)
- `setup-guide.md` — prerequisites, setup steps, operational commands, troubleshooting

## How to run

```text
/generate-docs [output-directory]
```

If `[output-directory]` is omitted, the output directory defaults to `docs/`.

## Templates and examples

- Templates: `TEMPLATES.md`
- Example runs + expected output: `EXAMPLES.md`

## Safety notes

- Do not overwrite existing docs (e.g., `README.md`).
- Mask secrets (API keys, tokens) using placeholders like `your-api-key-here`.
