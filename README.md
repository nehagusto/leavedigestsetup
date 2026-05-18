# Leave & PTO Monitor Agent

A Claude Code Skill that sets up a fully automated Slack monitoring Routine for anyone going on leave or PTO.

## What It Does

Once set up, the Routine runs hands-free every weekday on Anthropic's cloud infrastructure — no prompting needed. It:

- Reads your Slack channels and DMs
- Filters for what's actually relevant to your role
- Sends you **🚨 immediate alerts** for major events (reorgs, role changes, layoffs, project decisions)
- Sends **📋 digest recaps** on your chosen cadence (daily, a few times a week, weekly, or monthly)
- Stays **🔇 silent** when nothing notable happened
- Maintains a memory file so it never repeats itself run-to-run

## How to Use

### First-time setup (~10 minutes)

1. Open Claude Code (terminal or web at claude.ai/code)
2. Clone this repo or connect to it
3. Say: **"Set up my leave monitor"**
4. The wizard walks you through setup — you'll need:
   - A private GitHub repo you create for your personal data (the wizard will prompt you to create one)
   - Your Slack channels and key contacts
   - Any Google Docs describing your role or priorities (optional but improves quality)
5. At the end you'll receive a Google Doc with your ready-to-paste Routine prompt and setup instructions

### Reactivating for a future absence (~2 minutes)

When you return from leave, **pause or delete the Routine** in Claude Code but **keep your private repo**.

Next time you go on leave or PTO:
1. Open Claude Code and connect to this skill repo
2. Say: **"Reactivate my leave monitor"**
3. The wizard detects your saved profile and asks only 3 questions: leave type, what's changed, and cadence for this absence
4. You get a new Routine prompt in minutes

## What You'll Need

- Claude Code access (Pro, Max, Team, or Enterprise plan)
- A private GitHub repo for your personal data (the wizard will ask you to create one)
- Slack connected as a connector in Claude Code
- Google Drive connected as a connector (optional but recommended)

## Privacy

This public repo contains only the skill wizard — no personal data.

Your personal information (role, channels, people, recap history) is stored in **your own private repo** that you create during setup. It is never stored in this repo and is only visible to you.

## Files in This Repo

- `SKILL.md` — the wizard Claude Code follows to run setup. Do not edit.
- `README.md` — this file.

## Files in Your Private Repo (created during setup)

- `profile.md` — your stable profile: role, people, channels. Reused across absences. Update anytime your role changes.
- `run_config.md` — settings for the current absence: leave type, duration, alert triggers, cadence. Updated each reactivation.
- `last_recap.md` — the agent's memory. Auto-managed. Reset at each reactivation.
