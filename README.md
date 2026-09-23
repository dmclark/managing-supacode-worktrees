# Managing Supacode Worktrees

An Agent Skill that keeps Git worktrees synchronized with the Supacode
sidebar.

Supacode does not discover worktrees merely because they exist under
`~/.supacode/repos/`. Sidebar-visible worktrees must be created and managed
through the `supacode` CLI. This skill teaches agents—including Puck and
Amp—to use that workflow and verify both Supacode and Git state.

## What it does

- Creates sidebar-visible worktrees with `supacode repo worktree-new`.
- Registers each verified worktree with the canonical multi-directory Amp
  runner named `runner-1`.
- Directs Puck and other coordinators to create a new thread on `runner-1` with
  its `working_directory` set to the exact served worktree path.
- Checks out pull-request and feature branches without switching the primary
  repository checkout.
- Verifies registration through both `supacode worktree list` and
  `git worktree list`.
- Repairs clean worktrees that were created with plain Git and therefore do
  not appear in Supacode.
- Uses Supacode commands for archive, unarchive, and deletion.
- Protects dirty worktrees, unpushed commits, and non-equivalent branches from
  accidental deletion.

## Requirements

- [Supacode](https://supacode.sh/) with its `supacode` CLI on `PATH`.
- A Git repository registered with Supacode.
- A running Amp runner named `runner-1` with multi-directory support.
- An agent runtime that supports the Agent Skills directory convention.

Confirm both CLIs are available:

```bash
command -v supacode
supacode help
command -v amp
amp runner --help
```

## Install globally

Clone this repository into the global Agent Skills directory:

```bash
mkdir -p ~/.config/agents/skills
git clone \
  https://github.com/dmclark/managing-supacode-worktrees.git \
  ~/.config/agents/skills/managing-supacode-worktrees
```

Start a new agent session afterward so the skill registry reloads.

## Install for one project

Copy `SKILL.md` into the project's skill directory:

```text
<project>/.agents/skills/managing-supacode-worktrees/SKILL.md
```

Project-local installation is useful when every contributor should follow the
same Supacode worktree workflow. Global installation applies the workflow
across all repositories for one user.

## When it activates

The skill is designed to load for requests such as:

- “Create a Supacode worktree for this PR.”
- “Have Puck check out this branch locally.”
- “Why isn't this worktree in my Supacode sidebar?”
- “Archive or delete this Supacode worktree.”
- “Create a worktree under `~/.supacode`.”

## Core rule

For a worktree that should appear in Supacode, use:

```bash
supacode repo worktree-new \
  --repo "$repo_id" \
  --branch "$branch" \
  --base "$base" \
  --fetch \
  --name "$name" \
  --location "$location"
```

Do not substitute `git worktree add`. A plain Git worktree can be valid in
Git while remaining invisible to Supacode.

After Supacode and Git verification succeeds, the skill registers the exact
canonical path with `runner-1` and checks that `amp runner dirs list` reports
that path. It fails closed if the canonical runner is unavailable and never
starts a replacement runner.

Puck or another coordinator then creates a new thread on `runner-1` with
`working_directory` equal to the served worktree path. The workflow does not
open a destination Supacode tab, continue the source Amp thread, or submit
`/new` to another Amp session.

See [`SKILL.md`](./SKILL.md) for the complete workflow and safety rules.
