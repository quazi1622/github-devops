It covers the full loop: Git commands, Server setup, Runner configuration, and the CD Pipeline.

Markdown

# 🚀 CI/CD Cheatsheet: Static Site on DigitalOcean
**Stack:** GitHub Actions (Self-Hosted) + Nginx + nip.io

---

## 1. Git Workflow (The Daily Cycle)
How to save changes and trigger the pipeline.

```bash
# 1. Stage changes (Put files in the envelope)
git add .

# 2. Commit (Seal the envelope)
git commit -m "Update homepage content"

# 3. Push (Send to GitHub -> Triggers Action)
git push origin main
2. Server Setup (DigitalOcean Droplet)
One-time setup to prepare the server to host websites.

Bash

# SSH into the server
ssh root@<YOUR_DROPLET_IP>

# Update system
apt-get update

# Install Nginx Web Server
apt-get install -y nginx

# Allow traffic through firewall
ufw allow 'Nginx Full'

# (Optional) Check if Nginx is running
systemctl status nginx
3. GitHub Runner Installation (One-Time)
Connecting the Droplet to GitHub.

Pre-requisite: Go to Repo Settings > Actions > Runners > New Self-Hosted Runner.

Bash

# A. Create folder
mkdir actions-runner && cd actions-runner

# B. Download (Example - version may vary, check GitHub for latest link)
curl -o actions-runner-linux-x64-xxxx.tar.gz -L https://...
tar xzf ./actions-runner-linux-x64-xxxx.tar.gz

# C. CRITICAL: Allow running as Root
export RUNNER_ALLOW_RUNASROOT=1

# D. Configure (Use token from GitHub UI)
./config.sh --url [https://github.com/](https://github.com/)<USER>/<REPO> --token <TOKEN>
# (Press Enter through all prompts)

# E. Install as a Background Service (Do NOT use ./run.sh)
./svc.sh install
./svc.sh start

# Check status (Should say "Active: active (running)")
./svc.sh status
4. The Pipeline (.github/workflows/deploy.yml)
The instructions for the robot.

YAML

name: Deploy to Droplet

on:
  push:
    branches: ["main"]

jobs:
  deploy:
    runs-on: self-hosted
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4
      
      - name: Deploy to Nginx
        run: |
          # Copy index.html to default Nginx folder
          # -f forces overwrite if file exists
          cp -f index.html /var/www/html/index.html
          echo "Deployed successfully!"
5. Accessing Your Site (nip.io)
Since you don't have a custom domain yet, use nip.io for wildcard DNS mapping.

Your IP: 123.45.67.89 (Example)

Your URL: http://123.45.67.89.nip.io