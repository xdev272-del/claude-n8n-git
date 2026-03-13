# GitHub + n8n Webhook Setup Guide

This guide walks you through connecting a GitHub repository to the **GitHub Push to Main** n8n workflow so that every push to `main` triggers the automation.

---

## Prerequisites

- A running n8n instance (self-hosted or n8n Cloud)
- A GitHub account with admin access to the target repository
- A GitHub Personal Access Token (PAT) **or** OAuth App credentials

---

## Step 1: Create a GitHub Personal Access Token

1. Go to **GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)**.
2. Click **Generate new token (classic)**.
3. Give it a descriptive name (e.g., `n8n-webhook`).
4. Select the following scopes:
   - `repo` (full control of private repositories)
   - `admin:repo_hook` (read/write access to repository hooks)
5. Click **Generate token** and **copy the token immediately** — you won't see it again.

---

## Step 2: Add GitHub Credentials in n8n

1. Open your n8n instance.
2. Go to **Settings → Credentials → Add Credential**.
3. Search for **GitHub API** and select it.
4. Fill in the fields:
   - **User**: Your GitHub username
   - **Access Token**: Paste the PAT you generated in Step 1
5. Click **Save**. The credential will be named "GitHub Account" by default — this matches the workflow JSON.

---

## Step 3: Import the Workflow

1. In n8n, go to **Workflows → Add Workflow → Import from File**.
2. Select `github-push-workflow.json` from this directory.
3. The workflow will load with three nodes:
   - **GitHub Trigger** — listens for push events
   - **Extract Commit Data** — pulls out the commit message, author, and repo URL
   - **Webhook Response** — returns the formatted data
4. Open the **GitHub Trigger** node and configure:
   - **Repository Owner**: Your GitHub username or organization name
   - **Repository Name**: The target repository name
   - **Credential**: Select the "GitHub Account" credential from Step 2

---

## Step 4: Activate the Workflow

1. Toggle the **Active** switch in the top-right corner of the workflow editor.
2. n8n will automatically register a webhook in your GitHub repository via the API.
3. You can verify this by going to your GitHub repo → **Settings → Webhooks**. You should see a new webhook pointing to your n8n instance URL (e.g., `https://your-n8n-domain.com/webhook/github-push-webhook`).

> **Note:** If you are running n8n locally (e.g., `http://localhost:5678`), GitHub cannot reach your instance. You will need to use a tunnel service like [ngrok](https://ngrok.com) or [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/) to expose your local n8n to the internet.

---

## Step 5: Manual Webhook Setup (Alternative)

If n8n does not auto-register the webhook (e.g., due to permission issues), set it up manually:

1. Go to your GitHub repository → **Settings → Webhooks → Add webhook**.
2. Configure the webhook:
   - **Payload URL**: Your n8n webhook URL, e.g., `https://your-n8n-domain.com/webhook/github-push-webhook`
   - **Content type**: `application/json`
   - **Secret**: Leave blank (unless you configure a secret in n8n)
   - **Which events?**: Select **Just the push event**
3. Click **Add webhook**.

---

## Step 6: Test the Integration

1. Make a small change in your repository and push to `main`:
   ```bash
   echo "test" >> test.txt
   git add test.txt
   git commit -m "Test n8n webhook integration"
   git push origin main
   ```
2. In n8n, go to **Executions** to see the workflow run.
3. The execution output should show:
   ```json
   {
     "commitMessage": "Test n8n webhook integration",
     "authorName": "Your Name",
     "repositoryUrl": "https://github.com/you/your-repo"
   }
   ```

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Webhook not firing | Check GitHub → Settings → Webhooks → Recent Deliveries for error codes |
| 401 Unauthorized | Re-check that your PAT has `repo` and `admin:repo_hook` scopes |
| n8n not reachable | If self-hosted locally, use ngrok or a tunnel to expose your instance |
| Workflow not triggering | Ensure the workflow is **Active** (toggled on) in n8n |
| Wrong branch triggers | The GitHub Trigger sends all push events; add an IF node after the trigger to filter `$json.ref === "refs/heads/main"` if needed |
