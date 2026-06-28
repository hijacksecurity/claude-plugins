# Hijack Security — Claude Code plugins

The official Claude Code marketplace for [Hijack Security](https://hijacksecurity.com).
Add it once, then install any plugin below.

```bash
claude plugin marketplace add hijacksecurity/claude-plugins
claude plugin install <plugin>@hijacksecurity
```

## Plugins

| Plugin | Install | What it does |
|---|---|---|
| **Intercept** | `claude plugin install intercept@hijacksecurity` | Deep AI security scanning that supplements your scanners (container images, live cloud/infra posture, cross-file logic & auth bugs) and writes findings back to the [Intercept](https://intercept.hijacksecurity.com) platform as your system of record. See [`intercept/`](./intercept) for full docs. |

## Updating

```bash
claude plugin update intercept@hijacksecurity   # restart Claude Code to apply
```

`claude plugin marketplace update hijacksecurity` refreshes the catalog if a new plugin or version doesn't show up.

---

This repository is a generated artifact published automatically — please don't hand-edit it.
Issues and feedback: <https://github.com/hijacksecurity/claude-plugins/issues>
