---
name: playwright-cli
description: Browser automation via playwright-cli. Use when navigating websites, interacting with web pages, filling forms, taking screenshots, testing web UIs, or extracting information from pages. Prefer over writing Playwright scripts for agent-driven automation.
allowed-tools: Bash(playwright-cli:*)
---

# playwright-cli

Token-efficient CLI browser control. Always `snapshot` after navigation to get element refs — never guess them.

## When to use

- Agent-driven navigation, clicking, scraping, form flows → **this skill**
- CI test suites with assertions → `@playwright/test`
- Unit/logic tests → Vitest/Jest
- API-only → pytest/supertest

## Pattern

```bash
playwright-cli open https://example.com
playwright-cli snapshot          # get refs (e1, e2, ...)
playwright-cli click e3
playwright-cli fill e5 "value"
playwright-cli screenshot
playwright-cli close
```

## Key commands

| Action | Command |
|---|---|
| Navigate | `goto <url>`, `go-back`, `reload` |
| Interact | `click`, `fill`, `type`, `select`, `check`, `hover`, `drag` |
| Capture | `screenshot`, `screenshot --filename=x.png`, `pdf` |
| Auth | `state-save auth.json` / `state-load auth.json` |
| Network mock | `route "<pattern>" --body='...'`, `unroute` |
| Debug | `console`, `network`, `eval "document.title"` |
| Sessions | `-s=name open <url>`, `list`, `show`, `close-all` |
| Browser | `--browser=chrome\|firefox\|webkit`, `--persistent` |

Run `playwright-cli --help <command>` for full options.
