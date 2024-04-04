# Setting up Northwind application

## Introduction
This document presents our strategy for deploying the Northwind Python application onto cloud-based infrastructure while adhering to DevOps principles. The application includes a homepage, a customer listing page, and a page for adding new customers. Our objective is to deploy the application efficiently, leveraging contemporary tools and methodologies acquired during our training. We aim to deliver a scalable, resilient, and automated deployment solution that aligns with modern software development practices. By utilising advanced DevOps tools, we seek to streamline the deployment process and enhance overall efficiency. Initially, we will manually validate functionality before progressing to the development of scripts and automation.

**Architecture Overview**

![alt text](img/app-img/image-10.png)
Our deployment strategy involves implementing a two-tier architecture, separating the application layer from the database layer. This architecture enhances scalability, maintainability, and security.

## Toolset

Our deployment process incorporates the following advanced DevOps tools:

**Jenkins**: As our CI/CD automation server, Jenkins automates the build, test, and deployment phases of our application. Jenkins enables us to create robust pipelines that automatically trigger deployments upon code updates, ensuring rapid and reliable delivery of new features.


**AWS**: Amazon Web Services (AWS) serves as the foundation for our cloud infrastructure, offering a diverse range of scalable and reliable services. AWS services such as EC2 for compute resources are utilised to build a robust and resilient deployment environment.

## If time:

**Ansible**: Utilised for automation, Ansible streamlines the provisioning and configuration of infrastructure components. Its simplicity and versatility make it an ideal choice for orchestrating our deployment process.

**Terraform**: We employ Terraform for infrastructure as code (IaC), enabling us to define our cloud infrastructure in a declarative manner. By codifying our infrastructure, we can easily replicate environments, enforce consistency, and manage infrastructure changes more effectively.

## Setting up
To begin, we will set up our EC2 instance and open ports 20, 80, and 5000 on Ubuntu 20.24, then install the required dependencies and launch our app.

### Step 1: Create EC2 Instance
Launch an EC2 instance and ensure ports 20, 80, and 5000 are open in the security group.
![Create EC2 Instance](img/app-img/image.png)
![Security Group Configuration](img/app-img/image-1.png)

### Step 2: SSH into VM
Once the instance is running, SSH into the VM.

### Step 3: Update System
- `sudo apt-get update`: Update the package lists for upgrades.
- `sudo DEBIAN_FRONTEND=noninteractive apt-get upgrade -y`: Upgrade installed packages non-interactively.

### Step 4: Install Dependencies
- `sudo DEBIAN_FRONTEND=noninteractive apt-get install git nginx python3.9 python3-pip -y`: Install Git, Nginx, Python 3.9, and Pip.

### Step 5: Configure Nginx
- `sudo systemctl enable nginx`: Enable Nginx on system boot.
- `sudo sed -i "s@try_files \$uri \$uri/ =404;@proxy_pass http://127.0.0.1:5000;@" /etc/nginx/sites-available/default`: Update Nginx configuration to redirect port 3000 to 80.
- `sudo systemctl restart nginx`: Restart Nginx to apply changes.

### Step 6: Check Python version and set as default
Run the following commands to set 3.9 as the default version in our instance:
- `sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.9 1`
- `sudo update-alternatives --config python3`
Now check the default version.
![Python Version](img/app-img/image-2.png)

### Step 7: Install Python Packages
Use Pip to install required Python packages:
```bash
  # Waitress 3.0.0 - ran
sudo pip install waitress==3.0.0

# Flask 3.0.2 - ran
sudo pip install Flask==3.0.2

# Flask-sqlalchemy 3.0.2 - ran
sudo pip install Flask-SQLAlchemy==3.1.1

# SQLAlchemy 2.0.27 - ran
sudo pip install SQLAlchemy==2.0.27

# PyMySQL 1.1.0 - ran
sudo pip install pymysql==1.1.0

# Jinja 3.1.3 - ran
sudo pip install Jinja2==3.1.3

# Flask-Waitress 0.0.1 - ran
sudo pip install Flask-Waitress==0.0.1

# Install CFFI - ran
sudo pip install cffi

  ```
Check the versions for the python dependencies.
![Python Dependencies](img/app-img/image-3.png)

### Step 8: Clone Repository
- Create a directory for the repository: `mkdir repo`.
- Navigate into the directory: `cd repo/`.
- Clone the repository: `git clone https://github.com/Hussainajhar8/northwind_python_app-main`.

### Step 9: Set Flask App Environment Variable and Run Application
- Navigate into the cloned repository: `cd northwind_python_app-main/`.
- Set the FLASK_APP environment variable: `export FLASK_APP=northwind_web.py`.
- Run the application: `waitress-serve --port=5000 northwind_web:app`.
Confirm it is running.
![Application Running](img/app-img/image-4.png)
![Application Running](img/app-img/image-5.png)

### Step 10: Set Database Connection URI
Export the DB_CONNECTION_URI environment variable: `export DB_CONNECTION_URI='mysql+pymysql://<user>:<password>@<ip-of-db>:3306/northwind'`.
![Database Connection](img/app-img/image-6.png)

### Step 11: Restart Application
Restart the application to apply changes.
Confirm it is running as expected.
![Application Restarted](img/app-img/image-7.png)
![Application Restarted](img/app-img/image-8.png)

### Final Script

```bash
#!/bin/bash

# upgrade - ran
sudo apt-get update

# update - ran
sudo DEBIAN_FRONTEND=noninteractive apt-get upgrade -y

# install git - ran
sudo DEBIAN_FRONTEND=noninteractive apt-get install git -y

# install nginx - ran
sudo DEBIAN_FRONTEND=noninteractive apt-get install nginx -y

# enable nginx - ran
sudo systemctl enable nginx

# update nginx to redirect port 3000 to port 80 -
sudo sed -i "s@try_files \$uri \$uri/ =404;@proxy_pass http://127.0.0.1:5000;@" /etc/nginx/sites-available/default

# restart nginx - ran
sudo systemctl restart nginx

# Install Python 3.9 and set it as default
sudo apt-get install python3.9 -y
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.9 1
sudo update-alternatives --config python3

# Install pip
sudo apt install python3-pip -y

# Waitress 3.0.0 - ran
sudo pip install waitress==3.0.0

# Flask 3.0.2 - ran
sudo pip install Flask==3.0.2

# Flask-sqlalchemy 3.0.2 - ran
sudo pip install Flask-SQLAlchemy==3.1.1

# SQLAlchemy 2.0.27 - ran
sudo pip install SQLAlchemy==2.0.27

# PyMySQL 1.1.0 - ran
sudo pip install pymysql==1.1.0

# Jinja 3.1.3 - ran
sudo pip install Jinja2==3.1.3

# Flask-Waitress 0.0.1 - ran
sudo pip install Flask-Waitress==0.0.1

# Install CFFI - ran
sudo pip install cffi

# make repo directory - ran
mkdir repo

# cd into repo - ran
cd repo/

# git clone repo
git clone https://github.com/Hussainajhar8/northwind_python_app-main

# cd into northwind_python_app
cd northwind_python_app-main/

# set FLASK_APP environment variable
export FLASK_APP=northwind_web.py

# set DB var export DB_CONNECTION_URI='mysql+pymysql://<user>:<password>@<ip-of-db>:3306/northwind'
export DB_CONNECTION_URI='mysql+pymysql://ajhar:password@63.33.55.80:3306/northwind'

# run waitress
waitress-serve --port=5000 northwind_web:app
```

## Blockers:

The code seemed to have been going in a loop and the app wasn't working on 3.9. So we tried again on Python 3.8 and got the app working. This happened because our pip installs were being installed into the python3.8 folder instead of the 3.9. So when running the app with `waitress-serve --port=5000 northwind_web:app` it couldn't find the required packages.

We were able to get it working using 3.8 but also had to change it to Flask-SQLAlchemy==3.1.1 alt text

We then found that our VM installed everything into the default Python path of 3.8. Although we had 3.9 installed. As the requirements asked for 3.9, we had to find a way around this. Our updated script below has the new section that addresses these titles: Install Python 3.9 and set it as default.

```bash
# 8. Install Python 3.9 and set it as default
sudo apt-get install python3.9 -y
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.9 1
sudo update-alternatives --config python3
```

We also found that we had to install a dependency called cffi otherwise we got the below error.

![alt text](img/app-img/image-9.png)

After installing cffi with `sudo pip install cffi `on the server it worked fine.
