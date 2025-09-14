# GIT & VERSION CONTROL - PHỎNG VẤN

## 📋 MỤC LỤC
1. [Git Basics](#git-basics)
2. [Branching & Merging](#branching--merging)
3. [Advanced Git Commands](#advanced-git-commands)
4. [Git Workflow](#git-workflow)
5. [Collaboration & Team](#collaboration--team)
6. [Câu hỏi phỏng vấn thực tế](#câu-hỏi-phỏng-vấn-thực-tế)
7. [Troubleshooting](#troubleshooting)

---

## 🎯 GIT BASICS

### **Core Concepts**
```bash
# Git là gì?
- Distributed Version Control System
- Track changes in source code
- Collaborate với nhiều developers
- Backup và history của project

# Git vs SVN/CVS
- Distributed vs Centralized
- Offline working
- Faster operations
- Better branching and merging
```

### **Git Workflow Areas**
```bash
Working Directory → Staging Area → Repository
     (git add)        (git commit)

# Working Directory: Thư mục làm việc hiện tại
# Staging Area (Index): Nơi chuẩn bị commit
# Repository: Nơi lưu trữ commits
```

---

## 🌿 BRANCHING & MERGING

### **Branch Operations**
```bash
# Tạo và chuyển branch
git branch feature-login
git checkout feature-login
# hoặc
git checkout -b feature-login

# Merge strategies
git merge feature-login        # Fast-forward merge
git merge --no-ff feature-login  # Three-way merge
git rebase main               # Rebase onto main
```

### **Merge vs Rebase**
```bash
# Merge: Tạo merge commit
A---B---C main
     \
      D---E feature
           \
            F (merge commit)

# Rebase: Linear history
A---B---C---D'---E' main (feature rebased)
```

---

## 🔧 ADVANCED GIT COMMANDS

### **Git Reset Types**
```bash
# Soft reset: Giữ changes trong staging
git reset --soft HEAD~1

# Mixed reset (default): Unstage changes
git reset HEAD~1

# Hard reset: Xóa tất cả changes
git reset --hard HEAD~1
```

### **Git Stash**
```bash
# Lưu tạm changes
git stash
git stash push -m "WIP: feature login"

# Apply stash
git stash pop      # Apply và xóa stash
git stash apply    # Apply nhưng giữ stash

# List và manage stashes
git stash list
git stash drop stash@{0}
```

### **Cherry Pick**
```bash
# Lấy specific commit từ branch khác
git cherry-pick <commit-hash>

# Multiple commits
git cherry-pick A B C

# Range of commits
git cherry-pick A..C
```

---

## 🔄 GIT WORKFLOW

### **GitFlow**
```bash
# Main branches
- main/master: Production code
- develop: Integration branch

# Supporting branches
- feature/*: New features
- release/*: Prepare releases  
- hotfix/*: Emergency fixes

# Example workflow
git checkout -b feature/user-auth develop
# ... work on feature
git checkout develop
git merge feature/user-auth
git branch -d feature/user-auth
```

### **GitHub Flow**
```bash
# Simplified workflow
1. Create branch từ main
2. Make changes và commits
3. Open Pull Request
4. Review và merge
5. Delete branch

git checkout -b feature/new-feature
# ... work
git push origin feature/new-feature
# Create PR on GitHub
```

---

## 👥 COLLABORATION & TEAM

### **Remote Operations**
```bash
# Add remote
git remote add origin <url>
git remote add upstream <upstream-url>

# Fetch vs Pull
git fetch origin          # Download refs và objects
git pull origin main      # Fetch + merge
git pull --rebase origin main  # Fetch + rebase

# Push strategies
git push origin main
git push -u origin feature-branch  # Set upstream
git push --force-with-lease origin main  # Safe force push
```

### **Conflict Resolution**
```bash
# Merge conflicts
<<<<<<< HEAD
Your changes
=======
Their changes
>>>>>>> branch-name

# Resolve conflicts
1. Edit files manually
2. git add <resolved-files>
3. git commit

# Abort merge
git merge --abort
```

---

## ❓ CÂU HỎI PHỎNG VẤN THỰC TẾ

### **Câu hỏi cơ bản**

#### **1. "Git là gì và tại sao dùng Git?"**
**Trả lời:**
```
Git là distributed version control system giúp:
- Track changes trong source code
- Collaborate với team
- Maintain history và backup
- Support branching và merging
- Work offline
- Fast và efficient
```

#### **2. "Sự khác biệt giữa git pull và git fetch?"**
**Trả lời:**
```bash
# git fetch: Download data nhưng không merge
git fetch origin
# Remote refs được update, working directory không thay đổi

# git pull: Fetch + merge automatically  
git pull origin main
# Tương đương với:
git fetch origin
git merge origin/main
```

#### **3. "Giải thích git merge và git rebase?"**
**Trả lời:**
```bash
# Git merge: Kết hợp branches, preserve history
- Tạo merge commit
- Non-destructive operation
- Preserve branch context

# Git rebase: Re-apply commits lên base branch
- Linear history
- Cleaner project history
- Rewrites commit history (dangerous for shared branches)

# Khi nào dùng gì:
- Merge: Cho feature branches vào main
- Rebase: Clean up local commits trước khi merge
```

### **Câu hỏi nâng cao**

#### **4. "Làm thế nào để undo commit đã push?"**
**Trả lời:**
```bash
# Nếu commit chưa được share:
git reset --hard HEAD~1
git push --force-with-lease

# Nếu commit đã được share (safer):
git revert <commit-hash>
git push origin main

# Revert multiple commits:
git revert HEAD~3..HEAD
```

#### **5. "Xử lý merge conflicts như thế nào?"**
**Trả lời:**
```bash
# Steps to resolve:
1. git status (see conflicted files)
2. Open conflicted files
3. Resolve conflicts manually
4. git add <resolved-files>
5. git commit

# Tools hỗ trợ:
git mergetool
# hoặc VSCode, IntelliJ built-in merge tools

# Prevent conflicts:
- Pull regularly
- Small, frequent commits
- Clear communication with team
```

#### **6. "Git stash hoạt động như thế nào?"**
**Trả lời:**
```bash
# Use cases:
- Switch branches với uncommitted changes
- Pull updates mà có local changes
- Temporarily save work in progress

# Commands:
git stash                    # Save current changes
git stash pop               # Apply và remove stash
git stash list              # List all stashes
git stash apply stash@{1}   # Apply specific stash
git stash drop stash@{0}    # Delete specific stash
```

### **Câu hỏi tình huống**

#### **7. "Team member accidentally force pushed lên main branch, làm gì?"**
**Trả lời:**
```bash
# Immediate actions:
1. Communicate với team ngay lập tức
2. Check git reflog để find lost commits:
   git reflog origin/main
   
3. Restore lost commits:
   git reset --hard <previous-commit-hash>
   git push --force-with-lease
   
4. Prevention measures:
   - Enable branch protection rules
   - Disable force push trên main
   - Use pull requests workflow
   - Set up pre-receive hooks
```

#### **8. "Làm sao để maintain clean commit history?"**
**Trả lời:**
```bash
# Strategies:
1. Interactive rebase trước khi merge:
   git rebase -i HEAD~3
   # squash, reword, reorder commits
   
2. Squash merge for feature branches:
   git merge --squash feature-branch
   
3. Conventional commit messages:
   feat: add user authentication
   fix: resolve login redirect issue
   docs: update API documentation
   
4. Small, focused commits
5. Descriptive commit messages
6. Use git commit --amend for last commit fixes
```

---

## 🔧 TROUBLESHOOTING

### **Common Issues & Solutions**

#### **1. Accidental commit to wrong branch**
```bash
# Solution 1: Cherry-pick to correct branch
git checkout correct-branch
git cherry-pick <commit-hash>
git checkout wrong-branch
git reset --hard HEAD~1

# Solution 2: Create branch from current commit
git branch new-branch-name
git reset --hard HEAD~1
```

#### **2. Lost commits after reset**
```bash
# Use reflog to recover
git reflog
git reset --hard <commit-hash>

# Or create new branch from lost commit
git branch recovery-branch <commit-hash>
```

#### **3. Large file in history**
```bash
# Remove file from entire history
git filter-branch --tree-filter 'rm -rf large-file.zip' HEAD

# Or use git filter-repo (modern way)
git filter-repo --path large-file.zip --invert-paths
```

---

## 📚 BEST PRACTICES

### **1. Commit Messages**
```bash
# Good commit messages:
feat: add user login functionality
fix: resolve memory leak in image processing
docs: update installation guide
refactor: extract utility functions
test: add unit tests for authentication

# Bad commit messages:
"fix bug"
"updates"
"work in progress"
"asdfgh"
```

### **2. Branch Naming**
```bash
# Good branch names:
feature/user-authentication
bugfix/login-redirect-issue  
hotfix/security-vulnerability
release/v1.2.0

# Bad branch names:
"john-work"
"temp"
"fix"
"branch1"
```

### **3. Gitignore Strategy**
```bash
# Always ignore:
node_modules/
.env
.DS_Store
*.log
dist/
build/

# Language specific:
# Python
__pycache__/
*.pyc

# Java  
*.class
target/

# IDE
.vscode/
.idea/
```

---

## 🎯 TIPS PHỎNG VẤN

### **1. Demonstrate Understanding**
- Giải thích concepts với examples
- Show commands thực tế
- Mention trade-offs và best practices

### **2. Common Scenarios**
- Collaboration conflicts
- History cleanup
- Branch management
- Release workflows

### **3. Show Experience**
- Mention git workflows đã sử dụng
- Tools integration (IDE, GitHub/GitLab)
- Team processes và conventions

---

**Lưu ý:** Git là fundamental skill cho mọi developer. Hiểu sâu về Git workflow và troubleshooting sẽ thể hiện kinh nghiệm và professionalism của bạn!
