# AGENTS.md

Guide for agents and developers working on **tpi-practical-challenge-desktop**.


## Branch flow (hard rules)

- `develop` and `main` get **no direct push**: always via PR.
- No force-push; protected branches cannot be deleted.
- **One approval** required; merge is a **merge commit**.

| Source branch | Valid name | PR to |
| ------------- | ---------- | ----- |
| Feature       | `feature/*`| `develop` |
| Fix           | `fix/*`    | `develop` |
| `main`        | `main`     | `develop` |
| Release       | `release/*`| `main`    |
| Hotfix        | `hotfix/*` | `main`    |

- `main` is reached **only** from `release/*` or `hotfix/*`.
- Typical flow: create `feature/whatever` from `develop` → PR to `develop` → approval → merge. To publish: open `release/*` or `hotfix/*` to `main`.
- **Push authorization (mandatory for agents):** never push to any remote (`origin`) on your own initiative. Always request and obtain the user's **explicit authorization** before `git push` — propose the push and wait for approval. Pushing without it is a violation.

## Pull requests with GitHub CLI (`gh`)

**PRs are created exclusively with the GitHub CLI (`gh`)** — never via the web UI when an agent is doing the work. Eg:

```bash
# from the source branch, before opening the PR
git push -u origin feature/whatever
gh pr create --base develop --head feature/whatever \
  --title "feat: add ..." \
  --body "Describe the change. Link the related issue."
```

- Check the PR's branch-name validation before opening it (see `branching-name-check` above) — a wrong name will be flagged on the PR and isn't bypassable.
- Verify the PR state after creating it:

```bash
gh pr status
gh pr view --web          # open it in the browser for review/approval
```

### If `gh` is not installed

`gh` must be installed on the machine before any PR work. Install it, then authenticate:

**Install per OS:**
- Debian/Ubuntu: `(type -p wget >/dev/null || (sudo apt update && sudo apt-get install wget -y)) && sudo mkdir -p -m 755 /etc/apt/keyrings && out=$(mktemp) && wget -nv -O$out https://cli.github.com/packages/githubcli-archive-keyring.gpg && cat $out | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null && sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null && sudo apt update && sudo apt install gh -y`
- RHEL/Alma/Rocky: `sudo dnf install 'dnf-command(config-manager)' && sudo dnf config-manager --add-repo https://cli.github.com/packages/rpm/gh-cli.repo && sudo dnf install gh`
- macOS (Homebrew): `brew install gh`
- Windows: `winget install --id GitHub.cli` (note: gh lands at `C:\Program Files\GitHub CLI` — make sure it's on PATH)

**Authenticate:**
```bash
gh auth login
# Choose: GitHub.com → HTTPS → authenticate via browser
gh auth setup-git        # wire gh as a git credential helper
gh auth status           # verify
```
If interactive browser login isn't possible (headless/agent session), use the OAuth **device flow**: run `gh auth login`, pick "Login with a web browser", enter the displayed one-time code at `https://github.com/login/device`, and confirm. Verify with `gh auth status` before creating any PR.

## Language

- **All code is written in English**: identifiers, class/method/variable/constant names, endpoint paths, JSON keys, enum values, comments, Javadoc and log/validation messages.
- Prose documentation (this file, docs) may stay in Spanish.
- This includes code adapted from integration kits/templates: anglicize **before** committing, never in a follow-up commit (a "broken commit + fix commit" pair in the same branch is squashed, not reviewed).

## Development quality standards

**Scope & PR discipline**
- **Small, focused PRs**: one PR per logical change; do not mix unrelated edits into a feature.
- Follow the PR template (`.github/pull_request_template.md`) and link the related issue.
- **Rebase the branch onto `develop` before opening the PR** to minimize conflicts.
- **Never merge a PR while its CI is red.** Check the run status first and only merge on green (after the single required approval):
  ```bash
  gh pr checks <branch>     # status of each check
  gh run watch              # follow the run to completion before merging
  ```

**Versioning & release discipline**
- Bump `<version>` in the `pom.xml` **only on `release/*`/`hotfix/*`** — never on `feature/*` or `fix/*` — to avoid accidentally overwriting tags when the branch reaches `main`.
- On `release/*`, regenerate and commit the javadoc (`mvn javadoc:javadoc`), per the release flow.
- **Update `docs/app_doc` when behavior changes** (not only at release time).
- Keep a **`CHANGELOG.md`** fed from the merge commit types (`feat:`, `fix:`, `docs:`, …).

**Secrets**
- **Never commit credentials.** Keep secret values out of `application*.properties`, `application-dev/prod.properties`, and `.env`; use environment variables / CI secrets instead.
- The current `.gitignore` does **not** ignore `.env` — add `.env` to it if you use one.

## Commit convention

- **Conventional Commits** (`feat:`, `fix:`, `docs:`, `refactor:`, `chore:`, `test:`, `style:`, …).
- Commit messages in **English**.
- Bad: `"arreglé el bug del motor"` · Good: `"fix: handle empty attempt list in motor stub"`.