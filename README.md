# Hosting a Flask Web Application on OCI with the following OCI services:
- DNS Zone
- Layer 7 Load Balancer
- Custon Images
- Instance Configuration and Instance Pools
- Metric Based Autoscaling
- Instances and VCN
- Bastion

# Prerequisites:

- Import SSL Certificate in Oracle Cloud Infrastructure (OCI).
- Familiarity with creating and using Bastion Service.
- Basic understanding of OCI fundamentals.

# Step 1: Set Up Web Server
- Create a new Virtual Cloud Network with the Wizard to abstract away some networking.
- Set up a new OCI instance with the specified configuration (A1.Flex, 1 OCPU, 6GB Memory, Ubuntu) in the private subnet. Include Bastion Agent and choose SSH keys.
- Create a Bastion session and connect to the instance.


# Allow incoming HTTP and HTTPS traffic
```bash
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 5000 -j ACCEPT
```
```bash
sudo iptables -I INPUT 7 -m state --state NEW -p tcp --dport 80 -j ACCEPT
```
```bash
sudo netfilter-persistent save
```

# Install required packages
```bash
sudo apt update
```
```bash
sudo apt install -y python3-pip nginx
```

# Install virtualenv and virtualenvwrapper
```bash
pip3 install virtualenv virtualenvwrapper
```
```bash
echo 'export WORKON_HOME=~/envs' >> ~/.bashrc
```
```bash
echo 'export PATH=$PATH:/home/ubuntu/.local/bin' >> ~/.bashrc
```
```bash
echo 'source /home/ubuntu/.local/bin/virtualenvwrapper.sh' >> ~/.bashrc
```
```bash
source ~/.bashrc
```
# Create and activate a virtual environment

```bash
mkvirtualenv flask01
```
```bash
workon flask01
```
# Install Flask and Gunicorn
```bash
pip3 install Flask gunicorn
```
# Clone and setup your Flask project
```bash
git clone <your_project_repository>
```
```bash
cd flask-project
```
```bash
pip install -r requirements.txt
```
```bash
export FLASK_APP=app.py
```
# Run Flask app with Gunicorn
```bash
gunicorn -w 4 -b 0.0.0.0:5000 app:app
```
# Install and configure Nginx
```bash
sudo nano /etc/nginx/sites-available/demo_app.conf
```
- In demo_app.conf:
```bash
server {
   listen 80;
   server_name YOUR_DOMAIN_NAME;

   location / {
       proxy_pass http://127.0.0.1:5000;
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   }
}
```

- Activate and Check
```bash
sudo ln -s /etc/nginx/sites-available/demo_app.conf /etc/nginx/sites-enabled
sudo nginx -t
sudo service nginx restart
```
# Configure Gunicorn to start on boot
```bash
sudo nano /etc/systemd/system/flask-app.service
```
```bash
[Unit]
Description=Gunicorn instance to serve Flask application
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/flask-project
Environment="PATH=/home/ubuntu/envs/flask01/bin"
ExecStart=/home/ubuntu/envs/flask01/bin/gunicorn --workers 4 --bind 0.0.0.0:5000 app:app

[Install]
WantedBy=multi-user.target
```
- Check health and activate
```bash
sudo systemctl daemon-reload
sudo systemctl enable flask-app
sudo systemctl start flask-app
sudo systemctl status flask-app
```


# 2. Create Custom Image, Instance Configuration, Instance Pool and Autoscaling.

# Create Custom Image from the Server and launch new Instance with it
- Create a custom Image from the Webserver and use it to launch a new instance.
You could start a little Clean-Up and delete the first instance.
- Wait until the new Webserver is running and check if you can see the webpage: <publicip:5000>

# From the Current Instance Menu select Create Instance Configuration
- Set up the Instance Configuration

# Create Instance Pool with the freshly created Instance Config
- Choose 2 Instances
- Select Attach Load Balancer -> Select the Load Balancer, Backend Set and Port 80 -> Next + Create

# Configure Autoscaling Configurations
- Navigate to Autoscaling Configurations in Compute and start the process
- Choose Instance Pool
- Metric Based Autoscaling
- CPU Utilization -> Scale Out Rule > 70 -> Scale Down Rule < 20 -> Min. 1 | Max. 2 | Start 2

# Clean Up instances that are not needed
- Terminate the 2 instances we first created (that are not part of the instance pool) if not already done:
- The first Instance that was used to create custom image and the one we created with the custom image.
