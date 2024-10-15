Here's the Nginx configuration formatted as a shell script with proper syntax highlighting for better readability:

```shell
# =========================================
# Nginx Configuration for HTTP to HTTPS Redirection
# =========================================

# -----------------------------------------
# HTTP Block
# -----------------------------------------
server {
    listen 80;  # Listen on port 80 for HTTP
    listen [::]:80;  # Listen on port 80 for IPv6

    server_name your-domain.com;  # Replace with your actual domain

    # Redirect all HTTP requests to HTTPS
    return 301 https://$host$request_uri;
}

# -----------------------------------------
# HTTPS Block
# -----------------------------------------
server {
    listen 443 ssl;  # Listen on port 443 for HTTPS
    listen [::]:443 ssl;  # Listen on port 443 for IPv6

    server_name your-domain.com;  # Replace with your actual domain

    # SSL Certificates
    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;  # Update with your SSL path
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;  # Update with your SSL path

    # -----------------------------------------
    # SSL Protocols and Ciphers
    # -----------------------------------------
    ssl_protocols TLSv1.2 TLSv1.3;  # Supported protocols
    ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256';  # Supported ciphers
    ssl_prefer_server_ciphers on;  # Prefer server ciphers over client ciphers

    # -----------------------------------------
    # Security Headers
    # -----------------------------------------
    add_header X-Content-Type-Options nosniff;  # Prevent MIME type sniffing
    add_header X-Frame-Options DENY;  # Prevent clickjacking
    add_header X-XSS-Protection "1; mode=block";  # Enable XSS protection

    # -----------------------------------------
    # Proxy Settings for the Node.js Backend
    # -----------------------------------------
    location / {
        proxy_pass http://localhost:8080;  # Update with your backend URL
        proxy_http_version 1.1;  # Use HTTP/1.1 for the backend
        proxy_set_header Upgrade $http_upgrade;  # Support for WebSocket upgrades
        proxy_set_header Connection 'upgrade';  # Support for WebSocket connections
        proxy_set_header Host $host;  # Pass the original host header
        proxy_cache_bypass $http_upgrade;  # Bypass cache for WebSocket connections
    }
}
```

### Instructions

1. **Copy the Code:** Copy the entire code block above.
2. **Create Nginx Configuration File:** Open your terminal and create or edit your Nginx configuration file, typically found in `/etc/nginx/sites-available/default` or `/etc/nginx/conf.d/`.
   
   ```bash
   sudo nano /etc/nginx/sites-available/default
   ```

3. **Paste the Code:** Paste the copied configuration into the file.
4. **Save and Exit:** Save the changes and exit the text editor (in nano, you can do this by pressing `CTRL + X`, then `Y`, and finally `Enter`).
5. **Test the Configuration:** Run the following command to test the Nginx configuration for syntax errors:

   ```bash
   sudo nginx -t
   ```

6. **Reload Nginx:** If the configuration test is successful, reload Nginx to apply the changes:

   ```bash
   sudo systemctl reload nginx
   ```

Now your Nginx server should be configured correctly for HTTP to HTTPS redirection!
