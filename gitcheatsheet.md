### Configure
```bash
git config --global user.name "[name]"
git config --global user.email "[email]"
git config --global color.ui auto
```

### Create Repo
```bash
git init [project-name]
git clone [url]
```

### Make Changes
```bash
git status
git diff
git add [file]
git diff --staged
git reset [file]
git commit -m "[message]"
```

### Group Changes
```bash
git branch
git branch [branch-name]
git checkout [branch-name]
git merge [branch-name]
git branch -d [branch-name]
```

### Refactor Filenames
```bash
git rm [file]
git rm --cached [file]
git mv [file-original] [file-renamed]
```

### Suppress Tracking
```bash
git ls-files --other --ignored --exclude-standard
```

### Save Fragments
```bash
git stash
git stash pop
git stash list
git stash drop
```

### Review History
```bash
git log
git log --follow [file]
git diff [first-branch]...[second-branch]
git show [commit]
```

### Redo Commits
```bash
git reset [commit]
git reset --hard [commit]
```

### Synchronize Changes
```bash
git fetch [bookmark]
git merge [bookmark]/[branch]
git push [alias] [branch]
git pull
```

