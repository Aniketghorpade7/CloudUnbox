# Cloud Workshop: Hosting Your First Website on AWS EC2

This guide will walk you through launching a virtual server in the cloud (AWS EC2) and turning it into a live web server accessible from anywhere in the world.

## Prerequisites
* An active AWS Account.
* A downloaded SSH Key Pair file (e.g., `cloudUnbox.pem`).
* A basic terminal (Command Prompt, PowerShell, or Linux/macOS Terminal).

---

## Step 1: Launching the EC2 Instance

1. Log into the AWS Management Console and search for **EC2**.
2. Click the orange **Launch instance** button.

3. **Name and tags:** Give your server a name (e.g., `CloudWorkshop-WebServer`).
4. **Application and OS Images (AMI):** Select **Amazon Linux** (Amazon Linux 2023 AMI). This is optimized for AWS.
5. **Instance type:** Select **t2.micro** (This is eligible for the AWS Free Tier).
6. **Key pair (login):** Select your existing key pair (e.g., `cloudUnbox.pem`) from the dropdown menu.
7. **Network settings:** This is the most critical step for a web server! Check the following boxes:
   * **Allow SSH traffic from:** Anywhere (Port 22 - so you can log in).
   * **Allow HTTP traffic from the internet:** (Port 80 - so the world can see your website).

8. Click the orange **Launch instance** button on the right panel.

---

## Step 2: Preparing Your SSH Key

Before you can connect, you must secure your private key file. SSH will reject the connection if the key is readable by other users on your computer.

1. Open your terminal.
2. Navigate to the folder where your `cloudUnbox.pem` file is saved.
3. Run the following command to restrict permissions:
   ```bash
    chmod 400 cloudUnbox.pem
    ```

---

## Step 3: Connecting to Your Server

Go back to the AWS EC2 Console and click on your running instance.


1. Find your **Public IPv4 address** in the instance details (e.g., `54.82.46.85`).
2. In your terminal, run the SSH command to connect as the default `ec2-user`:

```bash
ssh -i "cloudUnbox.pem" ec2-user@<YOUR_PUBLIC_IP>
```

---

## Step 4: Installing the Web Server

Now that you are inside your cloud server, you need to install the software that will serve your website to the internet (Apache).

Update the system packages:
```bash
sudo dnf update -y
```
Install the Apache web server:

```bash
sudo dnf install -y httpd
```
Start the web server and enable it to run on boot:

```Bash
sudo systemctl start httpd
sudo systemctl enable httpd
```
Step 5: Deploying Your Website
The default folder for web files is /var/www/html. First, we need to take ownership of this folder so we can add our own files.

Grant ownership to the ec2-user:

```Bash
sudo chown -R ec2-user:ec2-user /var/www/html
```
Create your custom homepage:
Run this command to quickly generate an index.html file with a welcome message:

```Bash
echo "<h1>Welcome to the Cloud, First Years!</h1><p>This website is running live on an AWS EC2 instance.</p>" > /var/www/html/index.html
```
(Alternatively, you can use scp/github to upload an existing HTML/CSS website folder).

---

## Optional: Uploading an Existing Website (Using SCP)

If you have a complete website built on your computer, you can securely transfer the files directly to your EC2 server using the `scp` command. 

**Important:** Do NOT run these commands inside your EC2 SSH session! Open a **new terminal tab** on your local computer, and make sure you are in the same folder as your `.pem` key and your website files.



### To upload a single file:
If you just want to upload a single file (like an `index.html`):

```bash
scp -i "cloudUnbox.pem" index.html ec2-user@<YOUR_PUBLIC_IP>:/var/www/html/
```
To upload an entire website folder:
If you have a folder containing your HTML, CSS, and image files, add the -r (recursive) flag to upload everything inside the folder at once.

(Assume your website files are inside a folder named my-website):

```Bash
scp -i "cloudUnbox.pem" -r my-website/* ec2-user@<YOUR_PUBLIC_IP>:/var/www/html/
```
How it works:

scp: The command to securely copy files.

-i "cloudUnbox.pem": Uses your private key to authenticate.

-r: Tells the command to copy all folders and files recursively.

my-website/*: The files on your local computer you want to send.

ec2-user@<YOUR_PUBLIC_IP>:/var/www/html/: The exact destination path on the cloud server.


# Cloudflare Tunnel (Temporary HTTPS) Setup Guide

This guide shows how to expose an HTTP website running on an EC2 instance as a temporary HTTPS URL using Cloudflare Tunnel.

This is useful when mobile browsers block camera access on HTTP.

Powered by Cloudflare.

---

## Prerequisites

- EC2 instance (Amazon Linux or similar)
- Website running locally on port 80

Verify first:

```bash
curl http://localhost
```

You must see your HTML output.

 ### Step 1 — Download cloudflared

SSH into your EC2 instance and run:
```bash
cd ~
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 -o cloudflared
```
### Step 2 — Make it executable
```bash
chmod +x cloudflared
```
### Step 3 — Move binary into PATH
```bash
sudo mv cloudflared /usr/local/bin/
```
### Step 4 — Verify installation
```bash
cloudflared --version
```
You should see a version number.
If not, stop here and fix before continuing.

### Step 5 — Start temporary HTTPS tunnel (no account required)
```bash
cloudflared tunnel --url http://localhost:80
```
After a few seconds you’ll get output similar to:
```
https://random-words.trycloudflare.com
```
### Step 6 — Open URL on mobile
```
Copy the https://*.trycloudflare.com link and open it on your phone.
```
Your camera button should now work because the site is served over HTTPS.