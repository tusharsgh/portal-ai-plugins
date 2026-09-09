---
inclusion: always
---

# Spotify Portal Workflows

Use the Portal CLI (`npx @spotify/portal-cli`) as the source of truth for all Portal operations. Always run `--help` before relying on a command or flag. Prefer `--json` when consuming output. Never ask the user to paste tokens or credentials into chat.

---

## Setup

Use when the user asks to install, connect, or authenticate Portal, or select a Portal instance.

### Critical rules
1. Never ask the user to paste access tokens, authorization codes, or credentials into chat.
2. Never assume the default instance when more than one instance is listed — ask the user which to use.

### Workflow

**1. Verify Node.js and CLI discovery**
```bash
node --version && npm --version
npx @spotify/portal-cli --help
npx @spotify/portal-cli auth --help
```
The CLI must expose `auth`, `actions`, `owner`, `search`, and `service`. If absent, a newer CLI release is required — stop and report.

**2. Inspect existing instances**
```bash
npx @spotify/portal-cli auth list
```
Reuse an instance whose backend URL matches the user's target. If multiple exist, show names and URLs and ask the user to choose.

**3. Authenticate when needed**
```bash
npx @spotify/portal-cli auth login --instance <name> --backend-url <url>
```
The command may open a browser. Let the user complete auth there.

**4. Select and verify**
```bash
npx @spotify/portal-cli auth select --instance <name>
npx @spotify/portal-cli auth show --instance <name>
```

**5. Verify access**
```bash
npx @spotify/portal-cli actions list --json
```
Success proves the CLI can authenticate, reach the instance, and read available actions.

---

## Doctor (read-only diagnostics)

Use when setup is failing, commands are missing, or the user asks if Portal is ready. Do not install, authenticate, select an instance, or invoke a mutating action.

```bash
npx @spotify/portal-cli auth list
npx @spotify/portal-cli auth show
npx @spotify/portal-cli actions list --json
```

Return a compact table:

| Check | Status | Evidence or next action |
|-------|--------|------------------------|
| CLI | Ready or blocked | Runtime and required command surface |
| Authentication | Ready, ambiguous, or blocked | Instance name and backend URL |
| Actions | Ready or blocked | Result of `actions list --json` |

---

## Search

Use when the user asks to find services, APIs, systems, components, owners, or Portal documentation.

```bash
npx @spotify/portal-cli search <query> --limit 10 --json
```

- Use `--type software-catalog` or `--type techdocs` only when the request clearly targets one source.
- Return a shortlist — do not dump the raw result.
- Preserve exact entity references, titles, locations, and links.
- If the first search is empty, retry once with fewer or broader terms.
- Do not invent services or documentation when results are sparse.

---

## Service Briefing

Use when the user asks who owns a service, whether it is healthy, where its runbook or docs are, or requests an operational briefing.

```bash
npx @spotify/portal-cli owner --help
npx @spotify/portal-cli owner <service-name> --json
```

| User need | Command |
|-----------|---------|
| Owner, on-call, Slack, or runbook | `owner <service-name> --json` |
| Build, deployment, runtime, or incidents | `service status <entity-ref> --json` |
| Service documentation | `service docs <entity-ref> --json` |
| Complete briefing | Run all workflows above |

Treat unavailable dimensions as unavailable, not healthy. Preserve freshness evidence and source links.

---

## Actions

Use when the user asks what Portal can do, names an action ID, or wants to run a Portal operation.

**Discover and inspect**
```bash
npx @spotify/portal-cli actions list --json
npx @spotify/portal-cli actions <action-id> --help
```

**Invoke safely**
For read-only actions, use JSON output and pass only documented flags.

For mutations:
1. Inspect generated help.
2. Preview with `--dry-run --json` and show the user what will change.
3. Execute only after explicit user authorization.
4. Add `--yes` only when the action is marked destructive and the user authorized it.

```bash
npx @spotify/portal-cli actions <action-id> --input '<json>' --dry-run --json
```

Never infer successful execution from a dry run.

---

## Feedback

Use when the user explicitly asks to send feedback about the Portal CLI or these workflows.

```bash
npx @spotify/portal-cli actions telemetry:submit-feedback --text "<feedback>" --source cli --json
```

1. Show the exact text that will be sent and ask for confirmation before invoking.
2. Pass feedback verbatim — do not summarize or translate it.
3. Do not include secrets, tokens, personal data, or internal URLs.
4. A `{"submitted": true}` response means feedback was delivered (one-way — no reply or ticket is created).
