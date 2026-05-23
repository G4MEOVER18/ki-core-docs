# Contributing

## Documentation Updates

- All docs are in `docs/` — numbered for reading order
- Use the same section structure as existing files
- No secrets, API keys, passwords, or personal data in any file

## Config Templates

Templates live in `config/templates/`. They use `<PLACEHOLDER>` syntax for all sensitive values.

## Reporting Issues

Open an issue describing:
1. Which node / component is affected
2. What the expected vs actual behavior is
3. Any relevant log output (sanitized — remove keys/passwords)

## Node-Specific Notes

- AI-Core docs → edit on AI-Core, send to CyberNode for GitHub push
- Architecture changes → update `docs/01-architecture.md` first, then node-specific files
