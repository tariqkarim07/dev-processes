# 🚀 Laravel AWS Deployment via GitHub Actions

This project is automatically deployed to an AWS EC2 instance using **GitHub Actions**. This guide explains how to set up the pipeline from a **Windows machine** using SSH.

---

## 📁 Prerequisites

- Laravel app hosted on AWS EC2
- Access to AWS EC2 terminal (via EC2 Connect or previous session)
- GitHub repository for your Laravel app
- Git installed on your local machine (Git Bash or WSL recommended)
- SSH key pair (newly generated)

---

## 🔐 Step 1: Add Your SSH Public Key to EC2 Instance

1.1 **Copy your public key**  
   On Windows using Git Bash or WSL: (local machine)
   ```bash
   cat ~/.ssh/id_rsa.pub
   ```

1.2. **Add to AWS Authorized Keys**
1. Use EC2 Instance Connect or another SSH method to get into the AWS instance.

2. Run
```bash
 nano ~/.ssh/authorized_keys
```
3. Paste the public key you copied.

4. Save and exit (Ctrl + X, then Y, then Enter).

Now you can SSH from Windows using a tool like Git Bash:

```bash
ssh -i ~/.ssh/id_rsa ec2-user@your-aws-public-ip
```
Or with PuTTY, using the corresponding .ppk key.



---

## 🔧 Step 2: Add GitHub Repository Secrets

On GitHub:

1. Go to **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret** and add:

| Name              | Value                                                   |
|-------------------|---------------------------------------------------------|
| `SSH_PRIVATE_KEY` | Your private key (contents of `~/.ssh/id_rsa`)         |
| `HOST`            | Your EC2 public IP or DNS                              |
| `USERNAME`        | e.g., `ec2-user` or `ubuntu`                           |
| `DEPLOY_PATH`     | e.g., `/var/www/html/laravel`                          |

---

## ⚙️ Step 3: Create GitHub Actions Workflow

Create a new file in your project:
```
.github/workflows/deploy.yml
```

Paste this and edit according to requirement

```yaml
name: Deploy Laravel App to AWS

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up SSH
      run: |
        mkdir -p ~/.ssh
        echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
        chmod 600 ~/.ssh/id_rsa
        ssh-keyscan -H ${{ secrets.HOST }} >> ~/.ssh/known_hosts

    - name: Deploy via SSH
      run: |
        ssh ${{ secrets.USERNAME }}@${{ secrets.HOST }} << 'EOF'
          cd ${{ secrets.DEPLOY_PATH }}
          git pull origin main
          composer install --no-interaction --prefer-dist --optimize-autoloader
          php artisan migrate --force
          php artisan config:cache
          php artisan route:cache
          php artisan view:cache
        EOF

```



