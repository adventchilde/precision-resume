### Configure the upstream branch (one‑time setup)

```bash
git branch --set-upstream-to=origin/master master
```

This tells Git which remote branch to track, so future `git pull` commands know where to fetch from.

### 2. Enable automatic fetching in VS Code

VS Code can periodically fetch remote changes:

1. Open __Settings__ (`Ctrl+,`).
2. Search for __Git: Auto Fetch__ and enable it.
3. Optionally set __Git: Auto Fetch Interval__ (default is 5 minutes).

VS Code will now fetch remote updates in the background, showing a badge when you’re behind.

### 3. Use a Git alias for a quick pull‑and‑rebase

Add an alias to your global Git config so you can run a single command that always pulls the latest changes and rebases your local commits:

```bash
git config --global alias.up '!git fetch && git rebase @{u}'
```

Now `git up` will keep you up‑to‑date without creating merge commits.

### 7. Verify everything works

Run the following commands once to confirm the setup:

```bash
git remote -v          # Should show the GitHub URL
git branch -vv         # Should show [origin/master] next to master
git config --global -l | grep alias.up   
```
