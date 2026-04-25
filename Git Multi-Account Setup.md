# Git Multi-Account Setup (Single Machine, Strict Isolation)

This guide sets up **two completely isolated Git identities** (e.g., work + personal) on the same machine using:

* Folder-based Git config (identity isolation)
* SSH key separation (authentication isolation)

---

# 0. Goals

* Same machine for multiple GitHub accounts
* No accidental identity leakage
* No manual switching required
* No disruption to existing work setup

---

# 1. Prerequisites

* Git installed
* GitHub accounts (e.g., work + personal)
* Terminal access
* Basic familiarity with `cd`, `ls`

Check:

```bash
git --version
ssh -V
```

---

# 2. Folder Structure

Create two root folders:

```bash
mkdir -p ~/Desktop/work
mkdir -p ~/Desktop/personal
```

Example:

```
~/Desktop/
 ├── work/
 └── personal/
```

---

# 3. Git Identity Setup (Conditional Config)

## 3.1 Create personal config

```bash
nano ~/.gitconfig-personal
```

```ini
[user]
  name = Your Name
  email = your-personal-email@example.com
```

---

## 3.2 Create work config

```bash
nano ~/.gitconfig-work
```

```ini
[user]
  name = Your Name
  email = your-work-email@company.com
```

---

## 3.3 Update global config

```bash
nano ~/.gitconfig
```

```ini
[user]
  name = Your Name
  email = your-work-email@company.com   # fallback

[includeIf "gitdir:~/Desktop/personal/"]
  path = ~/.gitconfig-personal

[includeIf "gitdir:~/Desktop/work/"]
  path = ~/.gitconfig-work
```

---

## 3.4 Verify

```bash
cd ~/Desktop/personal
git config user.email   # expected: personal email

cd ~/Desktop/work
git config user.email   # expected: work email
```

---

# 4. SSH Setup (Authentication Isolation)

## 4.1 Generate personal SSH key

```bash
ssh-keygen -t ed25519 -C "your-personal-email@example.com" -f ~/.ssh/id_ed25519_personal
```

---

## 4.2 SSH config

```bash
nano ~/.ssh/config
```

```ini
Host github-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_personal

Host github-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519
```

---

## 4.3 Add key to GitHub

```bash
cat ~/.ssh/id_ed25519_personal.pub
```

* Copy output
* GitHub → Settings → SSH Keys → Add

---

## 4.4 Verify

```bash
ssh -T git@github-personal
ssh -T git@github-work
```

Expected:

```
Hi <personal-username>!
Hi <work-username>!
```

---

# 5. Clone Rules

## Personal repos (IMPORTANT)

```bash
git clone git@github-personal:username/repo.git
```

---

## Work repos

```bash
git clone https://github.com/org/repo.git
```

(or later migrate to SSH)

---

# 6. Convert Existing Repo to SSH

```bash
git remote set-url origin git@github-personal:username/repo.git
```

Verify:

```bash
git remote -v
```

---

# 7. Verification Checklist

Inside any repo:

```bash
git config user.email
git config --show-origin user.email
git remote -v
```

---

## Expected

### Personal repo

```
email → personal
origin → github-personal
```

### Work repo

```
email → work
origin → https or github-work
```

---

# 8. Safe Usage Rules

## MUST follow

* Keep repos inside correct folders
* Use SSH for personal repos
* Do not mix folder structure

---

## DO NOT

```bash
git clone https://github.com/...   # for personal
```

```bash
create repo outside ~/Desktop/work or personal
```

```bash
move repos randomly between folders
```

---

# 9. Common Errors

## ❌ Permission denied (publickey)

Cause:

* SSH key not added to GitHub

Fix:

```bash
cat ~/.ssh/id_ed25519_personal.pub
```

---

## ❌ Wrong email in commits

Cause:

* repo outside configured folder

Fix:

* move repo or check config

---

## ❌ not a git repository

Cause:

* folder missing `.git`

---

# 10. Optional: Migrate Work Repos to SSH

```bash
git remote set-url origin git@github-work:org/repo.git
```

---

# 11. Cleanup / Dismantle

## Remove SSH setup

```bash
rm -f ~/.ssh/id_ed25519_personal*
rm -f ~/.ssh/config
```

---

## Remove conditional configs

```bash
rm -f ~/.gitconfig-personal ~/.gitconfig-work
nano ~/.gitconfig   # remove includeIf sections
```

---

## Reset global identity

```bash
git config --global --unset user.name
git config --global --unset user.email
```

---

# 12. Mental Model

```
Folder → controls identity (email)
SSH alias → controls account (auth)
```

---

# 13. Final Outcome

* No manual switching
* No identity leakage
* Full separation of work & personal
* Safe, scalable, reversible setup

---

# 14. Quick Sanity Command

```bash
git config user.email && git remote -v
```

---

# 15. Future Improvements

* Convert all repos to SSH
* Add signing (GPG)
* Use separate GitHub profiles in browser
* Use direnv for environment isolation

---

# END
