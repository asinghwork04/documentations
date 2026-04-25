# Git Multi-Account Setup (Work + Personal) — macOS (Strict Isolation)

A **deterministic, low-risk setup** to use multiple GitHub accounts on one machine with:

* Folder-based identity isolation (`includeIf`)
* SSH key separation (auth isolation)
* macOS Keychain + SSH agent (no repeated passphrase prompts)

---

# 0. Goals

* Same machine, multiple GitHub accounts
* No identity leakage (email)
* No auth leakage (wrong account push)
* No manual switching
* Reversible without side effects

---

# 1. Prerequisites

```bash
git --version
ssh -V
```

You need:

* Git
* OpenSSH (default on macOS)
* Two GitHub accounts (work + personal)

---

# 2. Folder Structure (identity boundary)

```bash
mkdir -p ~/Desktop/work
mkdir -p ~/Desktop/aholic
```

```
~/Desktop/
 ├── work/      → work identity
 └── aholic/    → personal identity
```

---

# 3. Git Identity Setup (conditional config)

## 3.1 Personal config

```bash
nano ~/.gitconfig-aholic
```

```ini
[user]
  name = Your Name
  email = your-personal-email@example.com
```

---

## 3.2 Work config

```bash
nano ~/.gitconfig-work
```

```ini
[user]
  name = Your Name
  email = your-work-email@company.com
```

---

## 3.3 Global config (with includeIf)

```bash
nano ~/.gitconfig
```

```ini
[user]
  name = Your Name
  email = your-work-email@company.com   # fallback

[includeIf "gitdir:~/Desktop/aholic/"]
  path = ~/.gitconfig-aholic

[includeIf "gitdir:~/Desktop/work/"]
  path = ~/.gitconfig-work
```

---

## 3.4 Verify

```bash
cd ~/Desktop/aholic && git config user.email
cd ~/Desktop/work && git config user.email
```

Expected:

* `aholic` → personal email
* `work` → work email

---

# 4. SSH Setup (auth isolation)

## 4.1 Generate personal key

```bash
ssh-keygen -t ed25519 -C "your-personal-email@example.com" -f ~/.ssh/id_ed25519_aholic
```

---

## 4.2 SSH config

```bash
nano ~/.ssh/config
```

```ini
Host *
  AddKeysToAgent yes
  UseKeychain yes

Host github-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519
  IdentitiesOnly yes

Host github-aholic
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_aholic
  IdentitiesOnly yes
```

---

## 4.3 Add key to GitHub

```bash
cat ~/.ssh/id_ed25519_aholic.pub
```

* Copy output
* GitHub → Settings → SSH keys → Add

---

## 4.4 Load keys into Keychain (IMPORTANT)

```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519_aholic
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

👉 This ensures:

* passphrase asked once
* then remembered securely

---

## 4.5 Verify SSH

```bash
ssh -T git@github-aholic
ssh -T git@github-work
```

Expected:

```
Hi <personal-username>!
Hi <work-username>!
```

---

# 5. Clone & Remote Rules

## 5.1 Personal repos (MANDATORY SSH)

```bash
git clone git@github-aholic:username/repo.git
```

---

## 5.2 Work repos

```bash
git clone https://github.com/org/repo.git
```

(optional later migrate to SSH)

---

## 5.3 Convert existing personal repo

```bash
git remote set-url origin git@github-aholic:username/repo.git
```

---

# 6. Verification Checklist

Run inside repo:

```bash
git config user.email
git config --show-origin user.email
git remote -v
```

---

### Expected

**Personal repo**

```
email → personal
origin → git@github-aholic:...
```

**Work repo**

```
email → work
origin → https://... OR github-work
```

---

# 7. Safe Usage Rules

## MUST

* Keep repos inside correct folders
* Use SSH for personal repos
* Verify remote before first push

---

## DO NOT

```bash
git clone https://github.com/...   # for personal
```

```bash
create repo outside ~/Desktop/work or ~/Desktop/aholic
```

```bash
move repos randomly between folders
```

---

# 8. Edge Cases

### Repo outside folders

→ fallback = work email

---

### Folder name mismatch

```
~/Desktop/aholic2 → NOT matched
```

---

### Manual override

```bash
git config user.email something
```

→ overrides includeIf

---

# 9. Credential Helper (macOS)

Check:

```bash
git config --show-origin --get credential.helper
```

Typical:

```
osxkeychain
```

* Used only for HTTPS
* No conflict with SSH

---

# 10. Optional: migrate work repos to SSH

```bash
git remote set-url origin git@github-work:org/repo.git
```

---

# 11. Cleanup / Dismantle

## Remove SSH

```bash
rm -f ~/.ssh/id_ed25519_aholic*
rm -f ~/.ssh/config
```

---

## Remove Git configs

```bash
rm -f ~/.gitconfig-aholic ~/.gitconfig-work
nano ~/.gitconfig   # remove includeIf
```

---

## Reset identity

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

# 13. Quick sanity check

```bash
git config user.email && git remote -v
```

---

# 14. Final State

* Identity isolation ✔
* SSH isolation ✔
* No repeated passphrase ✔
* No manual switching ✔
* Reversible ✔

---

# END
