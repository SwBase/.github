# .github

Central repository for the **SwBase** organization. Holds shared issue
templates, label definitions and the org-wide automation workflows.

## Workflows

### Sync Issues To Checkout Development Project

`.github/workflows/sync-issues-to-checkout-development-project.yml`

Adds every issue across the whole SwBase org that carries the
`checkout-development` label to project **11** (*Checkout - Development*).
Because it searches org-wide it is not affected by the per-repo limit of the
built-in "Auto-add to project" workflow.

- **Add-only & idempotent** — already-added issues are not duplicated, and
  nothing is ever removed from the board.
- **Triggers**
  - `issues` (`opened`, `reopened`, `labeled`) — issues in this repo land on
    the board instantly.
  - `schedule` every 5 minutes — the org-wide safety net that picks up issues
    from all other repos (5 minutes is the lowest interval GitHub Actions
    allows).
  - `workflow_dispatch` — run manually to test.

### Sync Labels

`.github/workflows/sync-labels.yml`

Syncs the labels defined in `.github/labels.yml` to **all** non-archived repos
in the org. Create-or-update: existing labels are left untouched and nothing is
deleted.

- **Triggers**
  - `push` on `.github/labels.yml` or the workflow file itself.
  - `workflow_dispatch` — run manually.

## Running manually

Both workflows can be triggered by hand with the GitHub CLI (`workflow_dispatch`):

```bash
# Sync issues to the Checkout - Development project
gh workflow run sync-issues-to-checkout-development-project.yml --repo SwBase/.github

# Sync labels to all repos in the org
gh workflow run sync-labels.yml --repo SwBase/.github
```

Follow a run afterwards with `gh run watch --repo SwBase/.github`.

## Authentication — GitHub App

Both workflows authenticate through a single GitHub App (not tied to any one
user). The app issues a short-lived token per run via
`actions/create-github-app-token`. Set it up once:

### 1. Create the GitHub App

1. Go to https://github.com/organizations/SwBase/settings/apps → **New GitHub
   App** (org level, not your personal settings).
2. Fill in:
   - **GitHub App name**: e.g. `SwBase Project Sync`
   - **Homepage URL**: any URL (required field, content does not matter)
   - **Webhook**: turn **Active** OFF (not needed)
3. Under **Permissions** set exactly these rights:
   - Repository permissions → **Issues: Read and write** (labels fall under
     Issues; also covers reading issues for the project sync)
   - Repository permissions → **Metadata: Read-only** (usually on by default,
     needed for search and listing repos)
   - Organization permissions → **Projects: Read and write**
4. **Where can this app be installed?**: *Only on this account*.
5. **Create GitHub App**.

### 2. Get the credentials

On the app page after creating it:

1. Note the **App ID** (at the top).
2. Scroll to **Private keys** → **Generate a private key**. A `.pem` file is
   downloaded — store it safely, you only see it once.

### 3. Install the app on the org

1. On the app page, left menu → **Install App** → click **Install** for
   `SwBase`.
2. Choose **All repositories**.

### 4. Set the secrets

Add two org secrets at
https://github.com/organizations/SwBase/settings/secrets/actions:

- **`APP_ID`** — the App ID from step 2
- **`APP_PRIVATE_KEY`** — the full contents of the `.pem` file (including the
  `-----BEGIN...` / `-----END...` lines)

That's it — both workflows will use these secrets automatically.
