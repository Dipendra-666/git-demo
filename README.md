# GitHub Actions CI/CD – Automatic Deployment to Dev Server

## Project Overview

This repository demonstrates a **basic Continuous Deployment (CD) pipeline** using **GitHub Actions**.

**Goal**: Every time I push code to the `dev` branch, GitHub automatically connects to my development server via SSH and runs a script that updates the code on the server.

**Real-world use case**: This is how companies automatically deploy changes to test/staging servers without manual work (no FTP, no manual git pull).

## Technologies Used

- Git & GitHub (repository + branches)
- GitHub Actions (CI/CD)
- SSH + SSH keys
- Bash scripting
- Ubuntu in VirtualBox (used as dev server)

## Project Folder Structure
git-demo/
├── .github/
│   └── workflows/
│       └── deploy.yml                # GitHub Actions workflow file
├── cicd/
│   └── dev                           # Deployment script that runs on server
├── dev-deploy-test.txt               # Test file to verify deployment
├── README.md                         # This file
└── (other test files I created)
text## Step-by-Step What I Did (with commands & explanations)

### 1. Created GitHub Repository & dev branch

- Created repo on GitHub: `https://github.com/Dipendra-666/git-demo`
- On my local machine (Windows Git Bash):

```bash
git clone https://github.com/Dipendra-666/git-demo.git
cd git-demo
git checkout -b dev
git push -u origin dev
Why?
git checkout -b dev → creates and switches to new branch dev
git push -u origin dev → pushes the branch to GitHub and sets upstream tracking
2. Set up the dev server (Ubuntu VM in VirtualBox)

Installed Ubuntu in VirtualBox
On the VM (server):

Bashsudo apt update
sudo apt install git -y
mkdir -p /var/www/dev-wordpress
cd /var/www/dev-wordpress
git clone git@github.com:Dipendra-666/git-demo.git .
Why?
git clone ... . → clones the repo directly into the current folder (not creating subfolder)
3. Generated SSH keys on the server VM
Bashssh-keygen -t ed25519 -C "dipendra-server-key"

Press Enter for defaults (no passphrase for learning)
Public key created: ~/.ssh/id_ed25519.pub

Why ed25519?
Modern, secure, fast key type (better than old RSA)
4. Added public key as Deploy Key in GitHub

Copied public key:

Bashcat ~/.ssh/id_ed25519.pub

Pasted the whole line (ssh-ed25519 AAAAC3... dipendra-server-key)
into GitHub → repo → Settings → Deploy keys → Add deploy key
(Allow write access: unchecked – only read/pull needed)

Why?
So the server can do git pull without password
5. Created the deployment script on server
Bashcd /var/www/dev-wordpress
mkdir -p cicd
nano cicd/dev
Pasted this content:
Bash#!/bin/bash
set -e

BRANCH="dev"

cd /var/www/dev-wordpress
git add .
git stash
git checkout $BRANCH
git pull origin $BRANCH
Then:
Bashchmod +x cicd/dev
Why these commands?

set -e → stop script if any command fails
git stash → save any uncommitted changes temporarily
git checkout $BRANCH → switch to dev branch
git pull origin $BRANCH → get latest code from GitHub

Tested manually:
Bash./cicd/dev
6. Created GitHub Actions workflow
Created file .github/workflows/deploy.yml and pushed to dev branch:
YAMLname: Deploy to Dev Server

on:
  push:
    branches:
      - dev

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Execute remote script via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.DEV_HOST }}
          username: ${{ secrets.DEV_USER }}
          key: ${{ secrets.DEV_SSH_KEY }}
          port: ${{ secrets.DEV_PORT }}
          script: |
            chmod +x cicd/dev
            ./cicd/dev
Why this structure?

on: push: branches: - dev → only run when pushing to dev branch
appleboy/ssh-action → popular action to SSH into server and run commands
Uses GitHub Secrets to keep IP/username/key/port safe

7. Added GitHub Secrets
In repo → Settings → Secrets and variables → Actions → New repository secret

DEV_HOST → 192.168.1.66 (VM IP in bridged mode)
DEV_USER → dipendra
DEV_PORT → 22
DEV_SSH_KEY → full private key (cat ~/.ssh/id_ed25519 on VM)

8. Testing

Added test lines to dev-deploy-test.txt → commit & push to dev
Watched GitHub Actions tab (runs automatically)
Checked server: cat dev-deploy-test.txt → changes appeared

What I Learned

How GitHub Actions triggers on git push
Secure SSH setup (keys, deploy keys, secrets)
Writing deployment scripts in bash
VirtualBox networking (NAT vs Bridged Adapter)
Debugging CI/CD issues (connection refused, timeout, permissions)
Importance of testing manually before automating

This project helped me understand real DevOps pipelines used in companies.
Feel free to look at commit history and Actions runs for full details.
