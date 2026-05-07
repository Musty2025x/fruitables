# Jenkins CI/CD Pipeline for Static Website Deployment on AWS EC2

## Real-World Scenario

> You are a Junior DevOps Engineer at **Your Techie Hub**. The frontend team pushes updates to a GitHub repo. Your job is to automatically pull updates, deploy to a live web server, and ensure the website updates without manual intervention.

---

## Architecture

```
Developer
    │
    │  git push
    ▼
GitHub Repository (forked: Musty2025x/fruitables)
    │
    │  Webhook → triggers on push
    ▼
Jenkins (port 8080)
running on AWS EC2 — Ubuntu 22.04
    │
    │  Execute shell script
    ▼
Apache Web Server
/var/www/html/myapp
    │
    ▼
http://your-domain.com/myapp  (HTTPS secured)
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Source Control | GitHub (forked repository) |
| CI/CD Server | Jenkins (Freestyle Job) |
| Cloud Hosting | AWS EC2 — Ubuntu 22.04 — t3.small |
| Web Server | Apache2 |
| SSL | Let's Encrypt via Certbot |
| Domain | Any registrar (Namecheap, GoDaddy, Qservers, Cloudflare) |

---

## Prerequisites

- AWS account
- GitHub account

---

## Phase 1 — GitHub Repository Setup

### Step 1 — Fork the Repository

1. Go to [github.com/Musty2025x/fruitables](https://github.com/Musty2025x/fruitables.git)
2. Click **Fork** → **Create fork**

This gives you your own copy of the static website that Jenkins will deploy from.

---

## Phase 2 — Launch EC2 Server

### Step 2 — Create EC2 Instance

1. Go to **AWS Console → EC2 → Launch Instance**
2. Configure:

| Field | Value |
|---|---|
| AMI | Ubuntu 22.04 LTS |
| Instance Type | t3.small (minimum — Jenkins requires >1GB RAM) |
| Key Pair | Create new → download `.pem` file |
| Security Group | Allow ports 22, 80, 443 |

3. Click **Launch Instance**

## screenshot of AWS EC2 launch instance configuration showing Ubuntu 22.04, t3.small, key pair, and security group settings
> ![alt text](<asset/Screenshot 2026-05-07 095359.png>)
> ![alt text](<asset/Screenshot 2026-05-07 095337.png>)
> ![alt text](<asset/Screenshot 2026-05-07 095527.png>)

### Step 3 — Configure Security Group (Add Jenkins Port)

Steps 2 already opened ports 22, 80, and 443. Now add port 8080 for Jenkins:

1. Go to **EC2 → Security Groups → your instance's security group**
2. Click **Edit inbound rules → Add rule**
3. Add:

| Type | Protocol | Port | Source |
|---|---|---|---|
| SSH | TCP | 22 | Anywhere (0.0.0.0/0) |
| HTTP | TCP | 80 | Anywhere (0.0.0.0/0) |
| HTTPS | TCP | 443 | Anywhere (0.0.0.0/0) |
| Custom TCP | TCP | 8080 | Anywhere (0.0.0.0/0) |

4. Click **Save rules**

## screenshot of AWS EC2 security group inbound rules with ports 22, 80, 443, and 8080 open to
> ![alt text](<asset/Screenshot 2026-05-07 095804.png>)


### Step 4 — Connect via SSH

```bash
# Make sure your key file has correct permissions first
chmod 400 your-key.pem

# Connect
ssh -i your-key.pem ubuntu@YOUR_PUBLIC_IP

# Or with full path
ssh -i "/path/to/your-key.pem" ubuntu@YOUR_PUBLIC_IP
```

> **Tip:** In the AWS Console, click **Connect** on your instance to get the exact SSH command pre-filled with your public IP.

## Screenshot of AWS EC2 instance connection details showing the SSH command to connect to the instance
> ![alt text](<asset/Screenshot 2026-05-07 100403.png>)
> ![alt text](<asset/Screenshot 2026-05-07 100702.png>)

---

## Phase 3 — Install Required Software

### Step 5 — Update the server

```bash
sudo apt update && sudo apt upgrade -y
```
## screenshot of terminal showing sudo apt update and upgrade commands with progress
> ![alt text](<asset/Screenshot 2026-05-07 100754.png>)

### Step 6 — Install Java (required by Jenkins)

```bash
sudo apt install fontconfig openjdk-21-jre -y
java -version
```

Expected output:
```
openjdk version "21.0.x" ...
```
## screenshot of terminal showing Java installation and version output
> ![alt text](<asset/Screenshot 2026-05-07 100951.png>)
> ![alt text](<asset/Screenshot 2026-05-07 101049.png>)

### Step 7 — Install Jenkins

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins -y
```
## screenshot of terminal showing Jenkins installation commands and progress
> ![alt text](<asset/Screenshot 2026-05-07 101353.png>)
> ![alt text](<asset/Screenshot 2026-05-07 101454.png>)
> ![alt text](<asset/Screenshot 2026-05-07 101530.png>)

### Step 8 — Start Jenkins

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Verify it's running
sudo systemctl status jenkins
```

Expected:
```
● jenkins.service - Jenkins Continuous Integration Server
     Active: active (running)
```
## screenshot of terminal showing Jenkins service status with active (running)
> ![alt text](<asset/Screenshot 2026-05-07 101650.png>)


### Step 9 — Access Jenkins in the browser

```
http://YOUR_PUBLIC_IP:8080
```

### screenshot of browser showing Jenkins unlock page at http://YOUR_PUBLIC_IP:8080
> ![alt text](<asset/Screenshot 2026-05-07 101726.png>)

### Step 10 — Unlock Jenkins

Retrieve the initial admin password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Copy the output and paste it into the **Unlock Jenkins** page in the browser.
## screenshot of terminal showing command to retrieve Jenkins initial admin password and the output
> ![alt text](<asset/Screenshot 2026-05-07 101826.png>)


### Step 11 — Install plugins

Select **Install suggested plugins** and wait for installation to complete.

### Step 12 — Create admin user

Fill in your details and click **Save and Continue**, or click **Skip and continue as admin** to use the default credentials.

---

## screenshot of Jenkins plugin installation progress and admin user creation page
> ![alt text](<asset/Screenshot 2026-05-07 101927.png>)
> ![alt text](<asset/Screenshot 2026-05-07 102320.png>)
> ![alt text](<asset/Screenshot 2026-05-07 102350.png>)

## Phase 4 — Install Apache Web Server

### Step 13 — Install Apache

```bash
sudo apt install apache2 -y
```
## screenshot of terminal showing Apache installation command and progress
> ![alt text](<asset/Screenshot 2026-05-07 102445.png>)

### Step 14 — Start Apache

```bash
sudo systemctl start apache2
sudo systemctl enable apache2
```
## screenshot of terminal showing Apache start and enable commands
> ![alt text](<asset/Screenshot 2026-05-07 102557.png>)

### Step 15 — Verify Apache is running

Open in your browser:
```
http://YOUR_PUBLIC_IP
```

You should see the **Apache2 Ubuntu Default Page**.
## screenshot of browser showing Apache2 Ubuntu Default Page at http://YOUR_PUBLIC_IP
> ![alt text](<asset/Screenshot 2026-05-07 102623.png>)

### Step 16 — Create the app directory

```bash
# Create deployment directory
sudo mkdir -p /var/www/html/myapp

# Set ownership
sudo chown -R www-data:www-data /var/www/html/myapp

# Set permissions
sudo chmod -R 755 /var/www/html/myapp

# Test with a placeholder page
echo "<h1>Apache is working 🚀</h1>" | sudo tee /var/www/html/myapp/index.html
```

Verify by visiting:
```
http://YOUR_PUBLIC_IP/myapp
```

## screenshot of terminal showing commands to create app directory and test page, and browser showing the test page at http://YOUR_PUBLIC_IP/myapp
> ![alt text](<asset/Screenshot 2026-05-07 102803.png>)
> ![alt text](<asset/Screenshot 2026-05-07 102839.png>)

### Step 17 — Set Jenkins permissions (CRITICAL)

Jenkins needs write access to the deployment directory:

```bash
# Add Jenkins to www-data group
sudo usermod -aG www-data jenkins

# Give Jenkins ownership of the deployment directory
sudo chown -R jenkins:www-data /var/www/html/myapp

# Set directory permissions
sudo chmod -R 775 /var/www/html/myapp
```

Allow Jenkins to run `sudo` commands without a password (required for the deploy script):

```bash
sudo EDITOR=vim visudo
```

Add this line at the **very bottom** of the file:

```
jenkins ALL=(ALL) NOPASSWD: /usr/bin/mkdir, /usr/bin/rm, /usr/bin/cp, /usr/bin/chown, /usr/bin/systemctl
```

Save with `:wq` and exit.

> **Why this is necessary:** The Jenkins deploy script runs `sudo cp`, `sudo chown`, and `sudo systemctl restart apache2`. Without this permission, the build will fail with a `permission denied` error.

## screenshot of terminal showing commands to set Jenkins permissions and the visudo file with the added line for Jenkins
> ![alt text](<asset/Screenshot 2026-05-07 103341.png>)

---

## Phase 5 — Configure Jenkins Job (Freestyle)

### Step 18 — Create a new Job

1. Go to **Jenkins Dashboard → New Item**
2. Enter name: `deploy-static-site`
3. Select **Freestyle project**
4. Click **OK**

### Step 19 — Configure Source Code Management

1. Under **Source Code Management**, select **Git**
2. Enter your forked repository URL:
```
https://github.com/YOUR-USERNAME/fruitables.git
```
3. Branch: `*/master` (or `*/main` depending on your repo)

### Step 20 — Configure Build Trigger

Under **Build Triggers**, enable:

☑ **GitHub hook trigger for GITScm polling**

This tells Jenkins to listen for webhook events from GitHub.

### Step 21 — Add Deployment Script

1. Scroll down to **Build Steps**
2. Click **Add build step → Execute shell**
3. Paste the following script:

```bash
echo "Starting deployment..."

# Create directory if not exists
sudo mkdir -p /var/www/html/myapp

# Clean old files
sudo rm -rf /var/www/html/myapp/*

# Copy new files
sudo cp -r * /var/www/html/myapp/

# Set correct permissions
sudo chown -R www-data:www-data /var/www/html/myapp

# Restart Apache
sudo systemctl restart apache2

echo "Deployment completed!"
```

### Step 22 — Save the job

Click **Save**.

## screenshot of Jenkins job configuration showing Git source, build trigger, and shell script
> ![alt text](<asset/Screenshot 2026-05-07 103520.png>)
> ![alt text](<asset/Screenshot 2026-05-07 104232.png>)
> ![alt text](<asset/Screenshot 2026-05-07 104316.png>)
> ![alt text](<asset/Screenshot 2026-05-07 104507.png>)
> ![alt text](<asset/Screenshot 2026-05-07 104546.png>)

---

## Phase 6 — Enable Auto Deployment via GitHub Webhook

### Step 23 — Add GitHub Webhook

1. Go to your **forked GitHub repository**
2. Click **Settings → Webhooks → Add webhook**
3. Fill in:

| Field | Value |
|---|---|
| Payload URL | `http://YOUR_PUBLIC_IP:8080/github-webhook/` |
| Content type | `application/json` |
| Events | Just the push event |

4. Click **Add webhook**

> **Note:** GitHub must be able to reach your EC2 instance on port 8080. Confirm port 8080 is open in your security group before adding the webhook.

## screenshot of GitHub webhook configuration showing payload URL, content type, and event selection
> ![alt text](<asset/Screenshot 2026-05-07 104802.png>)
> ![alt text](<asset/Screenshot 2026-05-07 104822.png>)

---

## Phase 7 — Test the CI/CD Pipeline

### Step 24 — Trigger a deployment

Make a change to your forked repository and push:

```bash
git clone https://github.com/musty2025x/fruitables.git
cd fruitables

# Make a visible change
echo "<!-- deployed via Jenkins -->" >> index.html

git add .
git commit -m "test Jenkins CI/CD deployment"
git push origin master
```

## screenshot of changes in index.html, git commit, and push commands
> ![alt text](<asset/Screenshot 2026-05-07 105111.png>)

**What happens automatically:**
1. GitHub receives the push and fires the webhook to Jenkins
2. Jenkins detects the trigger and starts the `deploy-static-site` job
3. Jenkins pulls the latest code from GitHub
4. The shell script copies files to `/var/www/html/myapp` and restarts Apache
5. The website is live with the new changes

## screenshot of Jenkins dashboard showing the triggered build and build history
> ![alt text](<asset/Screenshot 2026-05-07 111123.png>)

Verify by visiting:
```
http://YOUR_PUBLIC_IP/myapp
```

## screenshot of Jenkins build console showing the deployment process and browser showing the updated website at http://YOUR_PUBLIC_IP/myapp

> ![alt text](<asset/Screenshot 2026-05-07 111058.png>)
---

## Phase 8 — Configure DNS and HTTPS

### Step 25 — Configure DNS Records

At your domain registrar (Namecheap, GoDaddy, Cloudflare, Qservers), add:

| Type | Name | Value | Purpose |
|---|---|---|---|
| `A` | `@` | `YOUR_EC2_PUBLIC_IP` | Root domain |
| `A` | `www` | `YOUR_EC2_PUBLIC_IP` | www subdomain |

Wait **5–30 minutes** for DNS propagation, then verify:
```
http://yourdomain.com/myapp
```
## screenshot of DNS records configuration at a domain registrar showing A records for @ and www pointing to the EC2 public IP
> ![alt text](<asset/Screenshot 2026-05-07 111310.png>)
> ![alt text](<asset/Screenshot 2026-05-07 111344.png>)

### Step 26 — Enable HTTPS with Certbot

```bash
sudo apt install certbot python3-certbot-apache -y
sudo certbot --apache
```

Follow the prompts — Certbot will:
- Auto-detect your domain from Apache config
- Request a free Let's Encrypt certificate
- Configure Apache to redirect HTTP → HTTPS automatically
- Schedule auto-renewal

## screenshot of terminal showing Certbot installation and configuration process
> ![alt text](<asset/Screenshot 2026-05-07 111454.png>)
> ![alt text](<asset/Screenshot 2026-05-07 111713.png>)
> ![alt text](<asset/Screenshot 2026-05-07 111724.png>)

### Step 27 — Verify HTTPS

```
https://yourdomain.com/myapp
```

The padlock confirms SSL is active and the site is secured.

## screenshot of browser showing the website with HTTPS and padlock at https://yourdomain.com/myapp
> ![alt text](<asset/Screenshot 2026-05-07 111846.png>)

---

## Output

### Jenkins Build Console Log

```
Started by GitHub push by Musty2025x
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/deploy-static-site
The recommended git tool is: NONE
Cloning the remote Git repository
Cloning repository https://github.com/Musty2025x/fruitables.git
 > git init /var/lib/jenkins/workspace/deploy-static-site
Fetching upstream changes from https://github.com/Musty2025x/fruitables.git
Checking out Revision abc1234 (refs/remotes/origin/master)

[deploy-static-site] $ /bin/sh -xe /tmp/jenkins-script.sh

+ echo 'Starting deployment...'
Starting deployment...

+ sudo mkdir -p /var/www/html/myapp
+ sudo rm -rf /var/www/html/myapp/*
+ sudo cp -r * /var/www/html/myapp/
+ sudo chown -R www-data:www-data /var/www/html/myapp
+ sudo systemctl restart apache2

+ echo 'Deployment completed!'
Deployment completed!

Finished: SUCCESS ✅
```

### Security Group — Inbound Rules

| Port | Protocol | Source | Purpose |
|---|---|---|---|
| 22 | TCP | 0.0.0.0/0 | SSH access |
| 80 | TCP | 0.0.0.0/0 | HTTP web traffic |
| 443 | TCP | 0.0.0.0/0 | HTTPS web traffic |
| 8080 | TCP | 0.0.0.0/0 | Jenkins UI + webhook |

### Jenkins Job Configuration Summary

| Setting | Value |
|---|---|
| Job type | Freestyle Project |
| Job name | `deploy-static-site` |
| Source | Git — forked `fruitables` repo |
| Build trigger | GitHub hook trigger for GITScm polling |
| Build step | Execute shell — deploy script |
| Deploy path | `/var/www/html/myapp` |

---

## Errors & Fixes

### Jenkins stuck `pending# waiting for next available executor`

**Cause:** Jenkins has no available executors to run the job.
**Fix:**
- Go to **Jenkins → Manage Jenkins → Manage Nodes and Clouds**  
- Click on the node (e.g., `master`) and check the number of executors. It should be at least 1. If it's 0, change it to 1 or more and save.
- Go to **Manage Jenkins → Manage Nodes and Clouds → Built-in Node → Configure → Set Free Space Threshold to a lower value (e.g. 100MB) and save.**  
- If the node is marked as offline, click on it and select **Mark this node online**.


### ❌ 1 — Jenkins build fails with `Permission denied`

**Cause:** Jenkins does not have write access to `/var/www/html/myapp`.

**Fix:**
```bash
sudo usermod -aG www-data jenkins
sudo chown -R jenkins:www-data /var/www/html/myapp
sudo chmod -R 775 /var/www/html/myapp

# Also ensure sudo access without password
sudo visudo
# Add at the bottom:
jenkins ALL=(ALL) NOPASSWD: ALL
```

---

### ❌ 2 — GitHub Webhook shows `Failed to connect` or times out

**Cause:** Port 8080 is not open in the EC2 Security Group.

**Fix:**
- Go to **AWS → EC2 → Security Groups → Inbound rules**
- Add a rule: Custom TCP, port 8080, source `0.0.0.0/0`
- Re-deliver the webhook from **GitHub → Settings → Webhooks → Recent Deliveries**

---

### ❌ 3 — Jenkins shows `Branch not found`

**Cause:** The branch specifier in the job is set to `*/master` but the repo uses `main`.

**Fix:** In the Jenkins job → Source Code Management → Branches to build, change to:
```
*/main
```

---

### ❌ 4 — `http://YOUR_IP/myapp` returns 403 Forbidden

**Cause:** Apache cannot read the files in `/var/www/html/myapp` due to incorrect permissions.

**Fix:**
```bash
sudo chmod -R 755 /var/www/html/myapp
sudo chown -R www-data:www-data /var/www/html/myapp
sudo systemctl restart apache2
```

---

### ❌ 5 — Certbot SSL Error (NXDOMAIN)

**Cause:** DNS records have not propagated yet, or an incorrect domain was entered.

**Fix:**
- Wait 10–30 minutes after adding DNS records
- Verify DNS is resolving correctly:
```bash
nslookup yourdomain.com
```
- Re-run `sudo certbot --apache` once DNS resolves correctly

---

### ❌ 6 — Apache default page shows instead of `/myapp`

**Cause:** Visiting `http://YOUR_IP` (root) instead of `http://YOUR_IP/myapp`.

**Fix:** Append `/myapp` to the URL, or configure Apache to redirect the root to `/myapp`:

```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```

Add inside the `<VirtualHost>` block:
```apache
RedirectMatch ^/$ /myapp/
```

```bash
sudo systemctl restart apache2
```

---

## Cleanup

To avoid ongoing AWS charges after the lab:

1. Go to **AWS Console → EC2 → Instances**
2. Select `your-instance`
3. Click **Instance State → Terminate instance**
4. Confirm termination

## screenshot of AWS EC2 instance termination process
> ![alt text](<asset/Screenshot 2026-05-07 112233.png>)

---

## Project Summary

| Component | Detail |
|---|---|
| Application | Static website (Fruitables — HTML/CSS/JS) |
| Repository | Forked from `github.com/musty2025x/fruitables` |
| CI/CD | Jenkins Freestyle Job — triggered by GitHub webhook |
| Server | AWS EC2 — Ubuntu 22.04 — t3.small |
| Web Server | Apache2 — serving from `/var/www/html/myapp` |
| Jenkins Port | 8080 |
| Deploy Path | `/var/www/html/myapp` |
| SSL | Let's Encrypt — Certbot Apache plugin |
| Custom Domain | `https://yourdomain.com/myapp` |

---

## References

- [Jenkins Installation on Ubuntu](https://www.jenkins.io/doc/book/installing/linux/)
- [Apache2 Documentation](https://httpd.apache.org/docs/)
- [GitHub Webhooks Guide](https://docs.github.com/en/developers/webhooks-and-events/webhooks)
- [Certbot — Apache](https://certbot.eff.org/instructions?ws=apache&os=ubuntufocal)
- [AWS EC2 Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html)
- [Fruitables Template — Source Repo](https://github.com/musty2025x/fruitables)