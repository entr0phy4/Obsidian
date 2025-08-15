# Git

## Overview
Git is a distributed version control system designed to handle projects of any size with speed and efficiency. It tracks changes in source code during software development and enables multiple developers to work on the same project.

## Basic Commands

### Repository Setup
```bash
# Initialize new repository
git init

# Clone existing repository
git clone https://github.com/user/repo.git
git clone git@github.com:user/repo.git  # SSH

# Add remote repository
git remote add origin https://github.com/user/repo.git

# View remotes
git remote -v
```

### Basic Workflow
```bash
# Check status
git status

# Add files to staging
git add filename.txt
git add .                    # Add all files
git add -A                   # Add all changes

# Commit changes
git commit -m "Commit message"
git commit -am "Add and commit"  # Add and commit tracked files

# Push to remote
git push origin main
git push -u origin main      # Set upstream

# Pull from remote
git pull origin main
git pull                     # From tracked branch
```

### Branching
```bash
# Create and switch to new branch
git checkout -b feature-branch
git switch -c feature-branch  # Modern syntax

# Switch branches
git checkout main
git switch main

# List branches
git branch                   # Local branches
git branch -a                # All branches
git branch -r                # Remote branches

# Delete branch
git branch -d feature-branch
git branch -D feature-branch  # Force delete

# Merge branch
git checkout main
git merge feature-branch

# Push new branch
git push -u origin feature-branch
```

## Advanced Operations

### Stashing
```bash
# Stash changes
git stash
git stash save "Work in progress"

# List stashes
git stash list

# Apply stash
git stash apply
git stash apply stash@{0}

# Pop stash (apply and remove)
git stash pop

# Drop stash
git stash drop stash@{0}
```

### History and Logs
```bash
# View commit history
git log
git log --oneline
git log --graph --oneline --all

# Show changes
git show HEAD
git show commit-hash

# View file history
git log -p filename.txt

# Search commits
git log --grep="search term"
git log --author="author name"
```

### Undoing Changes
```bash
# Unstage file
git reset HEAD filename.txt

# Discard working directory changes
git checkout -- filename.txt
git restore filename.txt      # Modern syntax

# Reset to previous commit
git reset --soft HEAD~1       # Keep changes staged
git reset --mixed HEAD~1      # Keep changes in working directory
git reset --hard HEAD~1       # Discard all changes

# Revert commit (creates new commit)
git revert commit-hash
```

## Configuration

### Global Configuration
```bash
# Set user information
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default editor
git config --global core.editor "code --wait"

# Set default branch name
git config --global init.defaultBranch main

# View configuration
git config --list
git config user.name
```

### SSH Key Setup
```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"

# Add to SSH agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Test SSH connection
ssh -T git@github.com
```

## Collaboration Workflows

### Feature Branch Workflow
```bash
# 1. Create feature branch
git checkout -b feature/user-authentication

# 2. Work on feature
git add .
git commit -m "Add user registration"

# 3. Push feature branch
git push -u origin feature/user-authentication

# 4. Create pull request (on GitHub/GitLab)

# 5. After approval, merge and cleanup
git checkout main
git pull origin main
git branch -d feature/user-authentication
```

### Gitflow Workflow
```bash
# Main branches: main (production), develop (integration)
# Feature branches: feature/feature-name
# Release branches: release/version-number
# Hotfix branches: hotfix/issue-name

# Start feature
git checkout develop
git checkout -b feature/new-feature

# Finish feature
git checkout develop
git merge feature/new-feature
git branch -d feature/new-feature
```

## Git Hooks

### Pre-commit Hook Example
```bash
#!/bin/sh
# .git/hooks/pre-commit

# Run tests before commit
npm test
if [ $? -ne 0 ]; then
    echo "Tests failed. Commit aborted."
    exit 1
fi

# Check code formatting
npm run lint
if [ $? -ne 0 ]; then
    echo "Linting failed. Commit aborted."
    exit 1
fi
```

### Post-receive Hook (Server)
```bash
#!/bin/sh
# Deploy after push to main branch

if [ "$1" = "refs/heads/main" ]; then
    cd /var/www/html
    git pull origin main
    # Restart services, run migrations, etc.
fi
```

## Common Scenarios

### Resolving Merge Conflicts
```bash
# When merge conflict occurs
git status                    # See conflicted files

# Edit conflicted files, then:
git add resolved-file.txt
git commit -m "Resolve merge conflict"

# Or abort merge
git merge --abort
```

### Interactive Rebase
```bash
# Rebase last 3 commits
git rebase -i HEAD~3

# In editor, choose action for each commit:
# pick = use commit
# reword = edit commit message
# edit = edit commit
# squash = combine with previous commit
# drop = remove commit
```

### Cherry-picking
```bash
# Apply specific commit to current branch
git cherry-pick commit-hash

# Cherry-pick range
git cherry-pick start-hash..end-hash
```

## Best Practices

### Commit Messages
```bash
# Good commit message format
type(scope): subject

body (optional)

footer (optional)

# Examples:
feat(auth): add user registration endpoint
fix(ui): resolve button alignment issue
docs(readme): update installation instructions
```

### .gitignore Examples
```bash
# Node.js
node_modules/
npm-debug.log*
.env

# Python
__pycache__/
*.pyc
.venv/

# IDE
.vscode/
.idea/

# OS
.DS_Store
Thumbs.db

# Build outputs
dist/
build/
```

### Repository Structure
```bash
project/
├── .git/
├── .gitignore
├── README.md
├── src/
├── tests/
├── docs/
├── config/
└── scripts/
```

## Troubleshooting

### Common Issues
```bash
# Undo last commit (keep changes)
git reset --soft HEAD~1

# Change last commit message
git commit --amend -m "New message"

# Remove file from Git but keep locally
git rm --cached filename.txt

# View what changed
git diff
git diff --staged
git diff commit1 commit2

# Find when bug was introduced
git bisect start
git bisect bad          # Current commit is bad
git bisect good v1.0    # Version 1.0 was good
# Git will checkout commits to test
git bisect good/bad     # Mark each test
git bisect reset        # When done
```

### Recovery Commands
```bash
# Find lost commits
git reflog

# Recover deleted branch
git checkout -b recovered-branch commit-hash

# Reset to remote state
git fetch origin
git reset --hard origin/main
```

## Related Topics
- [[GitHub]] - Git hosting platform
- [[GitLab]] - DevOps platform
- [[Version control]]
- [[CI/CD pipelines]]
- [[Code collaboration]]
