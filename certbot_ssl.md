# 🔒 SSL Setup on Nginx using Certbot

This guide provides **step-by-step instructions** to secure your Nginx server using **SSL** from Let's Encrypt via Certbot. Follow these instructions carefully to ensure a smooth and secure setup.

---

## 🚀 Prerequisites

Before proceeding, ensure that:

- ✅ **Nginx** is installed and running on your server.
- ✅ You have a **registered domain name**.
- ✅ The domain is correctly **pointed to your server's public IP** (check DNS settings).

---

## 🛠 Steps

### 1. 📥 Install Certbot

Certbot is used to automate the process of obtaining SSL certificates from Let's Encrypt.

1. Update your system’s package list:

    ```bash
    sudo apt update
    ```

2. Install `snapd` if it’s not already installed:

    ```bash
    sudo apt install snapd
    ```

3. Ensure `snapd` is up-to-date:

    ```bash
    sudo snap install core
    sudo snap refresh core
    ```

4. Install Certbot using Snap:

    ```bash
    sudo snap install --classic certbot
    ```

5. Create a symbolic link for Certbot:

    ```bash
    sudo ln -s /snap/bin/certbot /usr/bin/certbot
    ```

6. Verify Certbot installation:

    ```bash
    certbot --version
    ```

---

### 2. 🔑 Obtain SSL Certificate

To get the SSL certificate for your domain, run the following command:

```bash
sudo certbot --nginx -d your-domain.com
