# CoAgent Skill

A [Claude Code](https://claude.com/claude-code) skill that sets up [CoAgent](https://resilient-froyo-83213f.netlify.app) design feedback collection on any project in one command.

## What it does

Running `/coagent` in Claude Code will:

1. Open your browser to authenticate with CoAgent
2. Create a new feedback project
3. Install [Agentation](https://www.agentation.com/) (visual annotation toolbar)
4. Wire up the webhook so annotations flow into your CoAgent dashboard in real-time

No environment variables or manual configuration needed.

## Install

```bash
npx skills add faisal/coagent-skill
```

## Usage

From any project in Claude Code:

```
/coagent
```

Or pass a project name:

```
/coagent My Prototype
```

## Prerequisites

- [Node.js](https://nodejs.org/) (for the auth flow and Agentation)
- A [CoAgent](https://resilient-froyo-83213f.netlify.app) account
- A React, Next.js, or Vite project

## How it works

```
/coagent
  │
  ├─ Opens browser → CoAgent login
  ├─ You sign in & click "Authorize"
  ├─ CLI receives auth token
  ├─ Creates a project via Supabase API
  ├─ Installs agentation package
  ├─ Adds <Agentation webhookUrl="..." /> to your app
  │
  └─ Done — annotations flow to your CoAgent dashboard
```

## What is CoAgent?

CoAgent is a design prototype feedback platform. Designers create prototype sites, PMs leave feedback via the Agentation toolbar, and CoAgent captures those annotations into a central dashboard with real-time updates, filtering, threading, and status tracking.

## License

MIT
