
# Troubleshooting Missing Application Data in AMI Instance

## 1. Overview

When creating an Amazon Machine Image (AMI) from an EC2 instance, it's possible to encounter issues where the application data is not found. This document summarizes the troubleshooting steps taken to resolve this issue.

## 2. Connecting to the AMI Instance

To start troubleshooting, connect to your AMI instance using SSH:
```bash
ssh -i "your-key.pem" ubuntu@your-instance-ip
```

## 3. Checking for Application Data

After connecting, the first step is to locate your application data. Use the following command to search for your application files:
```bash
find / -name "*login_app*"
```

### Example Output
```
/home/ubuntu/login_app
/etc/nginx/sites-available/login_app
/etc/nginx/sites-enabled/login_app
```

## 4. Navigating to the Application Directory

Change to the directory where your application is expected to be:
```bash
cd /home/ubuntu/login_app
```

### Listing the Contents
Check the contents of the directory:
```bash
ls
```
### Example Output
```
app.py  myappenv  templates
```

## 5. Setting Up a Python Virtual Environment

Since you might not have the application environment set up, create a Python virtual environment:
```bash
python3 -m venv myappenv
```

Activate the virtual environment:
```bash
source myappenv/bin/activate
```

## 6. Installing Required Packages

Attempt to install required Python packages. If a `requirements.txt` file is available:
```bash
pip install -r requirements.txt
```

### Handling Missing Requirements File
If the `requirements.txt` file is not found:
```
ERROR: Could not open requirements file: [Errno 2] No such file or directory: 'requirements.txt'
```

You can install necessary packages manually:
```bash
pip install flask pymysql
```

## 7. Configuring Nginx

To ensure that your Flask application can be accessed via the web, configure Nginx.

### Navigating to Nginx Configuration
```bash
cd /etc/nginx/sites-available/
```

### Editing the Configuration
Open your Nginx configuration file:
```bash
sudo nano login_app
```

Add the following server block:
```nginx
server {
    listen 80;
    server_name 3.110.168.121;  # Use your public IP or domain

    location / {
        proxy_pass http://127.0.0.1:5000;  # Adjust if your app runs on a different port
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Creating a Symbolic Link
Link the configuration to the `sites-enabled` directory:
```bash
sudo ln -s /etc/nginx/sites-available/login_app /etc/nginx/sites-enabled/
```

## 8. Testing and Reloading Nginx

Check the Nginx configuration for syntax errors:
```bash
sudo nginx -t
```

If the configuration is valid, reload Nginx:
```bash
sudo systemctl reload nginx
```

## 9. Running the Flask Application

Navigate back to your application directory:
```bash
cd /home/ubuntu/login_app
```

Run your Flask application:
```bash
python app.py
```
Or, if you're using Gunicorn:
```bash
gunicorn -w 4 -b 127.0.0.1:5000 app:app
```

## 10. Accessing the Application

Try accessing your application using the public IP address:
```
http://3.110.168.121/
```

### If Nginx Shows Welcome Page Instead
If the Nginx welcome page appears, troubleshoot the following:

### 11. Troubleshooting Access Issues

- **Ensure Flask is Running**: Make sure your Flask app is running and listening on port `5000`.

- **Check Firewall Settings**:
```bash
sudo ufw status
```
If necessary, allow port `5000`:
```bash
sudo ufw allow 5000
```

- **Examine Nginx Logs**: Check the Nginx error log for any issues:
```bash
sudo tail -f /var/log/nginx/error.log
```

- **Test Direct Access to Flask App**:
Try accessing the Flask app directly:
```
http://3.110.168.121:5000/
```

## Final Note

If the login page is still not accessible after these checks, provide any errors from the Nginx logs or your Flask application's output for further troubleshooting.
```

This comprehensive guide captures the entire troubleshooting process and can be used for learning or presentation purposes. Let me know if you need any further adjustments!
