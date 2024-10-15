
```markdown
# Step 1: Set Up a Firewall Using UFW

This guide provides step-by-step instructions to secure your EC2 instance by configuring a firewall using **UFW (Uncomplicated Firewall)**. UFW helps to manage firewall rules easily and restrict access to your server, allowing only necessary traffic.

## Prerequisites

- You have **Nginx** installed and running.
- Your EC2 instance is accessible via **SSH**.

## Steps

### 1. Install UFW

If UFW is not installed on your system, you can install it using the following command:

```bash
sudo apt install ufw
```

### 2. Allow Nginx Traffic

You need to allow HTTP and HTTPS traffic for your web server (Nginx). Run the following command:

```bash
sudo ufw allow 'Nginx Full'
```

This command allows both HTTP (port 80) and HTTPS (port 443) traffic.

### 3. Allow SSH Traffic

To ensure that you don't lose access to your server, allow **OpenSSH** (port 22) traffic:

```bash
sudo ufw allow OpenSSH
```

### 4. Enable the Firewall

Now, you can enable UFW to activate the firewall rules:

```bash
sudo ufw enable
```

### 5. Verify UFW Status

To confirm that the firewall is active and the rules are applied, check the status:

```bash
sudo ufw status
```

You should see an output like this:

```bash
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
Nginx Full                 ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
Nginx Full (v6)            ALLOW       Anywhere (v6)
```

## What’s Next?

Once your firewall is set up and confirmed to be working correctly, you can proceed to the next steps to secure your server even more by enabling rate limiting, monitoring logs, and applying more security measures.

---

```bash
# Summary of Shell Commands

sudo apt install ufw
sudo ufw allow 'Nginx Full'
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status
```

---

Congratulations, your server's firewall is now configured and running, allowing only necessary traffic to your Node.js backend!
```

