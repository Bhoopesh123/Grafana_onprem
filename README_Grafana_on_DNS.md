# Grafana over DNS  
Install Grafana Stack on-prem on Ubuntu Box  

    Reference Documentation: https://grafana.com/docs/grafana/latest/setup-grafana/installation/

# 1. Install Grafana on Ubuntu Server:  
Install the prerequisite packages 

    sudo apt-get install -y apt-transport-https software-properties-common wget

Import the GPG key 

    sudo mkdir -p /etc/apt/keyrings/
    wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null

Add a repository for stable releases 

    echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
    echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com beta main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

Updates the list of available packages 

    sudo apt-get update
    sudo apt-get install grafana

# 2. Start the Grafana server 
Start it by following  commands.

    sudo systemctl daemon-reload
    sudo systemctl start grafana-server
    sudo systemctl status grafana-server

Configure the Grafana server to start at boot using systemd

    sudo systemctl enable grafana-server.service
    sudo systemctl restart grafana-server

Enable below two inbound ports to the VM: 

    3000
    80

# 3. Configure it Publicly via DNS 
certbot is an open-source program used to manage LetsEncrypt certificates, and snapd is a tool that assists in running certbot and installing the certificates. 

    sudo apt-get install snapd
    sudo snap install core; sudo snap refresh core
    sudo apt-get remove certbot
    sudo snap install --classic certbot
    sudo ln -s /snap/bin/certbot /usr/bin/certbot
    sudo certbot certonly --standalone
    sudo ln -s /etc/letsencrypt/live/subdomain.mysite.com/privkey.pem /etc/grafana/grafana.key
    sudo ln -s /etc/letsencrypt/live/subdomain.mysite.com/fullchain.pem /etc/grafana/grafana.crt
    sudo chgrp -R grafana /etc/letsencrypt/*
    sudo chmod -R g+rx /etc/letsencrypt/*
    sudo chgrp -R grafana /etc/grafana/grafana.crt /etc/grafana/grafana.key
    sudo chmod 400 /etc/grafana/grafana.crt /etc/grafana/grafana.key
    sudo chmod 644 grafana.key grafana.crt
    ls -l /etc/grafana/grafana.*

# 4. Configure Grafana HTTPS and restart Grafana 

Open the grafana.ini file and edit the following configuration parameters 

    cd /etc/grafana
    sudo vim grafana.ini

    [server]
    http_addr =
    http_port = 3000
    domain = mysite.com
    root_url = https://subdomain.mysite.com:3000
    cert_key = /etc/grafana/grafana.key
    cert_file = /etc/grafana/grafana.crt
    enforce_domain = False
    protocol = https

    sudo systemctl restart grafana-server
    sudo systemctl status grafana-server
    journalctl -u grafana-server -f

# 5. Access the grafana by below username and password:  

    url: https://subdomain.mysite.com:3000
    username: admin
    password: admin

Change the password after first login
