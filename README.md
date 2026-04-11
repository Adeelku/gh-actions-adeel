# GitHub Actions Foundation Workshop

> **Delivery guide** — follow each section in order. Each section includes talking points, live-demo steps, and sample YAML you can paste into a demo repository.
>
> **MS Learn references:**
> - [Automate your workflow with GitHub Actions — Part 1](https://learn.microsoft.com/en-us/training/paths/github-actions/)
> - [Automate your workflow with GitHub Actions — Part 2](https://learn.microsoft.com/en-us/training/paths/github-actions-2/)

---

## Prerequisites (for the presenter)

| Item | Details |
|------|---------|
| GitHub account | Free tier is fine; a GitHub Enterprise Cloud account unlocks environment protection rules |
| Demo repository | Create a public repo with a small Node.js or Python project (e.g. `npm init -y && echo "console.log('hello');" > index.js`) |
| Azure subscription (optional) | Needed only for the CD / Deploy-to-Azure demo |
| VS Code + GitHub Actions extension | For editing workflow YAML with syntax highlighting |

---

## Section 1 — What is GitHub Actions? (15 min) | 🕑 3:00 PM – 3:15 PM

### Talking Points

- GitHub Actions is a **CI/CD and workflow automation platform** built into every GitHub repository.
- It lets you automate tasks in response to events on your repository: pushes, pull requests, issue comments, schedules, manual dispatch, and more.
- It replaces external CI servers (Jenkins, CircleCI, etc.) with a first-class, integrated experience.
- Actions are **composable**: thousands of community-built actions are available on the [GitHub Marketplace](https://github.com/marketplace?type=actions).

> **MS Learn module:** [Automate development tasks by using GitHub Actions](https://learn.microsoft.com/en-us/training/modules/github-actions-automate-tasks/)

### Live Demo

1. Navigate to any popular open-source repo on GitHub (e.g. `microsoft/vscode`).
2. Click the **Actions** tab — show the list of workflows, run history, and status badges.
3. Click into a run to show the job graph, step logs, and timing.

---

## Section 2 — Core Concepts (25 min) | 🕑 3:15 PM – 3:40 PM

> **MS Learn module:** [Identify the components of GitHub Actions](https://learn.microsoft.com/en-us/training/modules/github-actions-automate-tasks/2b-identify-components-workflow)

### 2.1 Workflows, Jobs, and Steps

| Concept | Description |
|---------|-------------|
| **Workflow** | An automated process defined in a YAML file under `.github/workflows/`. A repo can have many workflows. |
| **Job** | A set of steps that execute on the **same runner**. Jobs run in parallel by default; use `needs:` for ordering. |
| **Step** | A single task inside a job — either a shell command (`run:`) or a reusable action (`uses:`). |
| **Action** | A standalone, reusable command. Can come from the Marketplace, another repo, or a local path. |

**Diagram to draw on whiteboard:**

```
Event (push) ──▶ Workflow (.yml file)
                    ├── Job 1 (runs-on: ubuntu-latest)
                    │     ├── Step 1: actions/checkout@v4
                    │     ├── Step 2: run: npm install
                    │     └── Step 3: run: npm test
                    └── Job 2 (needs: Job 1)
                          └── Step 1: deploy
```

### 2.2 YAML Syntax & Workflow File Location

- All workflow files live in **`.github/workflows/`**.
- File must have a `.yml` or `.yaml` extension.
- A minimal workflow:

```yaml
name: CI                        # Display name in the Actions tab

on: [push]                      # Trigger event

jobs:
  build:                        # Job ID (arbitrary name)
    runs-on: ubuntu-latest      # Runner label
    steps:
      - uses: actions/checkout@v4
      - run: echo "Hello, GitHub Actions!"
```

### 2.3 Triggers / Events

Walk the audience through the most common event triggers:

| Trigger | Syntax | Use Case |
|---------|--------|----------|
| Push | `on: push` or `on: push: branches: [main]` | Run CI on every push |
| Pull Request | `on: pull_request` | Validate PRs before merge |
| Schedule (cron) | `on: schedule: - cron: '0 8 * * 1'` | Weekly cleanup, stale-issue bot |
| Manual dispatch | `on: workflow_dispatch` | On-demand deployments |
| Repository dispatch | `on: repository_dispatch` | Trigger from external systems via API |
| Release | `on: release: types: [published]` | Publish packages on release |

**Demo — add a `workflow_dispatch` trigger:**

```yaml
on:
  push:
    branches: [main]
  workflow_dispatch:          # adds a "Run workflow" button in the UI
```

Show the audience the **"Run workflow"** button in the Actions tab.

### 2.4 Runners: GitHub-Hosted vs. Self-Hosted

> **MS Learn module:** [Manage runners](https://learn.microsoft.com/en-us/training/modules/manage-github-actions-enterprise/manage-runners)

| Aspect | GitHub-Hosted | Self-Hosted |
|--------|---------------|-------------|
| Maintenance | Fully managed by GitHub | You manage OS, software, updates |
| Clean environment | Fresh VM every job | Persists state between jobs (unless you clean up) |
| Cost | Free minutes included in plan; per-minute after | Free with GitHub Actions; you pay infrastructure |
| Customization | Limited to available images | Full control: hardware, GPU, on-prem network |
| Labels | `ubuntu-latest`, `windows-latest`, `macos-latest` | Custom labels you define |

**When to use self-hosted:**
- Need access to on-premises resources behind a firewall.
- Require special hardware (GPU, ARM, large memory).
- Want to reduce build times with persistent caches.

---

## Section 3 — Workflow Syntax Essentials (30 min) | 🕑 3:40 PM – 4:10 PM

> **MS Learn module:** [Build continuous integration workflows by using GitHub Actions](https://learn.microsoft.com/en-us/training/modules/github-actions-ci/)

### 3.1 Key Keywords: `on`, `jobs`, `steps`, `runs-on`, `uses`, `run`

Walk through each keyword with this annotated example:

```yaml
name: Node.js CI

on:                              # 1. TRIGGER
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:                            # 2. JOBS
  test:                          # Job ID
    runs-on: ubuntu-latest       # 3. RUNNER

    steps:                       # 4. STEPS
      - uses: actions/checkout@v4          # 5. USES (reusable action)
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci                        # 6. RUN (shell command)
      - run: npm test
```

### 3.2 Matrix Strategies

Matrix strategies let you test across multiple OS + version combos in parallel.

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [18, 20, 22]
      fail-fast: false           # don't cancel all if one fails

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test
```

**Key points:**
- This creates **9 parallel jobs** (3 OS × 3 Node versions).
- `fail-fast: false` lets all combinations finish even if one fails.
- You can add `exclude:` and `include:` to fine-tune the matrix.

### 3.3 Conditional Execution with `if`

```yaml
steps:
  - run: echo "This runs only on main"
    if: github.ref == 'refs/heads/main'

  - run: echo "This runs only on PRs"
    if: github.event_name == 'pull_request'

  - run: echo "Previous step failed"
    if: failure()                 # runs even if a previous step failed
```

**Common `if` expressions:**
| Expression | Meaning |
|------------|---------|
| `success()` | Default — run if all previous steps succeeded |
| `failure()` | Run only if a previous step failed |
| `always()` | Run regardless of outcome |
| `cancelled()` | Run only if the workflow was cancelled |
| `github.event_name == 'push'` | Run only for push events |
| `contains(github.event.head_commit.message, '[skip ci]')` | Skip CI based on commit message |

### 3.4 Job Dependencies with `needs`

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building..."

  test:
    needs: build                  # waits for build to succeed
    runs-on: ubuntu-latest
    steps:
      - run: echo "Testing..."

  deploy:
    needs: [build, test]          # waits for BOTH build and test
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying..."
```

**Draw the dependency graph:**

```
build ──▶ test ──▶ deploy
  └─────────────────┘
```

---

## Section 4 — Secrets & Variables (20 min) | 🕑 4:10 PM – 4:30 PM

> **MS Learn module:** [Manage encrypted secrets](https://learn.microsoft.com/en-us/training/modules/manage-github-actions-enterprise/manage-encrypted-secrets)

### 4.1 Repository, Environment, and Organization Secrets

| Scope | Where to Create | Visibility |
|-------|----------------|------------|
| **Repository** | Repo → Settings → Secrets and variables → Actions | Only this repo's workflows |
| **Environment** | Repo → Settings → Environments → (env name) → Secrets | Only jobs targeting that environment |
| **Organization** | Org → Settings → Secrets and variables → Actions | All repos (or selected repos) in the org |

**Precedence:** Environment secrets > Repository secrets > Organization secrets (most specific wins).

**Live Demo — create a repository secret:**

1. Go to repo **Settings → Secrets and variables → Actions**.
2. Click **New repository secret**.
3. Name: `MY_TOKEN`, Value: `some-demo-value`.
4. Show that the value is **write-only** — you can never read it back from the UI.

### 4.2 Using `${{ secrets.MY_SECRET }}` Safely

```yaml
steps:
  - name: Deploy
    run: |
      curl -H "Authorization: token ${{ secrets.DEPLOY_TOKEN }}" \
           https://api.example.com/deploy
```

**Security rules to emphasize:**
- Secrets are **masked in logs** — GitHub replaces them with `***`.
- **Never** echo or print secrets directly; GitHub attempts to mask them, but structured data could leak.
- Secrets are **not passed to workflows triggered from forks** (prevents exfiltration in open-source PRs).
- Use **environment-scoped secrets** for production credentials to add an approval gate.

**Managing secrets via CLI:**

```bash
# Set a secret
gh secret set DEPLOY_TOKEN --body "ghp_xxxxxxxxxxxx"

# List secrets (shows names, not values)
gh secret list

# Delete a secret
gh secret delete DEPLOY_TOKEN
```

### 4.3 Environment Variables and `${{ vars.* }}`

**Configuration variables** (non-secret, plain-text values):

```yaml
env:                                 # Workflow-level env vars
  APP_ENV: production

jobs:
  deploy:
    runs-on: ubuntu-latest
    env:                             # Job-level env vars
      REGION: eastus
    steps:
      - run: echo "Deploying to $REGION"
      - run: echo "App name is ${{ vars.APP_NAME }}"  # Repository/org variable
```

| Feature | `env:` in YAML | `${{ vars.* }}` | `${{ secrets.* }}` |
|---------|---------------|-----------------|-------------------|
| Defined in | Workflow file | Repo/Org Settings | Repo/Org/Env Settings |
| Visible in logs | Yes | Yes | Masked (`***`) |
| Use case | Build-time constants | Config values (URLs, names) | Credentials, tokens |

---

## Section 5 — Environments & Deployments (20 min) | 🕑 4:30 PM – 4:50 PM

> **MS Learn module:** [Build and deploy applications to Azure by using GitHub Actions](https://learn.microsoft.com/en-us/training/modules/github-actions-cd/)

### 5.1 Environment Protection Rules & Required Reviewers

Environments let you model deployment targets (dev, staging, production) with built-in governance.

**Setup steps (Live Demo):**

1. Go to repo **Settings → Environments → New environment**.
2. Name it `production`.
3. Enable **Required reviewers** — add yourself or a team.
4. Optionally restrict to specific branches (e.g., only `main` can deploy to production).

**Reference the environment in your workflow:**

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://myapp.azurewebsites.net   # shown as a deployment link
    steps:
      - uses: actions/checkout@v4
      - run: echo "Deploying to production..."
```

When this job triggers, it **pauses** and waits for the required reviewer to approve.

### 5.2 Deployment Approvals and Wait Timers

| Protection Rule | Effect |
|----------------|--------|
| **Required reviewers** | Job pauses until 1+ designated reviewers approve (up to 6 reviewers). |
| **Wait timer** | Job is delayed by 0–43,200 minutes (up to 30 days) after trigger. |
| **Branch restrictions** | Only specific branches/tags can deploy to this environment. |
| **Custom deployment protection rules** | Integrate with external systems (e.g., ServiceNow, Jira) for approval. |

**Demo — show the approval flow:**

1. Push a change to `main` that triggers the deploy job.
2. Show the workflow run pausing at the deployment step with a **"Review deployments"** prompt.
3. Click **Approve and deploy** to continue.

### 5.3 Environment-Specific Secrets and Variables

Each environment can have its own secrets and variables that **override** repo-level ones:

```
Repository secret:  DATABASE_URL = "postgres://dev-db:5432/myapp"
Environment (production) secret:  DATABASE_URL = "postgres://prod-db:5432/myapp"
```

When the job targets the `production` environment, it automatically gets the production `DATABASE_URL`.

**Full CD workflow example tying it all together:**

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm test
      - uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/

  deploy-staging:
    needs: build-and-test
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.myapp.com
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-output
      - run: echo "Deploying to staging with ${{ secrets.DEPLOY_KEY }}"

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://myapp.com
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-output
      - run: echo "Deploying to production with ${{ secrets.DEPLOY_KEY }}"
```

**Pipeline flow:**

```
build-and-test ──▶ deploy-staging (auto) ──▶ deploy-production (requires approval)
```

---

## Bonus Topics (if time allows) | 🕑 4:50 PM+

### Caching Dependencies

> **MS Learn unit:** [Cache, share and debug workflows](https://learn.microsoft.com/en-us/training/modules/github-actions-ci/cache-share-debug-workflows)

```yaml
- name: Cache npm dependencies
  uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-npm-
```

### Workflow Artifacts

```yaml
# Upload
- uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: coverage/
    retention-days: 10

# Download in another job
- uses: actions/download-artifact@v4
  with:
    name: test-results
```

### Status Badges

Add to your README:

```markdown
![CI](https://github.com/OWNER/REPO/actions/workflows/ci.yml/badge.svg)
```

### Debug Logging

Set these **repository secrets** to enable verbose logs:
- `ACTIONS_RUNNER_DEBUG` = `true`
- `ACTIONS_STEP_DEBUG` = `true`

---

## Workshop Wrap-Up Checklist

- [ ] Attendees can explain what GitHub Actions is and its core components
- [ ] Attendees have created a basic workflow triggered by `push`
- [ ] Attendees understand matrix strategies and conditional execution
- [ ] Attendees know how to create and use secrets at repo/org/environment scope
- [ ] Attendees can configure environment protection rules with required reviewers
- [ ] Attendees can describe the difference between GitHub-hosted and self-hosted runners

## Additional MS Learn Modules for Self-Study

| Module | Link |
|--------|------|
| Automate development tasks by using GitHub Actions | [Learn](https://learn.microsoft.com/en-us/training/modules/github-actions-automate-tasks/) |
| Build CI workflows by using GitHub Actions | [Learn](https://learn.microsoft.com/en-us/training/modules/github-actions-ci/) |
| Build and deploy to Azure by using GitHub Actions | [Learn](https://learn.microsoft.com/en-us/training/modules/github-actions-cd/) |
| Automate GitHub by using GitHub Script | [Learn](https://learn.microsoft.com/en-us/training/modules/automate-github-using-github-script/) |
| Publish to GitHub Packages | [Learn](https://learn.microsoft.com/en-us/training/modules/github-actions-packages/) |
| Create and publish custom GitHub actions | [Learn](https://learn.microsoft.com/en-us/training/modules/create-custom-github-actions/) |
| Manage GitHub Actions in the enterprise | [Learn](https://learn.microsoft.com/en-us/training/modules/manage-github-actions-enterprise/) |
