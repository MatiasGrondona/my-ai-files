# Git Submodules Cheatsheet

---

## 1. Create a local repo

```bash
git init my-ai-files
cd my-ai-files
```

---

## 2. Add a submodule

```bash
# ⚠️ Always use forward slashes / even on Windows — never backslashes \
git submodule add https://github.com/android/skills .skills/android/official
git submodule add https://github.com/anthropics/claude-skills .skills/claude/official
```

> The submodule is stored as a reference (pointer to a commit), not a full copy.
> The folder must not exist before running this command — if it does, delete it first:
> `Remove-Item -Recurse -Force .skills/android/official` (PowerShell)
> `rm -rf .skills/android/official` (bash)

---

## 3. Update submodules

### Update all submodules at once
```bash
git submodule update --remote --merge
```

### Update a specific submodule
```bash
git submodule update --remote --merge .skills/android/official
```

> `--remote` pulls the latest commit from the upstream branch.
> `--merge` merges the changes into your local tracking branch.

---

## 4. Connect to a remote repo

```bash
# ⚠️ Create the repo on GitHub first (empty, no README), then run:
git remote add origin https://github.com/your-username/my-ai-files.git
```

---

## 5. Stage, commit and push

```bash
git add .
git commit -m "Your commit message"

# ⚠️ Check your branch name first — it may be 'master' instead of 'main'
git branch

# Push accordingly
git push -u origin main    # if branch is main
git push -u origin master  # if branch is master
```

> After the first push with `-u`, future pushes only need `git push`.

---

## 6. Clone on another PC

```bash
# ⚠️ Always use --recurse-submodules when cloning
# Without it, submodule folders will be empty
git clone --recurse-submodules https://github.com/your-username/my-ai-files.git
```

### If you already cloned without the flag
```bash
git submodule update --init --recursive
```

---

## 7. Remove a submodule

```bash
# Step 1 — Deinitialize the submodule
git submodule deinit -f .skills/android/official

# Step 2 — Remove it from the git index
git rm -f .skills/android/official

# Step 3 — Delete the cached folder
Remove-Item -Recurse -Force .git/modules/.skills/android/official  # PowerShell
rm -rf .git/modules/.skills/android/official                       # bash

# Step 4 — Commit the removal
git commit -m "Remove android/official submodule"
```

> You also need to manually remove the submodule entry from `.gitmodules` if it persists.
