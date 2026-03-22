---
name: coagent
description: Set up CoAgent feedback collection on any project. Opens a browser for authentication, creates a project, installs Agentation, and wires up the webhook.
disable-model-invocation: true
argument-hint: [project-name]
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

# CoAgent Setup Skill

Set up CoAgent design feedback collection on the current project.

The user may pass a project name as `$ARGUMENTS`. If not provided, use the current directory name.

**Dashboard URL:** `https://resilient-froyo-83213f.netlify.app`

## Step 1: Browser-based authentication

Start a temporary local HTTP server and open the browser for the user to authenticate:

```bash
node -e "
const http = require('http');
const { exec } = require('child_process');
const timer = setTimeout(() => { console.error('Timed out waiting for auth'); process.exit(1); }, 120000);
const server = http.createServer((req, res) => {
  if (req.url.startsWith('/callback')) {
    const params = Object.fromEntries(new URL(req.url, 'http://localhost').searchParams);
    res.writeHead(200, { 'Access-Control-Allow-Origin': '*' });
    res.end('ok');
    clearTimeout(timer);
    server.close(() => process.exit(0));
    console.log(JSON.stringify(params));
  }
});
server.listen(0, () => {
  const port = server.address().port;
  const url = 'https://resilient-froyo-83213f.netlify.app/cli-auth?port=' + port;
  console.error('Open this URL to authenticate: ' + url);
  exec(process.platform === 'darwin' ? 'open \"' + url + '\"' : 'xdg-open \"' + url + '\"');
});
"
```

The script outputs JSON to stdout with `access_token`, `user_id`, `supabase_url`, and `publishable_key`. Parse these for subsequent steps.

Tell the user: "Opening your browser to authenticate with CoAgent. Please sign in and click Authorize."

## Step 2: Create a project

Use the auth token to create a new project via the Supabase REST API:

```bash
curl -s -X POST "$SUPABASE_URL/rest/v1/projects" \
  -H "apikey: $PUBLISHABLE_KEY" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -H "Prefer: return=representation" \
  -d '{"name": "PROJECT_NAME", "owner_id": "USER_ID"}'
```

Extract `id` and `webhook_secret` from the response.

## Step 3: Build the webhook URL

```
$SUPABASE_URL/functions/v1/webhook-receiver?project_id=$PROJECT_ID&secret=$WEBHOOK_SECRET
```

## Step 4: Set up Agentation

Check if Agentation is already installed and set up in the project:

- If an `<Agentation` component already exists in the codebase, just add or update the `webhookUrl` prop on it with the URL from Step 3. Done.
- If Agentation is not set up yet:
  1. Detect the package manager (`yarn.lock` → yarn, `pnpm-lock.yaml` → pnpm, `bun.lockb` → bun, otherwise npm). Install `agentation` as a dev dependency.
  2. Find the root component (`src/App.tsx`, `src/app/layout.tsx`, `app/layout.tsx`, `src/pages/_app.tsx`, etc.)
  3. Add `import { Agentation } from "agentation"` at the top
  4. Add `<Agentation webhookUrl="WEBHOOK_URL" />` as the last child inside the returned JSX, replacing `WEBHOOK_URL` with the actual URL from Step 3.
  5. If the root component can't be found, ask the user which file to modify.

## Step 5: Report success

Print a summary:
- Project name created in CoAgent
- Which file was modified
- Remind the user to start their dev server to see the Agentation toolbar
- Link to view feedback: `https://resilient-froyo-83213f.netlify.app/projects/$PROJECT_ID`

## Error Handling

- If the auth server times out (2 min): Tell the user and suggest retrying
- If project creation fails: Show the error from Supabase
- If Agentation is already fully configured with a webhookUrl: Just report the dashboard link, no changes needed
