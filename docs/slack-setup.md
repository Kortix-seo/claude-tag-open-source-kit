# Slack setup: from a manifest to a thread

Four steps, in the order the product expects them. The install takes a few
minutes; the grants in `kortix.yaml` are what decide what happens after.

## 1. Ship the project

```bash
curl -fsSL https://kortix.com/install | bash
kortix init claude-tag-kit        # or clone this repo; the manifest is already here
kortix validate
./scripts/check-grants.sh
kortix ship
```

`kortix ship` pushes the manifest, and the project reads it from its default
branch. `kortix validate` is the same check the change request merge runs, so an
error you see here is an error that would have blocked the merge.

If you keep the project in your own git remote, point Kortix at it and push as
usual; the manifest still only takes effect on the default branch.

## 2. Connect Slack

Open **Customize → Connectors** in the project, select the **Channels** scope
and choose **Add to Slack**. Pick the workspace, allow the install, and the
workspace is connected.

On a self-hosted instance, set `SLACK_CLIENT_ID`, `SLACK_CLIENT_SECRET` and
`SLACK_SIGNING_SECRET` and the same button installs your own Slack app. Without
all three, each project connects its own Slack app.

A connected workspace gives every channel its own settings: the agent that runs
in it, the model a new thread starts on, and the join policy that decides who
may take part.

## 3. Let people in

Invite the bot to the channel. Then every person who will use it sends
`/kortix login` once, opens the link, and signs in. That link is private, works
once, and expires in 10 minutes. A message sent before linking runs when the
person links within those 10 minutes.

- An unlinked sender gets a connect prompt, not a run.
- A linked member without access to the project gets a request-access card that
  account admins see in Kortix.

A workspace that ran the Claude in Slack app re-binds the channels it sat in;
[moving a team off the old Claude in Slack app](https://claudetagalternative.com/claude-in-slack)
covers what changed and what an admin re-checks.

## 4. Tag it with a task

Mention the bot in the channel with the task. In a channel only a mention
reaches the session; a reply without one stays between people, and the agent
reads the thread the next time someone mentions it. In a DM with the bot every
message reaches the session.

One Slack thread is one session. The agent works on its own cloud computer and
replies in that thread, with live progress and a **Stop** button. Later messages
in the thread continue the same session, also after the sandbox has stopped and
resumed.

## The settings that matter after install

| Setting | Where it lives | What it decides |
| --- | --- | --- |
| Agent | the channel, in the Channels scope | Which agent runs there; `default_agent` otherwise |
| Model | the channel, or the session's composer | Which model the next thread starts on |
| Join policy | the channel | Who may join a session that started there |

Join policy has three values. Open, the default, admits any linked project
member. Approval has the session owner approve or deny each new person. Owner
only keeps a session to the person who started it.

Model choice is where an open-source stack earns its keep. Pin one on the agent
in `kortix.yaml` with its `model` key, which takes `provider/model`, or choose a
model in the composer for a single session.

## What the agent may read and post

All projects in one Slack workspace share one bot, and Kortix keeps each
project's agent inside that project's own conversations: a channel or DM bound
to the project, and threads a session of the project started or joined. Anything
else is refused with `conversation_not_in_project`. Slack channel arguments are
ids such as `C0123ABCD`, not names; `kortix connectors show kortix_slack.<action>`
prints the schema before you call one.

## When something does not arrive

| Symptom | Check |
| --- | --- |
| No reply in the thread | Is the bot in the channel? Was the message a mention? A reply without one stays between people. |
| "Link your account" prompt | Send `/kortix login` in that workspace and open the link within 10 minutes. |
| A connector call sits at pending | The approval card in the thread, or **Review → Needs you**. Approve or deny; a retry does nothing. |
| A call is refused with `conversation_not_in_project` | The channel is bound to another project on the same workspace; the first message in a new channel shows a project picker. |
| Trigger digest never posts | The channel id in the `morning-digest` prompt is a placeholder; set it to the ops channel's id. |
