# Setting up a FastAPI Application with UVICORN and Nginx Reverse Proxy

This guide will walk you through the process of setting up a FastAPI application and configuring Nginx as a reverse proxy.

## 1. Create a System User

Create a dedicated system user for the API:

```bash
sudo adduser --system --group --home /var/www/potato_api potato_api
```

This command creates a system user named `potato_api` with a home directory at `/var/www/potato_api`.

## 2. Clone the Repository
Navigate to the project directory and set up a virtual environment:
```bash
cd /var/www/potato_api
```

Clone your project repository into the current directory:

```bash
git clone your_repository_url .
```

Replace `your_repository_url` with the actual URL of your Git repository.

## 3. Set Up the Project Environment

```bash
cd /var/www/potato_api
python -m venv venv
```

## 4. Install Dependencies

Activate the Project Enviroment and Install the required Python packages:

```bash
source venv/bin/activate
pip install -r requirements.txt
```

## 5. Set Correct Ownership

Ensure the `potato_api` user owns all files in the project directory:

```bash
sudo chown -R potato_api:potato_api /var/www/potato_api
```

## 6. Test the Application

You can test the application by running:

```bash
uvicorn main:app --reload
```
Replace `main` with the starting point of your api. If App works without any issues, move to Step 7.
## 7. Create a Systemd Service

Create a systemd service file to manage the API:

```bash
sudo nano /etc/systemd/system/potato_api.service
```

Add the following content to the file:

```ini
[Unit]
Description=Domain Info API Powered by FastAPI
After=network.target

[Service]
User=potato_api
Group=potato_api
WorkingDirectory=/var/www/potato_api
Environment="PATH=/var/www/potato_api/venv/bin:$PATH"
ExecStart=/var/www/potato_api/venv/bin/uvicorn --host 0.0.0.0 --port 5001 app:app
Restart=always

[Install]
WantedBy=multi-user.target
```

## 8. Enable and Start the Service

Reload systemd, enable the service to start on boot, and start it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable potato_api.service
sudo systemctl start potato_api.service
```

## 9. Check Service Status

Verify that the service is running:

```bash
sudo systemctl status potato_api
```

## 10. Configure Nginx Reverse Proxy

Install Nginx if not already installed:

```bash
sudo apt update
sudo apt install nginx
```

Create a new Nginx configuration file:

```bash
sudo nano /etc/nginx/sites-available/potato_api
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name your_domain.com;

    location / {
        proxy_pass http://localhost:5001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Replace `your_domain.com` with your actual domain name.

Create a symbolic link to enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/potato_api /etc/nginx/sites-enabled/
```

Test the Nginx configuration:

```bash
sudo nginx -t
```

If the test is successful, restart Nginx:

```bash
sudo systemctl restart nginx
```

Point `your_domain.com` to your server's ip address by creating "A" record and server's IP ADDRESS as value. Depending on your server provider, you may also need to unblock port 80 for HTTP Requests and port 443 for HTTPS requests.

Your FastAPI application should now be accessible through your domain, with Nginx acting as a reverse proxy.

## What's Next?

Optimise Service File & your NGNIX Configuration.
Add a SSL Certificate
