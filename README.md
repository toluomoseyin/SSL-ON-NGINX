# README: Setting Up SSL for Nginx on Ubuntu

This guide will walk you through the process of obtaining an SSL certificate from ZeroSSL, installing it on your Ubuntu server, configuring Nginx to use the certificate for your domain, and setting up a basic HTML page.

## Prerequisites

- An Ubuntu server (20.04 or later)
- Domain name (e.g., `newtestapp.name.ng`)
- Root or sudo access to the server
- Nginx installed and running
- Basic knowledge of using the command line

## Step 1: Install Nginx (if not already installed)

1. **Update the Package Index**:
   Open your terminal and run the following command to update the package index:

   ```bash
   sudo apt update
   ```

2. **Install Nginx**:
   Install Nginx using the following command:

   ```bash
   sudo apt install nginx
   ```

3. **Start and Enable Nginx**:
   Start the Nginx service and enable it to run at startup:

   ```bash
   sudo systemctl start nginx
   sudo systemctl enable nginx
   ```

4. **Check Nginx Status**:
   Verify that Nginx is running:

   ```bash
   sudo systemctl status nginx
   ```

## Step 2: Create a Simple HTML Page

1. **Create Document Root**:
   Create a directory for your website:

   ```bash
   sudo mkdir -p /var/www/newtestapp.name.ng
   ```

2. **Set Permissions**:
   Set the correct permissions for the directory:

   ```bash
   sudo chown -R www-data:www-data /var/www/newtestapp.name.ng
   sudo chmod -R 755 /var/www/newtestapp.name.ng
   ```

3. **Create an HTML File**:
   Use a text editor like nano to create an index.html file:

   ```bash
   sudo nano /var/www/newtestapp.name.ng/index.html
   ```

   Copy and paste the following HTML code:

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>Welcome to My Website</title>
       <style>
           body {
               font-family: Arial, sans-serif;
               margin: 0;
               padding: 0;
               background-color: #f4f4f4;
               color: #333;
           }
           header {
               background: #35424a;
               color: #ffffff;
               padding: 20px 0;
               text-align: center;
           }
           .container {
               width: 80%;
               margin: auto;
               overflow: hidden;
               padding: 20px;
           }
           footer {
               text-align: center;
               padding: 10px 0;
               background: #35424a;
               color: #ffffff;
               position: relative;
               bottom: 0;
               width: 100%;
           }
       </style>
   </head>
   <body>
       <header>
           <h1>Welcome to My Website</h1>
       </header>
       <div class="container">
           <h2>Hello, World!</h2>
           <p>This is a simple HTML page to display on your server.</p>
           <p>Feel free to modify this content as you like!</p>
       </div>
       <footer>
           <p>Copyright © 2024 Your Name</p>
       </footer>
   </body>
   </html>
   ```

   Save the file and exit the editor (in nano, press `CTRL + X`, then `Y`, and `Enter`).

## Step 3: Obtain SSL Certificates from ZeroSSL

1. **Create an Account on ZeroSSL**:
   - Go to [ZeroSSL](https://zerossl.com) and create an account if you don’t already have one.

2. **Create a New SSL Certificate**:
   - Once logged in, navigate to **"SSL Certificates"**.
   - Click on **"New SSL Certificate"**.
   - Enter your domain name (e.g., `newtestapp.name.ng`) and click **"Next"**.

3. **Choose the Validation Method**:
   - You can choose **Email**, **HTTP**, or **DNS** validation. Follow the instructions based on the method you choose.

4. **Download Your Certificates**:
   - After validation is complete, download the following files:
     - `certificate.crt`
     - `ca_bundle.crt`
     - `private.key` (if applicable)

## Step 4: Upload SSL Certificates to Your Server

1. **Connect to Your Server Using SFTP**:
   - Use an SFTP client (like FileZilla) to connect to your server using your SSH key.
   - Upload `certificate.crt`, `ca_bundle.crt`, and `private.key` to a directory on your server (e.g., `/etc/ssl/private/` and `/etc/ssl/certs/`).

2. **Set Correct Permissions**:
   - Connect to your server via SSH and set the correct permissions for the files:

   ```bash
   sudo chmod 600 /etc/ssl/private/private.key
   sudo chmod 644 /etc/ssl/certs/certificate.crt
   sudo chmod 644 /etc/ssl/certs/ca_bundle.crt
   ```

## Step 5: Combine SSL Certificates (if needed)

If your SSL provider requires a combined certificate file, you can create it by concatenating the files:

1. **Combine the Certificates**:
   If you encounter permission issues when running the following command, ensure you have the necessary permissions to write to the target directory or use the `sudo tee` command instead:

   ```bash
   sudo cat /etc/ssl/certs/certificate.crt /etc/ssl/certs/ca_bundle.crt | sudo tee /etc/ssl/certs/newtestapp.name.ng_combined.crt
   ```

2. **Set Permissions for the Combined Certificate**:

   ```bash
   sudo chmod 644 /etc/ssl/certs/newtestapp.name.ng_combined.crt
   ```

## Step 6: Configure Nginx to Use SSL

1. **Open Nginx Configuration**:
   - Edit your Nginx configuration file (create one if it doesn't exist):

   ```bash
   sudo nano /etc/nginx/sites-available/newtestapp.name.ng.conf
   ```

2. **Add or Modify Server Blocks**:
   - Add the following configuration to handle HTTPS requests:

   ```nginx
   server {
       listen 80;
       server_name newtestapp.name.ng www.newtestapp.name.ng;

       # Redirect HTTP to HTTPS
       return 301 https://$host$request_uri;
   }

   server {
       listen 443 ssl;
       server_name newtestapp.name.ng www.newtestapp.name.ng;

       ssl_certificate /etc/ssl/certs/newtestapp.name.ng_combined.crt;  # Use combined certificate
       ssl_certificate_key /etc/ssl/private/private.key;

       location / {
           root /var/www/newtestapp.name.ng;  # Adjust to your document root
           index index.html index.htm;
       }
   }
   ```

3. **Create a Symlink to Enable the Site**:
   - If your configuration is in `sites-available`, create a symlink to `sites-enabled`:

   ```bash
   sudo ln -s /etc/nginx/sites-available/newtestapp.name.ng.conf /etc/nginx/sites-enabled/
   ```

## Step 7: Test and Restart Nginx

1. **Test Nginx Configuration**:
   - Run the following command to test the Nginx configuration for any errors:

   ```bash
   sudo nginx -t
   ```

2. **Restart Nginx**:
   - If the test is successful, restart Nginx to apply the changes:

   ```bash
   sudo systemctl restart nginx
   ```

## Step 8: Verify SSL Installation

- Open a web browser and navigate to `https://newtestapp.name.ng`.
- You should see your website with a secure connection indicated by a padlock icon in the address bar.

## Troubleshooting

- If you encounter a **404 Not Found** error, check that your document root is set correctly and contains an `index.html` file.
- For any SSL-related errors, check the Nginx error logs for detailed messages:

```bash
sudo tail -f /var/log/nginx/error.log
```

## Additional Notes

- Make sure to keep your SSL certificates updated to avoid any downtime.
- Regularly check your Nginx configuration and logs for any issues.
