# BrowserStack Accessibility DevTools for GitHub

Catch accessibility issues on your pull requests. BrowserStack scans the changed
code on every PR and posts the findings right back on the PR — a summary comment,
inline comments on the offending lines, a merge check, and a SARIF upload to the
Security tab.

- 🔎 **Scans only what changed** in the PR (fast, even on large repos).
- 💬 **Results on the PR** — a sticky summary comment and inline comments.
- ✅ **Merge gate** — fail the check when accessibility issues are found (configurable).
- 🔐 **Zero write credentials on your runner** — every write is posted by the
  BrowserStack Accessibility App; your workflow's `GITHUB_TOKEN` stays read-only.

Learn more: <https://www.browserstack.com/docs/accessibility-dev-tools/features/remediate-github>

---

## Prerequisites

1. **Install the BrowserStack Accessibility GitHub App** on your organization or
   repository (a one-time step, done by a repo/org admin).
2. **Add a BrowserStack Service Account key** as repository or organization
   **Actions secrets**:
   - `BROWSERSTACK_USERNAME`
   - `BROWSERSTACK_ACCESS_KEY`

## Quick start

Add `.github/workflows/browserstack-a11y.yml`. It runs **automatically on every
pull request**, and can also be re-run on demand by commenting
`@AccessibilityDevTools` on the PR:

```yaml
name: BrowserStack Accessibility DevTools

on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review]
  issue_comment:
    types: [created]

permissions:
  contents: read # read the PR's changed files
  pull-requests: read # resolve the PR
  id-token: write # prove the run came from your CI (required)

concurrency:
  group: a11y-${{ github.event.pull_request.number || github.event.issue.number }}
  cancel-in-progress: true

jobs:
  a11y:
    if: >
      github.event_name == 'pull_request' ||
      (github.event.issue.pull_request &&
       contains(github.event.comment.body, '@AccessibilityDevTools'))
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: browserstack/browserstack-accessibility-devtools-action@main
        with:
          username: ${{ secrets.BROWSERSTACK_USERNAME }}
          access-key: ${{ secrets.BROWSERSTACK_ACCESS_KEY }}
          fail-on-severity: error
```

That's it — open a PR and the results appear within a couple of minutes. To
re-run manually, comment `@AccessibilityDevTools` on the PR.

## How it works

1. On a pull request (or an `@AccessibilityDevTools` comment), the workflow runs
   BrowserStack's accessibility CLI against the PR's changed files, authenticated
   by your Service Account key.
2. Results are posted back to the PR **by the BrowserStack Accessibility App** (a
   branded bot), so your workflow's default token never needs write access.

> **Why `id-token: write`?** GitHub mints a short-lived, repo-scoped OpenID
> Connect token that proves the request genuinely came from this repository's CI
> run. BrowserStack verifies it before posting. It carries **no** personal
> identity and grants no standing access.

## What gets posted

On every scan the App posts, automatically:

- a **sticky summary comment** (one per PR, updated in place across runs);
- **inline comments** on the offending lines, in the same format as the
  BrowserStack VS Code extension: `(rule): <description>` followed by how to fix it;
- a **Check Run** with the pass/fail result;
- a **SARIF upload** to GitHub code scanning (Security tab).

## Inputs

| Input              | Default | Description                                                                                             |
| ------------------ | ------- | ------------------------------------------------------------------------------------------------------- |
| `username`         | —       | **Required.** Service Account username (store as a secret).                                             |
| `access-key`       | —       | **Required.** Service Account access key (store as a secret).                                           |
| `fail-on-severity` | `error` | Fail the check when findings at/above this severity exist: `error`, `warning`, or `none` (never fails). |
| `ai-agent`         | —       | Optional (preview). Bare agent name to `@mention` for AI remediation hand-off — see below.              |

## Outputs

| Output           | Description                          |
| ---------------- | ------------------------------------ |
| `result`         | `pass` or `fail`.                    |
| `error-count`    | Number of error-severity findings.   |
| `warning-count`  | Number of warning-severity findings. |
| `findings-count` | Total findings.                      |
| `comment-url`    | Link to the posted PR summary comment. |

## AI remediation hand-off (preview)

Set `ai-agent` to the bare name of an agent you already use (e.g. `coderabbitai`).
When findings are posted, the App `@mentions` it on the comment, and your agent
acts under **your** own credentials and billing.

> This feature will be supported if your AI agent accepts triggers from a bot/App
> comment; behaviour varies by agent, and it is a silent no-op if the agent isn't
> configured to accept it.

## Support

<https://www.browserstack.com/support>
