## Multi-Laptop Git Workflow (Lenovo + HP)

This guide explains how to keep your GitHub code synchronized while working on multiple laptops using the same GitHub account.


### One-time Setup (on both laptops)

```bash
git config --global user.name "imon-n"
git config --global user.email "imon.eeecu@gmail.com"
git clone https://github.com/imon-n/REPO.git
cd REPO
```


### Before Starting Work (Mandatory)

Always pull the latest code before coding:

```bash
git pull origin main
```

### After Finishing Work (on current laptop)

```bash
git status
git add .
git commit -m "your commit message"
git push origin main
```


### When Switching to Another Laptop

On the other laptop, update the code:

```bash
git pull origin main
```


### If a Merge Conflict Occurs

```bash
# fix conflicted files manually
git add .
git commit -m "resolve conflict"
git push origin main
```


### Workflow Summary

**Pull before coding → Commit & Push after work → Pull on the other laptop**
