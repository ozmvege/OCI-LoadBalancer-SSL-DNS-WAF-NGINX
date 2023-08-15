# OCI-LoadBalancer-SSL-DNS-WAF-NGINX
Project with OCI

Project Name: Host Flask Webapp on with SSL and dns

# Prerequisets:
Import Certificate in OCI.

Create Loadbalancer:
Layer 7 -> Choose a name -> Select Public -> Get an IP -> Select VCN -> Select WRR -> rest default and next -> Select HTTPS and select your Cert (if you dont have it set up go to cert overview and set it up)

Create DNS Zone:
In DNS create a public DNS Zone with your domain.

After that go to your registrant and create at least 2 NS Records with the Nameserver for your (sub)domain.
Create A record that points to Load balancer IP

# 1. Set up Webserver
1.1 Create Instance and Load Balancer.
Create a new Oracle Cloud Infrastructure (OCI) instance with the specifications you mentioned (A1.Flex, 1 OCPU, 6GB Memory, Oracle Linux 8.0).
Set up a new Virtual Cloud Network (VCN).
Add rule in Security List to allow ingress Rule for port 443,80,5000
Generate SSH Keys


1.2 Setup Webserver

# Allow incoming HTTP and HTTPS traffic
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 5000 -j ACCEPT
sudo iptables -I INPUT 7 -m state --state NEW -p tcp --dport 80 -j ACCEPT
sudo netfilter-persistent save

# Update the package list and install Python 3 and pip
sudo apt update
sudo apt install -y python3-pip

# Install virtualenv and virtualenvwrapper
pip3 install virtualenv virtualenvwrapper

# Configure virtualenvwrapper in .bashrc
echo 'export WORKON_HOME=~/envs' >> ~/.bashrc
echo 'export PATH=$PATH:/home/ubuntu/.local/bin' >> ~/.bashrc
echo 'export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3' >> ~/.bashrc
echo 'export VIRTUALENVWRAPPER_VIRTUALENV_ARGS=" -p /usr/bin/python3 "' >> ~/.bashrc
echo 'source /home/ubuntu/.local/bin/virtualenvwrapper.sh' >> ~/.bashrc
source ~/.bashrc

# Create a new virtual environment
mkvirtualenv flask01

# Install Flask
pip3 install Flask

# Clone your Flask project repository
git clone <your project>

cd <project-name>
pip install -r requirements.txt if needed

# Activate the virtual environment if not active
workon flask01

# Set up the environment variable for your Flask app
export FLASK_APP=app.py

# Run the Flask app with Gunicorn (install Gunicorn if not already installed)
pip3 install gunicorn

sudo /home/ubuntu/envs/flask01/bin/gunicorn -w 4 -b 0.0.0.0:5000 app:app

If everything is working fine, Ctrl + C and install nginx


# Install Nginx:

sudo apt-get update
sudo apt-get install nginx

Create a Configuration File for Your App:
Create a new Nginx server block configuration file. You can name it something like demo_app.conf:

sudo nano /etc/nginx/sites-available/demo_app.conf

# Configure Nginx to Redirect HTTP Requests to Gunicorn:

Add the following configuration to the file, replacing placeholders as needed:
Replace "your_domain.com" with your actual domain


server {

   listen 80;
   
   server_name DOMAIN NAME;  # Your domain name

   location / {
   
       proxy_pass http://127.0.0.1:5000;  # Gunicorn binding address and port
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       # Additional proxy settings if needed
       
   }
}


# Enable the Configuration and Test:

Create a symbolic link to enable the Nginx configuration:
sudo ln -s /etc/nginx/sites-available/demo_app.conf /etc/nginx/sites-enabled


Test the Nginx configuration to ensure there are no syntax errors:
sudo nginx -t


If the test is successful, restart Nginx to apply the new configuration:
sudo service nginx restart



# Make automatically start up on boot
sudo nano /etc/systemd/system/flask-app.service

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

# Save

sudo systemctl daemon-reload

sudo systemctl enable flask-app
sudo systemctl start flask-app

sudo systemctl status flask-app

Now, Gunicorn will start automatically on machine startup using the systemd service. You can also manage the service with commands like stop, restart, and status using systemctl.


Check if everything is working properly


# 2. Create Custom Image, Instance C0nfiguration, Instance Pool and Autoscaling.

1. Create Custom Image from the Server -> Wait till running and check if u can see the webpage: <publicip:5000>

2. From the Current Instance select Create Instance Configuration

3.
Create Instance Pool with the freshly created Instance Config -> Choose 2 Instances -> Next
Configure first AD and than add a second one
Select Attach Load Balancer -> Select the Load Balancer, Backedn Set and Port 80 -> Next + Create

4. 
In Compute select Autoscaling Configurations
Choose Instance Pool
Metric Based Autoscaling -> -> CPU Utilization -> Scale Out Rule > 70 -> Scale Down Rule < 20 -> Min. 1 | Max. 2 | Start 2

5.
Terminate the first Instance and the one we created with the custom image.
