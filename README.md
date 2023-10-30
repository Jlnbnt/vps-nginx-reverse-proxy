# VPS Setup

### 0 . Initial steps

```bash
# Update
sudo apt update

# Upgrade
sudo apt upgrade
```

### 1 . Users

```bash
# Add a user (1 per project)
sudo adduser <username>

# Grant root permission's
sudo usermod -aG sudo <username>

# Change root password
sudo passwd root

# Logout then back in with the newly created user and
# Remove default user
sudo userdel -r <username>
```

### 2 . Hostname

```bash
# Replace the default VPS hostname
sudo hostnamectl set-hostname <newHostname>

# Replace the default VPS hostname
sudo nano /etc/hosts
```

### 2 . SSH

```bash
# Open the sshd_config
# Remove the comment before #Port 22 and replace with a new port
sudo nano /etc/ssh/sshd_config

# Restart sshd services
sudo systemctl restart sshd

# Reboot the VPS
sudo reboot
```

**Note :**

> You will now have to specify the port on each connection
> ex : user@ipAddress -p 50000

### 3 . Languages

```bash
# Set the default system language
sudo dpkg-reconfigure locales
```

> **Note :**
> Add the needed UTF-8 (fr-FR, en-GB) and set to en-GB by default

### 4 . Firewall

```bash
# Install UFW
sudo apt install ufw

# Enable UFW
sudo ufw enable

# Allow default port 22 (VPS rescue boot)
sudo ufw allow ssh

# Allow newly defined port (from step 2)
sudo ufw allow <newSshPort>

# (Optional) Disable UFW logging
sudo ufw logging off
```

### 5 . Fail2ban

```bash
# Install Fail2ban
sudo apt install fail2ban

# Check Fail2ban status
sudo systemctl status fail2ban

# Check if defaults-debian.conf is present, nano into it and check that enabled === true
cat /etc/fail2ban/jail.d/defaults-debian.conf

# Restart FAIL2BAN services
sudo service fail2ban restart
```

### 6 . Nginx

```bash
# Install Nginx
sudo apt install nginx -y

# Remove default config
sudo rm -rf /var/www/html
sudo rm /etc/nginx/sites-available/default
sudo rm /etc/nginx/sites-enabled/default
```

For each domain / subDomain create a config file

```bash
sudo nano /etc/nginx/sites-available/<domain>
```

and add the following :

```
# Port 80 is used for incoming http requests
# Port 443 is used for incoming https requests

server {
    # The URLs we want config to apply to
    server_name <domain> www.<domain>;

    # Mapping Docker containers to specified port ( here => localhost:3200 )
    location / {
      proxy_pass http://127.0.0.1:3200;
    }
}
```

```bash
# Create a symbolic link for the configs
sudo ln -s /etc/nginx/sites-available/domain.com /etc/nginx/sites-enabled
```

> **Note :**
> If a config already exist, remove it before creating the symbolic link

```bash
# Check that Nginx is running
sudo nginx -t

# Restart Nginx services
sudo systemctl restart nginx
```

> **Note :**
> You might need to clear the browser's cache and reset Nginx between changes

> **Note :**
> You will have to do the above manipulation for each domain / subdomain

```bash
# Add Nginx to UFW
sudo ufw allow 'Nginx Full'
```

### 7 . SSL Certificates

```bash
# Temporarily disable UFW
sudo ufw disable

# Install Certbot
sudo apt install certbot
sudo apt install python3-certbot-nginx

# Creating certificates
sudo certbot --nginx

# Re-enable UFW
sudo ufw enable
```

> **Note :**
> You will be prompted to enter the domains for which you need to generate a certificate. Leave empty and hit enter (it will generate a certificate for every domain / subdomain present in /etc/nginx/sites-available)

> **Notes :**
> Renewals should be automated and manage by a cron : /etc/cron.d/certbot

### 8 . Git

```bash
# Install Git
sudo apt install git

# Generate an SSH key pair for each project
ssh-keygen -t rsa

# Add the SSH key to Gitlab
cat ~/.ssh/id_rsa.pub
```

### 9 . Projects

Clone the relevant projects into their respective users

```bash
git clone <urlFromGitLab>
```

### 10 . Docker

> Make sure to install the correct version of Docker.

> ex : Docker : 20.X

> ex : Docker-compose : 1.29.X

```bash
# Install required packages
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
sudo apt install gnupg

# Add Docker GPG key
sudo curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Add Docker repository
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update
sudo apt update

# Install Docker Engine
sudo apt install -y docker-ce=5:20.10.12~3-0~debian-$(lsb_release -cs) docker-ce-cli=5:20.10.12~3-0~debian-$(lsb_release -cs) containerd.io

# Download the binary
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Add permissions
sudo chmod +x /usr/local/bin/docker-compose

# Check for the right versions
docker --version
docker-compose --version

# For each project, grant the docker permission to the user
sudo usermod -aG docker <username>
```

# Useful commands

### Users

```bash
# Add a user
sudo adduser <username>

# Change user's password
sudo passwd <username>

# List user's groups
sudo groups <username>

# Grant permission to user
sudo usermod -aG <groupname> <username>

# Delete a user
sudo userdel -r <username>

# List users
cut -d: -f1 /etc/passwd

# Login to an other user
sudo su - <username>
```

### Folders, files

```bash
# Rename a folder, file
sudo mv <oldName> <newName>

# Move a folder, file
sudo mv <fileName> <destinationDirectory>

# Remove a folder, file
sudo rm -rf <fileName>

# Copy a folder, file
sudo cp <fileName> <destinationDirectory>
```

### Nginx

```bash
# Install Nginx
sudo apt install nginx -y

# Check Nginx status
sudo systemctl status nginx

# Restart Nginx services
sudo systemctl restart nginx.service

# Stop Nginx services
sudo systemctl stop nginx

# Remove Nginx
sudo apt purge nginx nginx-common nginx-full -y
```

> **Note :**
> By default Nginx is utilizing port 80. You may want to stop Nginx services to have access to port 80.

### VPS

```bash
# Replace the default VPS hostname
sudo hostnamectl set-hostname <newHostname>

# Reboot the VPS
sudo reboot
```

If you need to enter the rescue mode :

1 . Restart the VPS in rescue mode from the VPS's dashboard

2 . Log to root@vpsIpAddress

```bash
# List the available volumes
lsblk

# Mount main volume (The one with most disk space)
mount /dev/sdb1 /mnt

# Chroot into it
chroot /mnt

# Perform tasks (reverse what has gone wrong)
cd /home/<user>
```

### UFW

```bash
# Install UFW
sudo apt install ufw

# Enable UFW
sudo ufw enable

# Allow port
sudo ufw allow <newSshPort>*

# Check UFW status
sudo ufw status verbose

# Delete UFW rule
sudo ufw status numbered
sudo ufw delete <number>

# Check what services are available
sudo ufw app list

# Disable UFW logging
sudo ufw logging off
```

### FAIL2BAN

```bash
# Check if defaults-debian.conf is present, nano into it and check that enabled === true
cd /etc/fail2ban/jail.d && ls

# Restart FAIL2BAN services
sudo service fail2ban restart
```

### CERTBOT

```bash
# Install Certbot
sudo apt install certbot
sudo apt install python3-certbot-nginx

# Add a certificate for a domain
sudo certbot --nginx -d domain.com

# List certificates
sudo certbot certificates

# Delete a certificate
sudo certbot delete --cert-name domain.com

# Renew certficates
sudo certbot renew --nginx
```

### DOCKER

```bash
# Remove all containers
docker stop $(docker ps -a -q) && docker rm $(docker ps -a -q)

# Remove all images
docker system prune --all --force

# Remove and clean docker & docker-compose
sudo apt remove docker-ce docker-ce-cli containerd.io

sudo apt purge docker-ce docker-ce-cli containerd.io

sudo rm /usr/local/bin/docker-compose

sudo rm /etc/apt/sources.list.d/docker.list
sudo apt update
sudo apt autoremove
sudo apt autoclean

# Check container's logs
docker logs <container> -f

# Restart Docker services
sudo systemctl restart docker

# Removes all unused images
docker system prune -a

# Removes all unused volumes
docker system prune --volumes

# Cleanup all resources
docker system prune
```

### PRISMA

**Note :**

> The below instructions might fix the prisma's "citext" problem.

```bash
# Apply migrations
docker exec <backend-container-name> yarn prisma migrate deploy

docker exec <backend-container-name> yarn prisma:generate

docker exec <backend-container-name> yarn db:push --accept-data-loss

# Switch into SQL mode
docker exec -it <database-container-name> psql -U <postgresUser> -d <databaseName>

# Add a User entry
INSERT INTO "User" (uuid, email, "password", "userName", roles)
VALUES (
'uuid',
'email@email.email',
'password',
'userName',
'{"Role1", "Role2"}'
);
```

### DEPLOYMENT

#### Backend

```bash
# Log as a backend user
sudo su - backend
# Go into the project folder
cd backend
# Makes sure to be on the right branch
git branch
# Pull the changes
git pull origin <branchName>
# Build backend's Docker container
docker-compose -f docker-compose.<container-name>.yml -f docker-compose.<container-name>.yml up --build -d
# Update db
docker exec <database-container-name> yarn db:push
# Logout
exit
```

#### Frontend

```bash
# Log as a frontend user
sudo su - frontend
# Go into the project folder
cd frontend
# Makes sure to be on the right branch
git branch
# Pull the changes
git pull origin dev
# Build frontend's Docker container
docker-compose up --build -d
# Logout
exit
```

### GENERAL

> **Notes :**
> By default, NextJs is running its apps on port 3000. That is why inside each NextJs app encapsulated inside a Docker container run on port 3000. We then map the port 3000 to an open Docker container's port.

> Do not run sudo apt upgrade, it will update docker version.

### LOCAL DEVELOPMENT

Update the git config :

```bash
nano ~/.ssh/config
```

following this model :

```
 Host gitlab.<repositoryName.com>
    HostName gitlab.com
    User git
    IdentityFile ~/.ssh/<projectName>/id_rsa
```

then in each project folder, set the remote url as follow :

```bash
git remote set-url origin git@gitlab.<repositoryName.com>:<path/to/project.git>
```
