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

## Stealth & anti-bot

**robots.txt is not enforced** — playwright-cli ignores it by default. No action needed.

**Prefer real Chrome over Chromium** — Chromium exposes `navigator.webdriver=true`; real Chrome does not:
```bash
playwright-cli open --browser=chrome https://target.com
```

**Use a persistent profile** — in-memory sessions look synthetic; a persistent profile accumulates cookies, history, and fingerprint signals that match a real user:
```bash
playwright-cli open --browser=chrome --persistent https://target.com
```

**Connect to an existing browser** — bypasses headless detection entirely by attaching to a real running browser:
```bash
playwright-cli open --extension
```

**Inject stealth overrides** — patch `navigator.webdriver` and other headless signals before the page loads:
```bash
playwright-cli run-code "async page => {
  await page.addInitScript(() => {
    Object.defineProperty(navigator, 'webdriver', { get: () => undefined });
  });
}"
```

**Mock bot-detection endpoints** — intercept and spoof fingerprint/analytics calls:
```bash
playwright-cli route "**/fingerprint*" --status=200 --body='{}'
playwright-cli route "**/bot-detection*" --status=200
```

**Act human** — bot walls score interaction cadence; add pauses between actions, move the mouse before clicking:
```bash
playwright-cli mousemove 200 300
playwright-cli mousedown
playwright-cli mouseup
# then click the actual target
playwright-cli click e5
```

**Handle CAPTCHAs and redirects** — take a screenshot when a page gate is hit, identify what type it is, then decide: solve via `eval`, load a saved auth state, or switch to `--extension` mode with a real browser session that already passed it.
