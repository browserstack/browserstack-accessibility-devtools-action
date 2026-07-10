# BrowserStack Accessibility for GitHub

Catch accessibility issues on your pull requests. Comment
**`@AccessibilityDevTools`** on a PR and BrowserStack scans the changed code,
then posts the findings right back on the PR — as a summary comment, inline
suggestions, and an optional merge check.

- 🔎 **Scans only what changed** in the PR (fast, even on large repos).
- 💬 **Results on the PR** — a summary comment and inline, one‑click suggestions.
- ✅ **Optional merge gate** — fail the check when accessibility errors are found.
- 🤖 **Optional agent hand‑off (preview)** — the findings comment can `@mention` a PR
  agent you already use, which then acts under _your_ credentials. BrowserStack runs no
  AI model and never handles your AI keys. **v1 is scan + report**; whether the agent
  reviews or fixes is up to that agent (see "Optional: agent hand-off" below for what
  works today).

Learn more: <https://browserstack.com>

---

## Prerequisites

1. **Install the BrowserStack Accessibility GitHub App** on your organization or
   repository (a one‑time step, done by a repo/org admin).
2. **Add a BrowserStack Service Account key** as repository or organization
   **Actions secrets**:
   - `BROWSERSTACK_USERNAME`
   - `BROWSERSTACK_ACCESS_KEY`

   A Service Account is a non‑personal, admin‑managed key built for CI — it uses
   no user license and can be rotated or revoked independently. (Enterprise plan.)

## Quick start

Add `.github/workflows/browserstack-a11y.yml`:

```yaml
name: BrowserStack Accessibility
on:
  issue_comment:
    types: [created]

permissions:
  contents: read # read the PR's changed files
  pull-requests: read # resolve the PR
  id-token: write # prove the run came from your CI (required)

concurrency:
  group: a11y-${{ github.event.issue.number }}
  cancel-in-progress: true

jobs:
  a11y:
    # Fires on a PR comment mentioning @AccessibilityDevTools. The action then verifies
    # the commenter has write/maintain/admin permission before scanning.
    if: >
      github.event.issue.pull_request &&
      contains(github.event.comment.body, '@AccessibilityDevTools')
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      # @v1 tracks the latest v1.x; pin to a full commit SHA for supply-chain hardening.
      - uses: browserstack/browserstack-accessibility-devtools-action@v1
        with:
          username: ${{ secrets.BROWSERSTACK_USERNAME }}
          access-key: ${{ secrets.BROWSERSTACK_ACCESS_KEY }}
          comment: true
          check-gate: true
          fail-on-severity: error
```

A ready-to-copy version lives at [`examples/browserstack-a11y.yml`](examples/browserstack-a11y.yml).

Then, on any pull request, comment:

```
@AccessibilityDevTools
```

The scan runs and results appear on the PR within a couple of minutes.

## How it works

1. You comment **`@AccessibilityDevTools`** on the PR.
2. The workflow runs BrowserStack's accessibility CLI against the PR's changed
   files, authenticated by your Service Account key.
3. Results are posted back to the PR **by the BrowserStack Accessibility App**
   (a branded bot), so your workflow's default token never needs write access.

> **Why `id-token: write`?** GitHub mints a short‑lived, repo‑scoped OpenID
> Connect token that proves the request genuinely came from this repository's CI
> run. BrowserStack verifies it before posting. It carries **no** personal
> identity and grants no standing access.

## Inputs

| Input                | Default  | Description                                                                                                                              |
| -------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `username`           | —        | **Required.** Service Account username (store as a secret).                                                                              |
| `access-key`         | —        | **Required.** Service Account access key (store as a secret).                                                                            |
| `comment`            | `true`   | Post a summary findings comment on the PR.                                                                                               |
| `check-gate`         | `true`   | Publish a Check Run / commit status for the PR.                                                                                          |
| `fail-on-severity`   | `error`  | Fail the check at this severity and above: `error`, `warning`, or `none`.                                                                |
| `inline-suggestions` | `false`  | Add one‑click suggestion blocks on offending lines.                                                                                      |
| `sarif`              | `false`  | Publish results to GitHub code scanning (Security tab). Uploaded by the App server-side — your workflow token needs no extra permission. |
| `comment-mode`       | `update` | `update` a single sticky comment across runs, or post a `new` one each time.                                                             |
| `remediation`        | `false`  | Enable the optional agent hand‑off (preview; see below).                                                                                 |
| `ai-agent`           | —        | Required when `remediation: true`. The agent's name (e.g. `coderabbitai`, `claude`); we @‑mention it.                                    |

## Outputs

| Output           | Description                          |
| ---------------- | ------------------------------------ |
| `result`         | `pass` or `fail`.                    |
| `error-count`    | Number of error‑severity findings.   |
| `warning-count`  | Number of warning‑severity findings. |
| `findings-count` | Total findings.                      |
| `comment-url`    | Link to the posted PR comment.       |
| `sarif-file`     | Path to the SARIF file (if enabled). |

## Optional: agent hand‑off (preview)

Set `remediation: true` and name your agent with `ai-agent`. When findings are
posted, the App `@mentions` that agent on the PR (for example, `@coderabbitai`), and
**your** agent acts under **your** credentials and billing.

```yaml
with:
  username: ${{ secrets.BROWSERSTACK_USERNAME }}
  access-key: ${{ secrets.BROWSERSTACK_ACCESS_KEY }}
  remediation: true
  ai-agent: coderabbitai # BrowserStack posts "@coderabbitai"
```

What works today (honest status — this is a preview, off by default):

- BrowserStack **runs no AI model and never stores your AI credentials.**
- The agent must accept a trigger from a **bot/App** comment.
  - **CodeRabbit** — responds to the mention, but produces a **review**, not a fix PR.
  - **Claude** (`claude-code-action` / Anthropic Claude GitHub App) — a genuine fix
    path, but **not yet reliable** for bot-authored triggers (upstream issues; being
    validated). Treat as experimental.
  - **GitHub Copilot's coding agent** — **not supported** (it only acts on comments from
    a human with write access).
- If your agent ignores bot authors by default, allow‑list the **BrowserStack
  Accessibility** bot in its config.

> **v1 is scan + report.** A guaranteed AI-_fix_ path is not part of v1; the hand-off
> above is a preview and is a **silent no-op** if the agent isn't configured to accept it.

## Privacy & security

- Uses a **Service Account** key (not a personal login); store it as an encrypted
  Actions secret and rotate via your admin.
- The App's credentials stay on BrowserStack's side — they are **never** placed on
  your runner.
- The scan reads only the pull request's changed files.

## Support

- Product & docs: <https://browserstack.com>
- Accessibility API reference (including rate limits): <https://www.browserstack.com/docs/accessibility/api>

## License

[MIT](LICENSE)
