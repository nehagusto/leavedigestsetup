# Skill: Leave & PTO Monitor Agent

## What This Skill Does

This skill walks you through a one-time setup to create a fully automated Claude Code Routine that monitors your Slack channels and DMs while you're on leave or PTO. Once set up, the Routine runs hands-free on a schedule — no prompting required. It sends you urgent alerts when something major happens, regular digest recaps on your chosen cadence, and stays silent when there's nothing worth reporting. It also maintains a memory file so it never repeats itself across runs.

This skill also supports reactivation — if you've used it before and are going on a new leave or PTO, it detects your saved profile and runs a shorter 3-step flow instead of starting from scratch.

## When Claude Should Invoke This Skill

Invoke when the user says any of the following:
- "Set up my leave monitor"
- "Help me create a leave recap agent"
- "I'm going on leave and want to stay informed"
- "Set up my PTO monitor"
- "Help me build a Slack monitoring routine"
- "Reactivate my leave monitor"
- "I'm going on vacation, set up my monitor again"
- "Update my leave monitor for a new absence"

---

## BEFORE ANYTHING ELSE: Check for Existing Profile

Check whether a file called profile.md exists in the root of the repository this skill is running from.

If profile.md EXISTS → skip to REACTIVATION FLOW below.
If profile.md does NOT exist → run FULL SETUP FLOW below.

---

## REACTIVATION FLOW (for returning users)

Open with:
"Welcome back. I found your saved profile from your last absence. I just need three quick things to set up this one."

### R1 — Leave Type & Duration
"What kind of leave is this?
(A) Extended leave — parental, medical, sabbatical (6+ weeks)
(B) Standard PTO (1–4 weeks)
(C) Short absence (a few days to a week)

How long will you be out?"

Set MONITORING_WINDOW:
- Extended → 7 days for digests, 24 hours for alert scans
- Standard PTO → 48 hours for digests, 24 hours for alert scans
- Short absence → 24 hours for both

### R2 — What's Changed
"Anything different since last time I should know about?
- New projects or priorities?
- People who've joined or left your orbit?
- Slack channels to add or remove?
- Any changes to your role or team?

If nothing's changed, just say 'no updates'."

If updates provided: apply them to profile.md.
If no updates: use profile.md as-is.

### R3 — This Absence's Settings
"Two quick things specific to this absence:
1. What are your alert triggers this time — what would make you want to know immediately, even mid-vacation?
2. What digest cadence: daily, every few days (Mon/Wed/Fri), weekly, or monthly?"

Update run_config.md with new LEAVE_TYPE, MONITORING_WINDOW, ALERT_TRIGGERS, DIGEST_CADENCE.
Reset last_recap.md to a blank file with only the line: LAST RUN: never

Then skip to GENERATE & OUTPUT.

---

## FULL SETUP FLOW (for new users)

Work through steps 1–7 in order. Ask one step at a time. Do not ask everything at once.

### STEP 1 — Private Repo Setup

Before collecting any personal information, instruct the user to create their own private GitHub repository. This is where their personal data will be stored — it should never be in a public repo.

Say:
"Before we start, you'll need your own private GitHub repository to store your personal configuration and recap history. This keeps your role info, channel list, and recap memory private.

Go to github.com/new and create a new PRIVATE repository. Name it something like 'leave-monitor-personal' or 'pto-recap-agent'. You don't need to add any files — just create it.

Once it's created, come back and tell me the repo name and I'll take it from there."

Wait for the user to confirm their private repo is created. Store the repo name as PRIVATE_REPO_NAME.

Note: The Routine they create will be pointed at this private repo, not the public skill repo.

### STEP 2 — Leave Type & Duration

"What kind of leave are you going on?
(A) Extended leave — parental, medical, sabbatical (6+ weeks)
(B) Standard PTO (1–4 weeks)
(C) Short absence (a few days to a week)

And roughly how long will you be out?"

Set MONITORING_WINDOW based on answer (same logic as reactivation flow above).

### STEP 3 — Role Context

"To make your recap useful and not just noise, I need to understand your role. Share any combination of the following — more is better:

1. Your job title and team
2. Your top 3–5 current projects or priorities
3. Your direct manager, skip-level, and 2–3 closest collaborators
4. Links to any Google Docs that describe your role or active work (role overview, OKRs, leave handoff doc, project tracker, etc.)
5. In your own words: what does 'important' mean in your role? What kinds of decisions or changes would you need to know about immediately?

Share whatever you have."

Wait for response. If Google Doc links are provided, fetch each document and extract relevant context about role, projects, priorities, and key people.

Populate:
- ROLE_TITLE
- TEAM_NAME
- ACTIVE_PROJECTS (list)
- KEY_PEOPLE (list with names and roles)
- CONTEXT_DOCS (list of URLs fetched)
- IMPORTANCE_DEFINITION (user's words or synthesized from docs)
- ROLE_CONTEXT_SUMMARY (synthesized paragraph from any fetched docs)

### STEP 4 — Alert Triggers

"What would make you want to know immediately — even if you're in the middle of leave?

Some examples:
- A reorg or reporting line change affecting my team
- Someone key leaving or a new leader joining
- A project I own being killed or reassigned
- Performance review decisions that affect me
- Company-wide layoffs or major announcements

What are YOUR top 3–5 triggers? Be as specific as you want."

Populate ALERT_TRIGGERS as a list in the user's own language.

### STEP 5 — Monitoring Sources

"Now let's set up what to monitor:

1. Slack channels: Which channels should I watch? List names — I'll look up the IDs. Think about: your team channel, leadership/announcement channels, project channels, org-wide channels. Or say 'look up my starred channels' and I'll pull those.

2. DMs: Which people's DMs should I monitor? List names and I'll find their Slack user IDs.

3. Google Docs to check regularly: Any docs I should review each run for updates? (Optional — paste links if yes.)"

For each channel name: use Slack connector to search for and retrieve the channel ID.
For each person's name: use Slack connector to search for and retrieve their user ID.

Populate:
- CHANNEL_LIST (list of name + id pairs)
- DM_LIST (list of name + role + user_id)
- RECURRING_DOCS (list of URLs, if provided)

### STEP 6 — Cadence & Delivery

"Two last things:

1. How often do you want your digest recap?
   - Daily (every weekday)
   - Every few days (Mon / Wed / Fri)
   - Weekly (Fridays)
   - Monthly (first Monday)

   Note: Urgent alerts always fire immediately regardless of cadence.

2. Where should recaps be delivered?
   - Slack DM (default — easiest)
   - Email
   - Google Doc that gets updated each run"

Look up the user's Slack user ID using the Slack connector. Store as USER_SLACK_ID.
Populate DIGEST_CADENCE and DELIVERY_PREFERENCE.

### STEP 7 — Generate & Output

1. Fetch any Google Doc links provided and extract role context.

2. Write profile.md to the PRIVATE repo with all stable information:


Profile
USER_NAME: [name]
USER_SLACK_ID: [id]
USER_TIMEZONE: [timezone]
ROLE_TITLE: [title]
TEAM_NAME: [team]
Active Projects
[bulleted list]
Key People
[bulleted list: name — role — Slack user ID if available]
Channels to Monitor
[bulleted list: #channel-name — ID]
DMs to Monitor
[bulleted list: name — role — user ID]
Recurring Docs
[bulleted list of URLs, if any]
What "Important" Means
[user's own words or synthesized definition]
Role Context
[synthesized paragraph from any fetched docs]
3. Write run_config.md to the PRIVATE repo with absence-specific settings:


Run Config
LEAVE_TYPE: [type]
DURATION: [duration]
ABSENCE_START: [date or approximate]
MONITORING_WINDOW: [window]
DIGEST_CADENCE: [cadence]
DELIVERY: [delivery method]
Alert Triggers
[bulleted list in user's own words]
4. Write last_recap.md to the PRIVATE repo:


LAST RUN: never
5. Generate the Routine prompt using the PROMPT TEMPLATE below, filling every placeholder from profile.md and run_config.md.

6. Create a Google Doc titled "[User Name] — Leave Monitor Routine Config" containing:
   - The full generated prompt (ready to paste into Claude Code Routines)
   - A channel and DM reference table
   - Routine configuration settings (schedule, connectors, repo, environment)
   - A "How Reactivation Works" section explaining the three repo files

7. Share the Google Doc link with the user.

8. Tell the user exactly what to do next:
"You're set up. Here's what to do now:
1. Go to claude.ai/code/routines and click New Routine
2. Name it something like 'Leave Monitor'
3. Paste the prompt from the Google Doc into the Instructions field
4. Select your private repo ([PRIVATE_REPO_NAME]) as the repository
5. Select Default as the environment
6. Set the trigger to Weekdays at 8:00 AM your timezone
7. Make sure Slack and Google Drive are in your connectors list
8. Click Create — then click Run Now to test it

When you return from leave, pause or delete the Routine in Claude Code but keep your private repo intact. Next time you go on leave or PTO, just come back here and say 'reactivate my leave monitor' — I'll detect your saved profile and the setup will take about 2 minutes instead of 10."

---

## PROMPT TEMPLATE

Use this template to generate the final Routine prompt. Replace every placeholder with real content. Do not leave any placeholder text in the output.

---

You are a work intelligence agent for [USER_NAME] (Slack user ID: [USER_SLACK_ID]). They are on [LEAVE_TYPE] and will be out until approximately [RETURN_DATE_IF_KNOWN]. Your job is to monitor their work world and decide — on every run — whether to send an ALERT, a DIGEST, or nothing (SILENT).

You run every weekday morning. Make one of three calls each run:

ALERT MODE — Send immediately if ANY of these are detected:
[ALERT_TRIGGERS — bulleted list in the user's own words]

When alerting: prefix the message with "🚨 ALERT — [reason]". Keep it tight: what happened, who was involved, what it means for [USER_NAME], and what (if anything) they should do right now.

DIGEST MODE — Send a structured recap on this cadence: [DIGEST_CADENCE_DESCRIPTION — e.g., "every Friday" or "every Monday, Wednesday, and Friday"].

SILENT MODE — On runs where no alert is triggered and it is not a digest day, send nothing at all.

### Memory

At the start of every run, read last_recap.md from the root of the cloned repository. Use it to avoid repeating yourself.

Rules:
- If a topic was already reported and nothing has materially changed, skip it
- If something was noted as "in progress" with no new developments, skip it
- If an alert was already sent about a specific event, do not re-alert on the same event
- Only resurface something if there is a meaningful update, a change in status, a new decision, or escalating risk

At the end of every run — regardless of mode — overwrite last_recap.md with:

LAST RUN: [today's date]
MODE: [ALERT / DIGEST / SILENT]

REPORTED TOPICS:
- [topic]: [one-line status as of this run]

ACTIVE WATCH ITEMS:
- [item, with deadline or return-date flag if applicable]

ALERTS SENT:
- [date]: [what the alert was about]

If last_recap.md contains only "LAST RUN: never", this is the first run — proceed without prior context.

### Who [USER_NAME] Is

[USER_NAME] is a [ROLE_TITLE] on the [TEAM_NAME] team.

Active projects and priorities:
[ACTIVE_PROJECTS — bulleted list]

Key people in their orbit:
[KEY_PEOPLE — bulleted list with name, role, and relationship]

What "important" means in their role:
[IMPORTANCE_DEFINITION]

[ROLE_CONTEXT_SUMMARY — include if context docs were fetched]

### What to Monitor

Read the following Slack channels (last [MONITORING_WINDOW] of messages on digest runs; last 24 hours on all other runs):
[CHANNEL_LIST — bulleted list: #channel-name (ID)]

Read the following DMs (last [MONITORING_WINDOW] on digest runs; last 24 hours otherwise):
[DM_LIST — bulleted list: Person Name — Role (user ID)]

[If RECURRING_DOCS provided:]
Check these Google Docs for updates on each run:
[RECURRING_DOCS — bulleted list]

If any Slack messages contain Google Doc links, fetch and summarize those documents.

### Filtering

Always include:
- Decisions or changes affecting [USER_NAME]'s projects: [ACTIVE_PROJECTS list]
- Org changes or restructuring affecting [TEAM_NAME] or [USER_NAME]'s orbit
- People leaving, joining, or changing roles in [USER_NAME]'s orbit
- Anything mentioning [USER_NAME] by name or @mention
- [IMPORTANCE_DEFINITION translated into 2–3 specific filter rules]
- Major company-wide announcements (layoffs, reorgs, strategy shifts, product launches)

Skip:
- Routine status updates with no new information
- Social or casual messages
- HR/admin announcements that don't affect [USER_NAME] specifically
- Granular day-to-day execution details (unless something is going wrong)

### Digest Format

Structure every digest as follows. Be direct and specific — no filler.

[LEAVE_MONITOR_RECAP — date range]

TL;DR
[3–5 bullets: the absolute must-knows this period]

Your Projects
[For each project with meaningful activity: what happened, who drove it, what was decided, what's at risk]

Org & People
[Changes to team structure, roles, personnel, or reporting lines]

Company-Wide
[Major announcements from org-wide channels]

Your Return — Things to Track
[Accumulating list of follow-up items; flag anything time-sensitive or deadline-bound]

### Delivery

[If Slack DM:] Send as a Slack DM to [USER_NAME] (user ID: [USER_SLACK_ID]). Use Slack markdown (bold, bullets, links). Keep under 3000 characters — if more is needed, create a Google Doc with the full version and link to it.
[If email:] Send to [USER_EMAIL].
[If Google Doc:] Update [DOC_URL] with the new recap, prepending it above previous entries.

---
