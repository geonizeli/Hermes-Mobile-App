# Agent instructions (Hermes-Mobile-App)

Cloud agents and Cursor agents working in this repository must follow these rules in addition to the user’s message.

## Pull requests — default is **no PR**

- **Do not open, update, or push a feature branch for a pull request** unless the user clearly asks you to change the codebase, implement a fix, or open a PR.
- Treat the following as **read-only / explain-only** unless the user then asks you to fix something:
  - GitHub Actions run or job URLs
  - CI failure links, screenshots, or logs
  - “What failed?” / “Why did this fail?”
  - Questions about runners, queues, secrets, or docs
- After diagnosis, **stop**: give a short root cause and what a human can do (re-run, wait, change settings). Do not add workflows, retry logic, or README churn “while you’re here.”
- If a code change might help but was **not** requested, **ask once** whether to implement and whether to open a PR — do not assume.

## When a PR *is* appropriate

Open a PR only when the user has asked for implementation (e.g. “fix this,” “add X,” “open a PR,” “merge a solution”). Then:

- Keep the diff minimal and scoped to what they asked for.
- One logical change per PR when possible.
- In the PR description, state what the user asked for; do not frame unsolicited work as the main goal.

## CI and iOS builds (context only)

- **Build unsigned iOS IPA** requires GitHub **hosted macOS arm64** runners. Failures with **zero steps** and *not acquired by Runner* are GitHub capacity, not necessarily a repo defect.
- Do not change `runs-on` labels or add retry workflows unless the user explicitly requests that.

## Testing

- Do not run large test or release pipelines for diagnosis-only tasks.
- Run tests when you were asked to implement a change and validation is expected.

## Cursor Cloud

- Prefer answering in chat over opening PRs for investigation threads.
- If the user's Cursor team has stricter rules, those win over this file.