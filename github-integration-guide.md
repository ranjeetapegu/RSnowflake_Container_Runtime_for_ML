# GitHub Integration Guide

Complete step-by-step guide for integrating a local project with GitHub.

---

## Table of Contents
1. [Initial Setup](#initial-setup)
2. [SSH Authentication Setup](#ssh-authentication-setup)
3. [Connect Local Repository to GitHub](#connect-local-repository-to-github)
4. [Push Your Code](#push-your-code)
5. [Daily Git Workflow](#daily-git-workflow)
6. [Alternative: HTTPS with Personal Access Token](#alternative-https-with-personal-access-token)
7. [Troubleshooting](#troubleshooting)

---

## Initial Setup

### 1. Create a GitHub Repository

1. Go to [GitHub](https://github.com)
2. Click the `+` icon in the top right corner
3. Select "New repository"
4. Enter repository name (e.g., `RSnowflake_Container_Runtime_for_ML`)
5. Choose public or private
6. **Do NOT** initialize with README, .gitignore, or license if you have existing code
7. Click "Create repository"

### 2. Initialize Local Git Repository (if not already done)

```bash
# Navigate to your project directory
cd /path/to/your/project

# Initialize git repository
git init

# Check status
git status

# Add all files
git add .

# Create initial commit
git commit -m "Initial commit"
```

---

## SSH Authentication Setup

SSH is the recommended method for authenticating with GitHub.

### Step 1: Generate SSH Key

```bash
# Generate a new SSH key
ssh-keygen -t ed25519 -C "your_email@example.com" -f ~/.ssh/id_ed25519_personal
```

**Note:** Press Enter when prompted for a passphrase (or set one for extra security)

### Step 2: Start SSH Agent and Add Key

```bash
# Start the SSH agent
eval "$(ssh-agent -s)"

# Add your SSH key to the agent (macOS)
ssh-add --apple-use-keychain ~/.ssh/id_ed25519_personal

# For Linux, use:
# ssh-add ~/.ssh/id_ed25519_personal
```

### Step 3: Copy SSH Public Key

```bash
# Display your public key
cat ~/.ssh/id_ed25519_personal.pub

# On macOS, copy to clipboard:
pbcopy < ~/.ssh/id_ed25519_personal.pub

# On Linux with xclip:
# xclip -sel clip < ~/.ssh/id_ed25519_personal.pub
```

### Step 4: Add SSH Key to GitHub

1. Go to [GitHub SSH Settings](https://github.com/settings/keys)
2. Click "New SSH key"
3. Enter a descriptive title (e.g., "MacBook Pro - Personal")
4. Paste your public key into the "Key" field
5. Click "Add SSH key"
6. Confirm with your GitHub password if prompted

### Step 5: Configure SSH Config File

```bash
# Create or edit SSH config
mkdir -p ~/.ssh
cat >> ~/.ssh/config <<'EOF'

# Default GitHub (personal)
Host github.com
  HostName github.com
  User git
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519_personal
EOF
```

**For multiple GitHub accounts**, use different host aliases:

```bash
cat >> ~/.ssh/config <<'EOF'

# Personal GitHub
Host github.com-personal
  HostName github.com
  User git
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519_personal

# Work GitHub
Host github.com-work
  HostName github.com
  User git
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519_work
EOF
```

### Step 6: Test SSH Connection

```bash
# Test the connection
ssh -T git@github.com

# Expected output:
# Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

---

## Connect Local Repository to GitHub

### Step 1: Add Remote Repository

```bash
# Using SSH (recommended)
git remote add origin git@github.com:username/repository-name.git

# Or using HTTPS
# git remote add origin https://github.com/username/repository-name.git
```

**Example:**
```bash
git remote add origin git@github.com:ranjeetapegu/RSnowflake_Container_Runtime_for_ML.git
```

### Step 2: Verify Remote

```bash
# Check remote configuration
git remote -v

# Expected output:
# origin  git@github.com:username/repository-name.git (fetch)
# origin  git@github.com:username/repository-name.git (push)
```

### Step 3: Change Remote URL (if needed)

```bash
# Switch from HTTPS to SSH
git remote set-url origin git@github.com:username/repository-name.git

# Switch from SSH to HTTPS
git remote set-url origin https://github.com/username/repository-name.git
```

---

## Push Your Code

### First Push

```bash
# Push to main branch and set upstream
git push -u origin main

# If your default branch is 'master':
# git push -u origin master
```

**Note:** The `-u` flag sets up tracking between your local branch and the remote branch.

### Verify on GitHub

1. Go to your repository on GitHub
2. Refresh the page
3. You should see all your files

---

## Daily Git Workflow

### Basic Workflow

```bash
# 1. Check status
git status

# 2. Add files
git add .                    # Add all files
git add filename.txt         # Add specific file
git add *.py                 # Add all Python files

# 3. Commit changes
git commit -m "Description of changes"

# 4. Push to GitHub
git push

# 5. Pull latest changes from GitHub
git pull
```

### Check What Changed

```bash
# See what changed in tracked files
git diff

# See what changed in staged files
git diff --staged

# See commit history
git log

# See compact commit history
git log --oneline
```

### Branch Management

```bash
# Create new branch
git branch feature-name

# Switch to branch
git checkout feature-name

# Create and switch in one command
git checkout -b feature-name

# List all branches
git branch -a

# Merge branch into current branch
git merge feature-name

# Delete local branch
git branch -d feature-name

# Delete remote branch
git push origin --delete feature-name
```

---

## Alternative: HTTPS with Personal Access Token

If you prefer HTTPS over SSH, you'll need a Personal Access Token (PAT).

### Step 1: Create Personal Access Token

1. Go to [GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)](https://github.com/settings/tokens)
2. Click "Generate new token" → "Generate new token (classic)"
3. Give it a descriptive name
4. Set expiration (recommended: 90 days)
5. Select scopes:
   - `repo` (all repo permissions)
   - `workflow` (if using GitHub Actions)
6. Click "Generate token"
7. **Copy the token immediately** (you won't see it again!)

### Step 2: Configure Git to Use Token

```bash
# Add remote with HTTPS
git remote add origin https://github.com/username/repository-name.git

# When you push, use your token as the password
git push -u origin main
# Username: your_github_username
# Password: your_personal_access_token
```

### Step 3: Cache Credentials (Optional)

```bash
# Cache credentials for 1 hour
git config --global credential.helper 'cache --timeout=3600'

# Or use macOS Keychain
git config --global credential.helper osxkeychain

# Or use Windows Credential Manager
git config --global credential.helper wincred
```

---

## Troubleshooting

### Permission Denied (publickey)

```bash
# Test SSH connection
ssh -T git@github.com

# If it fails, check:
# 1. SSH key is added to agent
ssh-add -l

# 2. Add key if missing
ssh-add ~/.ssh/id_ed25519_personal

# 3. Verify SSH config
cat ~/.ssh/config

# 4. Test with verbose output
ssh -vT git@github.com
```

### Remote Already Exists

```bash
# Remove existing remote
git remote remove origin

# Add new remote
git remote add origin git@github.com:username/repository-name.git
```

### Authentication Failed (HTTPS)

```bash
# Make sure you're using a Personal Access Token, not your password
# Update your credentials:
git config --global --unset credential.helper
git push  # This will prompt for new credentials
```

### Wrong User Authenticated

```bash
# Check current Git user
git config user.name
git config user.email

# Set correct user for this repository
git config user.name "Your Name"
git config user.email "your_email@example.com"

# Or set globally
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```

### Push Rejected (Non-Fast-Forward)

```bash
# Pull latest changes first
git pull origin main

# If there are conflicts, resolve them, then:
git add .
git commit -m "Resolved merge conflicts"
git push origin main
```

### Large Files Error

```bash
# Use Git LFS for files larger than 100MB
git lfs install
git lfs track "*.psd"
git add .gitattributes
git commit -m "Add Git LFS tracking"
```

---

## Quick Reference Commands

```bash
# Status and Info
git status                          # Show working tree status
git log                            # Show commit history
git remote -v                      # Show remote repositories

# Basic Operations
git add .                          # Stage all changes
git commit -m "message"            # Commit staged changes
git push                           # Push to remote
git pull                           # Pull from remote

# Branch Operations
git branch                         # List branches
git checkout -b new-branch         # Create and switch to new branch
git merge branch-name              # Merge branch into current

# Undo Operations
git reset HEAD~1                   # Undo last commit (keep changes)
git reset --hard HEAD~1            # Undo last commit (discard changes)
git checkout -- filename           # Discard changes to file

# Remote Operations
git remote add origin URL          # Add remote repository
git remote set-url origin URL      # Change remote URL
git push -u origin main            # Push and set upstream
```

---

## Additional Resources

- [GitHub Documentation](https://docs.github.com/)
- [Git Documentation](https://git-scm.com/doc)
- [GitHub SSH Setup Guide](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [Git Cheat Sheet](https://education.github.com/git-cheat-sheet-education.pdf)

---

## Your Current Setup

**Repository:** `https://github.com/ranjeetapegu/RSnowflake_Container_Runtime_for_ML.git`

**Remote Configuration:**
```bash
git remote -v
# origin  git@github.com:ranjeetapegu/RSnowflake_Container_Runtime_for_ML.git (fetch)
# origin  git@github.com:ranjeetapegu/RSnowflake_Container_Runtime_for_ML.git (push)
```

**SSH Key:** `~/.ssh/id_ed25519_personal`

**Email:** `ranjeetapegu@gmail.com`

---

*Last Updated: November 12, 2025*

