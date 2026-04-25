# Git Multi-Account Setup (Work + Personal) — macOS (Fool-Proof Guide)

A **deterministic, production-grade setup** to use multiple GitHub accounts on one machine with:

* Folder-based identity isolation (`includeIf`)
* SSH key separation (auth isolation)
* macOS Keychain + SSH agent (no repeated passphrase prompts)
* Guardrails to prevent wrong-account pushes/clones

---

# 0. Goals

* Same machine, multiple GitHub accounts
* No identity leakage (wrong `user.email`)
* No auth leakage (wrong account push)
* No manual switching
* Easy to verify, easy to revert

---

# 1. Prerequisites

```bash id="q2o8t1"
git --version
ssh -V
```

You need:

* Git
* OpenSSH (default on macOS)
* Two GitHub accounts (work + personal)

---

# 2. Folder Structure (Identity Boundary)

```bash id="c4p8l9"
mkdir -p ~/Desktop/work
mkdir -p ~/Desktop/aholic
```

```id="a6k9m0"
~/Desktop/
 ├── work/      → work identity
 └── aholic/    → personal identity
```

> **Rule:** Repo location determines commit identity.

---

# 3. Git Identity Setup (Conditional Config)

## 3.1 Personal config

```bash id="u9y2dn"
nano ~/.gitconfig-aholic
```

```ini id="s1l7f3"
[user]
  name = Your Name
  email = your-personal-email@example.com
```

---

## 3.2 Work config

```bash id="m5w0k2"
nano ~/.gitconfig-work
```

```ini id="r8e2c6"
[user]
  name = Your Name
  email = your-work-email@company.com
```

---

## 3.3 Global config (with includeIf)

```bash id="g3n6v4"
nano ~/.gitconfig
```

```ini id="d2b5x8"
[user]
  name = Your Name
  email = your-work-email@company.com   # fallback (IMPORTANT)

[includeIf "gitdir:~/Desktop/aholic/"]
  path = ~/.gitconfig-aholic

[includeIf "gitdir:~/Desktop/work/"]
  path = ~/.gitconfig-work
```

> **Do NOT remove global user.email** — it is your safe fallback.

---

## 3.4 Verify identity

```bash id="v7q1c0"
cd ~/Desktop/aholic && git config user.email
cd ~/Desktop/work && git config user.email
```

Expected:

* `aholic` → personal email
* `work` → work email

---

# 4. SSH Setup (Authentication Isolation)

## 4.1 Generate personal key

```bash id="k4h2z7"
ssh-keygen -t ed25519 -C "your-personal-email@example.com" -f ~/.ssh/id_ed25519_aholic
```

---

## 4.2 SSH config (STRICT + KEYCHAIN)

```bash id="p0n6r5"
nano ~/.ssh/config
```

```ini id="t9x4s2"
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

## 4.3 Add key to GitHub (personal)

```bash id="w8f2m1"
cat ~/.ssh/id_ed25519_aholic.pub
```

* Copy output
* GitHub → Settings → SSH Keys → Add

---

## 4.4 Load keys into macOS Keychain (CRITICAL)

```bash id="j3v8q4"
ssh-add --apple-use-keychain ~/.ssh/id_ed25519_aholic
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

> Ensures: passphrase asked once → then remembered securely

---

## 4.5 Verify SSH routing

```bash id="z6b2y9"
ssh -T git@github-aholic
ssh -T git@github-work
```

Expected:

```id="n1c7e5"
Hi <personal-username>!
Hi <work-username>!
```

---

# 5. CLONING RULES (CRITICAL — MOST COMMON FAILURE POINT)

## 🚫 NEVER USE (for personal repos)

```bash id="o4t7q3"
git clone git@github.com:username/repo.git
```

> This bypasses your SSH routing and uses the wrong account.

---

## ✅ Personal repos (MANDATORY)

```bash id="b5r2c8"
git clone git@github-aholic:username/repo.git
```

---

## ✅ Work repos (recommended)

### Option A — HTTPS (simple, current safe setup)

```bash id="e9m1k6"
git clone https://github.com/org/repo.git
```

---

### Option B — SSH (optional, consistent)

```bash id="x2s8n4"
git clone git@github-work:org/repo.git
```

---

# 6. Fixing a wrongly cloned repo

If you cloned using `github.com` instead of alias:

```bash id="f7k3p0"
git remote set-url origin git@github-aholic:username/repo.git
```

Verify:

```bash id="y1h9d2"
git remote -v
```

---

# 7. Verification Checklist (RUN INSIDE REPO)

```bash id="l0q5z8"
git config user.email
git config --show-origin user.email
git remote -v
```

---

## Expected

### Personal repo

```id="c2n4u7"
email → personal
origin → git@github-aholic:...
```

---

### Work repo

```id="h3v6b1"
email → work
origin → https://... OR github-work
```

---

# 8. Common Failure Scenarios

## ❌ Wrong SSH host

Using:

```bash id="q6w2e1"
git@github.com:...
```

→ authenticates as wrong account
→ private repo access fails

---

## ❌ Repo outside folders

```bash id="k8t3m9"
~/Desktop/random-repo
```

→ uses fallback (work email)

---

## ❌ Folder mismatch

```bash id="d4r7n2"
~/Desktop/aholic2
```

→ not matched by includeIf

---

## ❌ Manual override

```bash id="s5p8x1"
git config user.email something
```

→ overrides system

---

# 9. Credential Helper (macOS)

Check:

```bash id="u3y6g9"
git config --show-origin --get credential.helper
```

Expected:

```id="w7c2k5"
osxkeychain
```

* Used only for HTTPS
* Does NOT interfere with SSH

---

# 10. SSH Agent Persistence (Optional but Recommended)

If prompts reappear in new terminals:

```bash id="v8r1z4"
nano ~/.zshrc
```

Add:

```bash id="m2q9k6"
eval "$(ssh-agent -s)"
ssh-add --apple-use-keychain ~/.ssh/id_ed25519_aholic 2>/dev/null
ssh-add --apple-use-keychain ~/.ssh/id_ed25519 2>/dev/null
```

---

# 11. Guardrails (Prevent Mistakes)

## Alias for safe cloning

```bash id="r9k4c1"
alias clone-personal='git clone git@github-aholic:'
alias clone-work='git clone https://github.com/'
```

---

# 12. Cleanup / Dismantle

## Remove SSH

```bash id="t1m7y3"
rm -f ~/.ssh/id_ed25519_aholic*
rm -f ~/.ssh/config
```

---

## Remove Git configs

```bash id="z5x8c2"
rm -f ~/.gitconfig-aholic ~/.gitconfig-work
nano ~/.gitconfig   # remove includeIf
```

---

## Reset identity

```bash id="n6p3r7"
git config --global --unset user.name
git config --global --unset user.email
```

---

# 13. Mental Model

```id="b8c1n4"
Folder → controls identity (email)
SSH alias → controls account (auth)
Host in remote URL → decides which key is used
```

---

# 14. Quick Sanity Command

```bash id="q4d9k2"
git config user.email && git remote -v
```

---

# 15. Final State

* Identity isolation ✔
* SSH isolation ✔
* No passphrase spam ✔
* No manual switching ✔
* No accidental cross-account pushes ✔

---

# END
