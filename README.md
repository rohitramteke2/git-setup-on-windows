# 🔧 Git & GitHub — Complete Setup Guide

A single-file, zero-fluff reference for setting up Git, GitHub, and SSH from scratch on any operating system.

---

## 📖 What's Inside

| #  | Section                        | Covers                                                        |
|----|--------------------------------|---------------------------------------------------------------|
| 1  | Install Git                    | macOS, Ubuntu/Debian, Fedora/RHEL, Windows                    |
| 2  | First-Time Configuration       | Name, email, editor, pull strategy, credentials               |
| 3  | Create a GitHub Account        | Signup and 2FA                                                |
| 4  | Generate an SSH Key            | Ed25519 key generation, SSH agent, adding key to GitHub       |
| 5  | Multiple GitHub Accounts       | SSH config aliases for personal + work accounts               |
| 6  | Create a Repository            | Remote-first and local-first workflows                        |
| 7  | Daily Git Workflow             | status → add → commit → push cycle, diffs, logs              |
| 8  | Branching                      | Create, switch, list, delete (local & remote)                 |
| 9  | Merging & Rebasing             | Merge, rebase, conflict resolution                            |
| 10 | Undoing Things                 | restore, reset (soft/mixed/hard), revert, amend               |
| 11 | Stashing                       | stash, pop, apply, list, drop                                 |
| 12 | Working with Remotes           | add, set-url, fetch, pull, push, force-push                   |
| 13 | Tags                           | Lightweight, annotated, push, delete                          |
| 14 | Pull Requests                  | Fork → branch → PR workflow, keeping forks in sync            |
| 15 | `.gitignore`                   | Common patterns, global ignore, untracking files              |
| 16 | Useful Aliases                 | Shorthand commands for everyday use                           |
| 17 | GPG Signing Commits            | Key generation, config, verified badges                       |
| 18 | SSH Troubleshooting            | Permission errors, agent issues, firewall bypass (port 443)   |
| 19 | Quick Reference Card           | One-liner table of the most common commands                   |

---

## 🖥️ Supported Platforms

| Platform            | Tested |
|---------------------|--------|
| macOS (Homebrew)    | ✅      |
| Ubuntu / Debian     | ✅      |
| Fedora / RHEL       | ✅      |
| Windows (Git Bash)  | ✅      |
| WSL2                | ✅      |

---

## 🚀 Quick Start

If you just want to go from zero to pushing code:

```bash
# 1. Install Git (macOS example)
xcode-select --install

# 2. Configure identity
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# 3. Generate SSH key
ssh-keygen -t ed25519 -C "you@example.com"

# 4. Start agent & add key
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# 5. Copy public key → paste at github.com/settings/keys
cat ~/.ssh/id_ed25519.pub

# 6. Test connection
ssh -T git@github.com

# 7. Clone and start working
git clone git@github.com:your-username/your-repo.git
cd your-repo
```

---

## 📂 File

```
.
├── README.md                              ← you are here
└── git-github-complete-setup-guide.md     ← the full guide
```

---

## 🤝 Who Is This For

- **Beginners** setting up Git and GitHub for the first time.
- **Developers** switching to a new machine who need the setup steps in one place.
- **Teams** onboarding new members who need a standard Git configuration.
- Anyone tired of googling "how to add SSH key to GitHub" every six months.

---

## ✏️ Contributing

Found a typo, outdated command, or missing platform? Open an issue or submit a PR. Keep changes concise and tested on at least one platform.

---

## 📄 License

This guide is released under the [MIT License](LICENSE). Use it, share it, adapt it.

---

---

# 📘 Full Command Reference

Everything below is the complete guide with every command, ready to copy-paste.

---

## 1. Install Git

### macOS

```bash
# Option A: Xcode Command Line Tools (simplest)
xcode-select --install

# Option B: Homebrew
brew install git
```

### Ubuntu / Debian

```bash
sudo apt update
sudo apt install git -y
```

### Fedora / RHEL

```bash
sudo dnf install git -y
```

### Windows

Download from [git-scm.com](https://git-scm.com/download/win) and run the installer (select "Git Bash" and "Use Git from the Windows Command Prompt").
Or with winget:

```powershell
winget install --id Git.Git -e --source winget
```

### Verify

```bash
git --version
# git version 2.x.x
```

---

## 2. First-Time Git Configuration

```bash
# Required — your name and email (use the email linked to your GitHub account)
git config --global user.name "Your Full Name"
git config --global user.email "you@example.com"

# Set default branch name to main (instead of master)
git config --global init.defaultBranch main

# Set default editor (pick one)
git config --global core.editor "code --wait"   # VS Code
git config --global core.editor "nano"           # Nano
git config --global core.editor "vim"            # Vim

# Enable colored output
git config --global color.ui auto

# Set pull strategy (avoids merge-commit noise)
git config --global pull.rebase true

# Store credentials (optional, for HTTPS users)
git config --global credential.helper store      # plain-text file
git config --global credential.helper cache       # in-memory, times out
```

### Review your config

```bash
git config --global --list
```

Config lives at `~/.gitconfig` (global) and `.git/config` (per-repo).

---

## 3. Create a GitHub Account

1. Go to [github.com](https://github.com) → **Sign up**.
2. Choose a username, enter your email, and set a password.
3. Verify your email address by clicking the link GitHub sends you.
4. (Optional) Enable two-factor authentication: **Settings → Password and authentication → Two-factor authentication**.

---

## 4. Generate an SSH Key

SSH lets you push and pull without typing your password every time.

### 4.1 Check for Existing Keys

```bash
ls -al ~/.ssh
# Look for files like id_ed25519, id_rsa, etc.
```

If you already have a key pair you want to reuse, skip to step 4.3.

### 4.2 Generate a New Key

Ed25519 is the recommended algorithm (faster, more secure, shorter keys).

```bash
ssh-keygen -t ed25519 -C "you@example.com"
```

You'll see:

```
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/you/.ssh/id_ed25519):
```

Press **Enter** to accept the default path. Then set a passphrase (recommended) or press Enter for none.

This creates two files:

| File                      | What it is                           |
|---------------------------|--------------------------------------|
| `~/.ssh/id_ed25519`      | **Private key** — never share this   |
| `~/.ssh/id_ed25519.pub`  | **Public key** — this goes on GitHub |

> **Legacy systems that don't support Ed25519:** use `ssh-keygen -t rsa -b 4096 -C "you@example.com"` instead.

### 4.3 Start the SSH Agent and Add Your Key

```bash
# Start the agent in the background
eval "$(ssh-agent -s)"
# Agent pid 12345

# Add your private key
ssh-add ~/.ssh/id_ed25519
```

#### macOS only — persist across reboots

Create or edit `~/.ssh/config`:

```
Host github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
```

Then add with the keychain flag:

```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

#### Windows (Git Bash)

```bash
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_ed25519
```

For persistence on Windows, configure the OpenSSH Authentication Agent service to start automatically.

### 4.4 Copy the Public Key

```bash
# macOS
pbcopy < ~/.ssh/id_ed25519.pub

# Linux (requires xclip)
xclip -selection clipboard < ~/.ssh/id_ed25519.pub

# Windows (Git Bash)
clip < ~/.ssh/id_ed25519.pub

# Or just print it and copy manually
cat ~/.ssh/id_ed25519.pub
```

### 4.5 Add the Key to GitHub

1. Go to **GitHub → Settings → SSH and GPG keys** ([direct link](https://github.com/settings/keys)).
2. Click **New SSH key**.
3. Title: something descriptive like `My Laptop` or `Work MacBook`.
4. Key type: **Authentication Key**.
5. Paste the public key into the **Key** field.
6. Click **Add SSH key**.

### 4.6 Test the Connection

```bash
ssh -T git@github.com
```

Expected output:

```
Hi your-username! You've successfully authenticated, but GitHub
does not provide shell access.
```

If you see a fingerprint prompt the first time, type `yes` to continue.

---

## 5. SSH Config for Multiple GitHub Accounts

If you have a personal and a work account, edit `~/.ssh/config`:

```
# Personal account
Host github.com-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_personal

# Work account
Host github.com-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_work
```

Then clone using the alias:

```bash
git clone git@github.com-personal:your-user/repo.git
git clone git@github.com-work:your-org/repo.git
```

Set per-repo identity:

```bash
cd work-repo
git config user.name "Work Name"
git config user.email "work@company.com"
```

---

## 6. Create a Repository

### On GitHub (remote first)

1. Click **+** → **New repository** on GitHub.
2. Name it, choose public/private, optionally add a README.
3. Clone it locally:

```bash
# SSH (recommended)
git clone git@github.com:your-username/repo-name.git

# HTTPS (if not using SSH)
git clone https://github.com/your-username/repo-name.git

cd repo-name
```

### Locally first (then push to GitHub)

```bash
mkdir my-project && cd my-project
git init

# Create something
echo "# My Project" > README.md

git add .
git commit -m "Initial commit"

# Create the repo on GitHub first, then link it
git remote add origin git@github.com:your-username/my-project.git
git branch -M main
git push -u origin main
```

---

## 7. Daily Git Workflow

### The basic cycle

```bash
# 1. Check status
git status

# 2. Stage changes
git add file.txt           # one file
git add .                  # everything in current dir
git add -A                 # everything everywhere
git add -p                 # interactively pick hunks

# 3. Commit
git commit -m "Short description of the change"

# 4. Push to remote
git push
```

### See what changed

```bash
git diff                   # unstaged changes
git diff --staged          # staged changes (about to commit)
git diff main..feature     # difference between two branches
git log                    # commit history
git log --oneline --graph  # compact visual history
git log -5                 # last 5 commits
```

---

## 8. Branching

```bash
# Create and switch to a new branch
git checkout -b feature/login-page

# Or the newer syntax
git switch -c feature/login-page

# List branches
git branch            # local
git branch -a         # local + remote

# Switch branches
git checkout main
git switch main

# Delete a branch
git branch -d feature/login-page       # safe (warns if unmerged)
git branch -D feature/login-page       # force delete

# Push a new branch to remote
git push -u origin feature/login-page

# Delete a remote branch
git push origin --delete feature/login-page
```

---

## 9. Merging and Rebasing

### Merge

```bash
git checkout main
git merge feature/login-page
```

### Rebase (replays your commits on top of main for a linear history)

```bash
git checkout feature/login-page
git rebase main
```

### Resolve merge conflicts

```bash
# After a conflict, Git marks files. Edit them, then:
git add resolved-file.txt
git commit                          # for merge
git rebase --continue               # for rebase
git rebase --abort                  # abandon the rebase
```

---

## 10. Undoing Things

```bash
# Unstage a file (keep changes in working dir)
git restore --staged file.txt

# Discard uncommitted changes to a file
git restore file.txt

# Amend the last commit message
git commit --amend -m "Better message"

# Amend the last commit with new file changes
git add forgotten-file.txt
git commit --amend --no-edit

# Undo the last commit but keep changes staged
git reset --soft HEAD~1

# Undo the last commit and unstage changes
git reset HEAD~1

# Undo the last commit and discard all changes (DESTRUCTIVE)
git reset --hard HEAD~1

# Revert a commit (creates a new undo commit — safe for shared branches)
git revert <commit-hash>
```

---

## 11. Stashing

Temporarily shelve changes without committing.

```bash
git stash                     # stash tracked changes
git stash -u                  # include untracked files
git stash save "WIP: login"  # with a label

git stash list                # see all stashes
git stash pop                 # apply most recent and remove it
git stash apply               # apply but keep in stash list
git stash drop stash@{0}     # delete a specific stash
git stash clear               # delete all stashes
```

---

## 12. Working with Remotes

```bash
# List remotes
git remote -v

# Add a remote
git remote add origin git@github.com:user/repo.git
git remote add upstream git@github.com:original-author/repo.git

# Change remote URL (e.g. HTTPS → SSH)
git remote set-url origin git@github.com:user/repo.git

# Fetch (download changes without merging)
git fetch origin

# Pull (fetch + merge)
git pull origin main

# Push
git push origin main

# Force push (use with care — rewrites remote history)
git push --force-with-lease origin feature-branch
```

---

## 13. Tags

```bash
# Lightweight tag
git tag v1.0.0

# Annotated tag (recommended for releases)
git tag -a v1.0.0 -m "Release version 1.0.0"

# Tag a past commit
git tag -a v0.9.0 <commit-hash> -m "Beta release"

# Push tags
git push origin v1.0.0       # one tag
git push origin --tags        # all tags

# List and delete
git tag                       # list
git tag -d v1.0.0             # delete local
git push origin :refs/tags/v1.0.0  # delete remote
```

---

## 14. Pull Requests (GitHub Workflow)

```bash
# 1. Fork the repo on GitHub (for open-source contributions)
# 2. Clone your fork
git clone git@github.com:your-username/forked-repo.git
cd forked-repo

# 3. Add the original repo as upstream
git remote add upstream git@github.com:original-owner/repo.git

# 4. Create a feature branch
git checkout -b fix/typo-in-readme

# 5. Make changes, commit, push
git add .
git commit -m "Fix typo in README"
git push -u origin fix/typo-in-readme

# 6. Open a Pull Request on GitHub
#    Go to your fork → "Compare & pull request" → fill in details → Create

# 7. Keep your fork in sync
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

---

## 15. `.gitignore`

Create a `.gitignore` in the repo root:

```gitignore
# Dependencies
node_modules/
vendor/
__pycache__/
*.pyc

# Build output
dist/
build/
*.o
*.class

# Environment and secrets
.env
.env.local
*.pem
*.key

# OS junk
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp
*.swo

# Logs
*.log
logs/
```

### Global gitignore (for OS/editor files across all repos)

```bash
git config --global core.excludesfile ~/.gitignore_global
```

Then add OS/editor patterns to `~/.gitignore_global`.

### Already tracked a file you now want ignored?

```bash
git rm --cached file.txt
# Then add file.txt to .gitignore and commit
```

---

## 16. Useful Aliases

Add to `~/.gitconfig` under `[alias]`:

```ini
[alias]
    s   = status
    co  = checkout
    br  = branch
    ci  = commit
    lg  = log --oneline --graph --decorate --all
    last = log -1 HEAD --stat
    unstage = restore --staged
    undo = reset HEAD~1
    amend = commit --amend --no-edit
    wip = !git add -A && git commit -m 'WIP'
```

Or set them from the command line:

```bash
git config --global alias.s "status"
git config --global alias.lg "log --oneline --graph --decorate --all"
```

---

## 17. GPG Signing Commits (Optional)

Verified "Verified" badges on GitHub commits.

```bash
# Generate a GPG key
gpg --full-generate-key
# Choose RSA and RSA, 4096 bits, your GitHub email

# Find your key ID
gpg --list-secret-keys --keyid-format=long
# Look for sec   rsa4096/ABCDEF1234567890

# Tell Git to use it
git config --global user.signingkey ABCDEF1234567890
git config --global commit.gpgsign true

# Export the public key and add it to GitHub (Settings → SSH and GPG keys)
gpg --armor --export ABCDEF1234567890
```

---

## 18. SSH Troubleshooting

| Problem                                  | Fix                                                                          |
|------------------------------------------|------------------------------------------------------------------------------|
| `Permission denied (publickey)`          | Run `ssh-add ~/.ssh/id_ed25519` and verify the public key is on GitHub       |
| Agent not running                        | Run `eval "$(ssh-agent -s)"`                                                 |
| Wrong key being used                     | Check `ssh -vT git@github.com` for which key it tries                        |
| Firewall blocks port 22                  | Use SSH over HTTPS port — see below                                          |
| Key not persisting after reboot (macOS)  | Add `UseKeychain yes` to `~/.ssh/config`                                     |

### SSH over HTTPS port (port 443) — bypasses firewalls

Edit `~/.ssh/config`:

```
Host github.com
  HostName ssh.github.com
  Port 443
  User git
  IdentityFile ~/.ssh/id_ed25519
```

Test:

```bash
ssh -T -p 443 git@ssh.github.com
```

### Verbose debug

```bash
ssh -vvv git@github.com
```

### Fix file permissions

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 644 ~/.ssh/config
```

---

## 19. Quick Reference Card

| Task               | Command                                                      |
|--------------------|--------------------------------------------------------------|
| Init a repo        | `git init`                                                   |
| Clone a repo       | `git clone git@github.com:user/repo.git`                     |
| Stage all          | `git add -A`                                                 |
| Commit             | `git commit -m "message"`                                    |
| Push               | `git push`                                                   |
| Pull               | `git pull`                                                   |
| New branch         | `git switch -c branch-name`                                  |
| Merge              | `git merge branch-name`                                      |
| View log           | `git log --oneline --graph`                                  |
| Stash              | `git stash` / `git stash pop`                                |
| Undo last commit   | `git reset HEAD~1`                                           |
| Check remote       | `git remote -v`                                              |
| Switch HTTPS→SSH   | `git remote set-url origin git@github.com:user/repo.git`     |

---

*Generated on September 28, 2026. Covers Git 2.x and GitHub's current SSH workflow.*
