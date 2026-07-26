---
name: managing-supacode-worktrees
description: "Creates, reviews, archives, and deletes Supacode-managed Git worktrees through the supacode CLI so the sidebar stays synchronized. Use when a user asks Puck or Supacode to create a worktree, check out a PR or branch locally, manage a worktree under ~/.supacode, or fix a worktree missing from the Supacode sidebar."
compatibility: "Requires the Supacode CLI and a repository registered with Supacode."
---

# Managing Supacode worktrees

Use Supacode's CLI as the source of truth for sidebar-visible worktrees.
Putting a plain Git worktree under `~/.supacode/repos/` does not register it
with Supacode.

## Hard rules

- For a worktree that should appear in Supacode, use
  `supacode repo worktree-new`; do **not** use `git worktree add`.
- Do not edit `~/.supacode/sidebar.json`, `layouts.json`, or other Supacode
  state by hand.
- Use `supacode worktree archive`, `unarchive`, or `delete` for registered
  worktrees so Git and sidebar state stay synchronized.
- Before replacing or deleting any worktree, verify it is clean and check for
  unpushed commits. Ask before deleting dirty work, an unmerged branch, or any
  branch whose remote equivalence is not proven.
- Keep the repository's primary/runtime checkout untouched unless the user
  explicitly asks to change it.

## Discover the repository

1. Confirm the CLI is available:

   ```bash
   command -v supacode
   supacode --version
   ```

2. List Supacode repository IDs:

   ```bash
   supacode repo list
   ```

   Repository and worktree IDs are URL-encoded absolute paths with a trailing
   slash. Prefer `$SUPACODE_REPO_ID` when running in a Supacode terminal;
   otherwise select the ID whose decoded path matches the primary repository
   root.

3. Inspect existing Supacode and Git worktrees before choosing a name:

   ```bash
   supacode worktree list
   git worktree list
   ```

## Create a sidebar-visible worktree

Choose:

- `repo_id`: `$SUPACODE_REPO_ID` or the encoded ID from `supacode repo list`;
- `branch`: the local branch name Supacode should create;
- `base`: normally `origin/<remote-branch>` for PR review or the integration
  branch for new work;
- `name`: a short folder name such as `pr-16-review`;
- `location`: normally `~/.supacode/repos/<repository-name>`.

Then run:

```bash
supacode repo worktree-new \
  --repo "$repo_id" \
  --branch "$branch" \
  --base "$base" \
  --fetch \
  --name "$name" \
  --location "$location" \
  --timeout 180
```

For PR review, base the worktree on the PR's remote head and confirm the
resulting local branch tracks that remote branch. Never switch the primary
checkout merely to review a PR.

Supacode requires the requested local branch name to be available. If a local
branch already exists, do not delete it reflexively. First prove that it is not
checked out, has no unique commits, and exactly matches the intended remote
tip. Delete and let Supacode recreate it only when the user has authorized that
replacement; otherwise choose a distinct review branch name.

## Verify registration

Successful filesystem creation is not sufficient. Verify all three layers:

```bash
supacode worktree list
git worktree list
git -C "$worktree_path" status --short --branch
```

The new encoded worktree ID must appear in `supacode worktree list`, the path
must appear in `git worktree list`, and the worktree must be on the intended
branch/commit. Registered Supacode worktrees normally appear as locked in
`git worktree list`.

Optionally focus the new sidebar entry:

```bash
supacode worktree focus --worktree "$worktree_id"
```

Report the path, branch, commit, tracking remote, cleanliness, and confirmation
that the primary checkout was unchanged.

## Repair an unregistered plain Git worktree

If a worktree exists under `~/.supacode/repos/` but is absent from
`supacode worktree list`, it was likely created with plain Git.

1. Check cleanliness, branch tracking, local/remote tip equality, and unpushed
   commits.
2. Obtain approval before removing it or deleting its local branch.
3. Remove the unregistered worktree with `git worktree remove`; Supacode cannot
   manage an entry it does not know about.
4. If Supacode reports that the branch name is unavailable, delete the
   redundant local branch only after proving it exactly matches the remote and
   replacement is authorized.
5. Recreate it with `supacode repo worktree-new`.
6. Verify it appears in both `supacode worktree list` and `git worktree list`.

## Archive or delete

Use the encoded worktree ID:

```bash
supacode worktree archive --worktree "$worktree_id"
supacode worktree unarchive --worktree "$worktree_id"
supacode worktree delete --worktree "$worktree_id"
```

Deletion may also delete the local branch according to the user's Supacode
settings. Treat deletion as destructive: inspect status and commits first, and
obtain explicit approval.
