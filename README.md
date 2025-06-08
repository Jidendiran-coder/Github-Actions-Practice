# Github-Actions-Practice

Perfect! Let’s go **step by step** to set up this GitHub Actions-based EC2 HTML deployment with backup and Nginx restart. I’ll wait for your confirmation after each step before moving on.

---

## ✅ Step 1: Prepare Your EC2 Instance

Please confirm the following are set up on your EC2 instance:

1. **Instance is running and accessible** via its **public IP or DNS**.
2. **Nginx is installed** and serving files from a known path (e.g., `/var/www/html`).
3. **You can SSH into the instance** from your local terminal like this:

   ```bash
   ssh -i /path/to/your-key.pem ec2-user@your-ec2-public-ip
   ```

📌 Replace `/path/to/your-key.pem`, `ec2-user`, and `your-ec2-public-ip` with actual values.

---

### ✅ Tasks for you:

* Try SSH-ing into your EC2 instance from your terminal.
* Run this command on EC2 to confirm Nginx is installed and working:

```bash
sudo systemctl status nginx
```

1. Can you successfully SSH into your EC2?
2. Is Nginx installed and running?

Once done, we’ll move to Step 2.

---

## ✅ Step 2: Prepare Your SSH Key for GitHub

We now need to take your **EC2 private key** (usually a `.pem` file) and store it securely in your GitHub repository as a secret.

---

### 🔐 Instructions:

1. Open your `.pem` file (e.g., `my-ec2-key.pem`) in a text editor (Notepad++ or VS Code).

2. Copy the entire content — it should start with:

   ```
   -----BEGIN OPENSSH PRIVATE KEY-----
   ```

   or

   ```
   -----BEGIN RSA PRIVATE KEY-----
   ```

3. Go to your GitHub **repository** (not your GitHub profile).

4. Navigate to:

   ```
   Settings → Secrets and variables → Actions → New repository secret
   ```

5. Add a new secret:

   * **Name**: `EC2_SSH_KEY`
   * **Value**: Paste your private key content

---


## ✅ Step 3: Add Remaining GitHub Secrets

We now need to add 3 more secrets:

| Secret Name       | Description                                               | Example                                        |
| ----------------- | --------------------------------------------------------- | ---------------------------------------------- |
| `EC2_HOST`        | Your EC2 instance's public IP or DNS                      | `ec2-3-91-123-45.compute.amazonaws.com`        |
| `EC2_USER`        | Username to SSH into EC2                                  | `ec2-user` (Amazon Linux) or `ubuntu` (Ubuntu) |
| `EC2_TARGET_PATH` | The full path where `index.html` should go (Nginx folder) | `/var/www/html`                                |

---

### 🛠️ Instructions:

1. Go to your GitHub repo → **Settings → Secrets and variables → Actions**
2. Click **“New repository secret”** and add:

   * **Secret Name:** `EC2_HOST`
     **Secret Value:** your EC2 public DNS or IP

   * **Secret Name:** `EC2_USER`
     **Secret Value:** `ec2-user` or `ubuntu` (depending on your EC2 image)

   * **Secret Name:** `EC2_TARGET_PATH`
     **Secret Value:** `/var/www/html`

---

Great job! ✅ All secrets are in place.

---

## ✅ Step 4: Add the `main.yml` GitHub Actions Workflow

We’ll now add the deployment workflow file to your GitHub repo.

---

### 📁 Folder Structure in Your Repo

You need to place the workflow in:

```
.ygithub/workflows/main.yml
```

> If the `.github/workflows` folder doesn’t exist, you’ll need to create it.

---

### 📄 `main.yml` — Paste This:

Create a file called `main.yml` in `.github/workflows/` with the following content:

```yaml
name: Deploy HTML to EC2 on Push

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Set up SSH key
        run: |
          echo "${{ secrets.EC2_SSH_KEY }}" > private_key.pem
          chmod 600 private_key.pem

      - name: Backup old HTML & deploy new one
        run: |
          REMOTE_USER="${{ secrets.EC2_USER }}"
          REMOTE_HOST="${{ secrets.EC2_HOST }}"
          TARGET_PATH="${{ secrets.EC2_TARGET_PATH }}"
          FILE_NAME="index.html"

          echo "Backing up old file on EC2..."
          ssh -i private_key.pem -o StrictHostKeyChecking=no $REMOTE_USER@$REMOTE_HOST "sudo cp $TARGET_PATH/$FILE_NAME $TARGET_PATH/${FILE_NAME}.bak || echo 'No previous file to back up.'"

          echo "Copying new file to EC2..."
          scp -i private_key.pem -o StrictHostKeyChecking=no ./$FILE_NAME $REMOTE_USER@$REMOTE_HOST:/tmp/

          echo "Moving file into place..."
          ssh -i private_key.pem -o StrictHostKeyChecking=no $REMOTE_USER@$REMOTE_HOST "sudo mv /tmp/$FILE_NAME $TARGET_PATH/$FILE_NAME"

          echo "Restarting Nginx..."
          ssh -i private_key.pem -o StrictHostKeyChecking=no $REMOTE_USER@$REMOTE_HOST "sudo systemctl restart nginx"

      - name: Clean up private key
        run: rm -f private_key.pem
```

---

### ✅ Tasks for you:

1. Create folder `.github/workflows/` if it doesn’t exist.
2. Save the above file inside it as `main.yml`.
3. Ensure your repo has a file named `index.html` in the root.

Once this is done, and we’ll **trigger a push to test** the deployment.

✅ The workflow is now in place.

---

## ✅ Step 5: Trigger the Workflow

To test the automation:

### 🧪 Trigger a Push

1. Open your repo locally or via GitHub web editor.
2. Make a small change to the `index.html` file (even adding a comment is fine).
3. Commit and push the change to the `main` branch.

> This will trigger the GitHub Actions workflow we just created.

---

### 🔍 Monitor the Workflow

1. Go to your GitHub repo.
2. Click on the **"Actions"** tab.
3. You should see a new workflow run under **"Deploy HTML to EC2 on Push"**.
4. Click it to monitor the job steps (checkout, backup, copy, restart).

---


