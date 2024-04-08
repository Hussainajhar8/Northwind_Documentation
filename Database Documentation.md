# Two Tier Web Application

## The Database

Our task was to create a two-tier web application that interacts with a MySQL database. The database schema was provided in the form of a SQL script. The schema is based on the Northwind database, a sample database used by Microsoft for tutorials and demonstrations.

We are using an AWS EC2 instance running Ubuntu 20.04 to host the MySQL database. The database is accessible by the application, which is running on a second AWS EC2 instance.

Things to consider when setting up the database:

- The database should be configured to allow remote connections.
- The database should be populated with the schema provided.
- A new user should be created with the necessary privileges to interact with the database.

## Manually setting up the database

Initially, we set up the database manually to understand the process and the commands required. This would help us automate the process later.

1. Get an EC2 created manually using a Ubuntu 20.04 image
2. Use the same configurations as previous database vms
3. Create a security group with 80, 22 and 3306 port access
4. Ensure it's outbound rules allowed 'all' traffic so it can access packages etc for installation
   
When SSH'd into the EC2 we began to put a script together using our manual commands:

```bash
# Update package lists
sudo apt update -y

# Upgrade packages non-interactively, and automatically handle prompts
sudo DEBIAN_FRONTEND=noninteractive apt-get upgrade -y

# install git - ran
sudo DEBIAN_FRONTEND=noninteractive apt-get install git -y

# Install MySQL Server
sudo apt-get update

sudo apt-get install mysql-server

sudo systemctl enable mysql

sudo systemctl restart mysql

# Allow connections
sudo sed -i 's/^bind-address\s*=.*/bind-address = 0.0.0.0/' /etc/mysql/mysql.conf.d/mysqld.cnf
sudo systemctl restart mysql.service

sudo systemctl restart mysql

# Clone the repository
mkdir -p ~/repo && cd $_
git clone https://github.com/followcrom/northwind_python_app.git

# Import the database schema
sudo mysql -u root -proot < ~/repo/northwind_python_app/northwind_sql.sql
```

2. We then used 'sudo nano' to create a file to run the script. 
3. Used the 'chmod' command so it was ready to run.


## Finished script

This can run on a fresh Ubuntu 20.04 server to install MySQL Server, configure it for remote connections, clone the repository containing the database schema, import the database schema, and create a new MySQL user 'group_2' with the necessary privileges.

```bash
#!/bin/bash

# Update and upgrade packages
sudo apt update -y
sudo DEBIAN_FRONTEND=noninteractive apt-get upgrade -y

# Install MySQL Server
sudo apt-get install -y mysql-server

# Configure MySQL Server for non-interactive secure installation
sudo mysql -e "ALTER USER 'root'@'localhost' IDENTIFIED BY '********';"
sudo mysql -e "FLUSH PRIVILEGES"

# Enable and restart MySQL service
sudo systemctl enable mysql
sudo systemctl restart mysql

# Configure MySQL to allow remote connections
MYSQL_CONF="/etc/mysql/mysql.conf.d/mysqld.cnf"
sudo sed -i 's/^bind-address\s*=.*/bind-address = 0.0.0.0/' $MYSQL_CONF
sudo systemctl restart mysql

# Clone the repository containing the application
mkdir -p ~/repo && cd $_
git clone https://github.com/followcrom/northwind_python_app.git

# Import the database schema
sudo mysql -u root -p***** < ~/repo/northwind_python_app/northwind_sql.sql

# Create a new MySQL user 'group_2' and grant privileges
sudo mysql -u root -p***** -e "CREATE USER IF NOT EXISTS 'group_2'@'*****' IDENTIFIED BY 'password';"
sudo mysql -u root -p***** -e "GRANT CREATE, ALTER, DROP, INSERT, UPDATE, DELETE, SELECT, REFERENCES, RELOAD on *.* TO 'group_2'@'*****'"
sudo mysql -u root -p***** -e "FLUSH PRIVILEGES;"
```

## Automating the Set-up

We wanted to automate the set-up process as much as possible. This would make it easier to deploy the application in different environments and reduce the chances of human error. To do this, we utilised Terraform to provision the EC2 instances, and included the above bash script as user data, which would run on the creation of the EC2 instance.

### Use Terraform

1, Run `terraform init` in a Terraform working directory. This command initializes the working directory and downloads any necessary plugins and modules.

2, `terraform plan` creates an execution plan. It will not make any changes to your infrastructure. It shows what actions Terraform will take to change the infrastructure.

3, `terraform apply` applies the changes required to reach the desired state of the configuration. It will prompt you to confirm the changes before making them.

## Blockers

We encountered a few blockers when setting up the database. One issue was configuring the MySQL server to allow remote connections. By default, MySQL only allows connections from the localhost. We had to modify the MySQL configuration file to allow connections from any IP address. This was done by changing the `bind-address` parameter in the MySQL configuration file to `

Creating a new user with the necessary privileges was also a challenge. We had to grant the application VM the necessary privileges to interact with the database. This was done using the `GRANT` statement in MySQL.

Installing MySQL with minimal commands was not the best way to do it. Ultimately, we had to add another step to define the root password. This was done using the `ALTER USER` statement in MySQL and has been added to the script.
