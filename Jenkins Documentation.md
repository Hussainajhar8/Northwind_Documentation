# Jenkins

## Server Config

This script will install Jenkins and the required depencencies for running and testing the Python Flask Application

Note, that Jenkins will require some manual set up as well as three plugins.
- git parameter
- github
- ssh agent

<br>

```bash
# add jenkins key + repo
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null  

# update with new repo
sudo apt-get update

# upgrade
sudo apt-get upgrade -y
  
# install jenkins
sudo apt-get install jenkins

# jenkins depencencies
sudo apt install fontconfig openjdk-17-jre -y

# enable and start jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins

# login to jenkins web portal and complete set up
# jenkins plugins required:
#     git parameter
#     github plugin
#     ssh agent

# these are the depencences for testing and running the python application

# Install Python 3.9 and set it as default
sudo apt-get install python3.9 -y
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.9 1
sudo update-alternatives --config python3

# install pip
sudo apt install python3-pip -y

# Waitress 3.0.0
sudo pip install waitress==3.0.0

# Flask 3.0.2
sudo pip install Flask==3.0.2

# Flask-sqlalchemy 3.1.1
sudo pip install Flask-SQLAlchemy==3.1.1

# SQLAlchemy 2.0.27
sudo pip install SQLAlchemy==2.0.27

# PyMySQL 1.1.0
sudo pip install pymysql==1.1.0

# Jinja 3.1.3
sudo pip install Jinja2==3.1.3

# Flask-Waitress 0.0.1
sudo pip install Flask-Waitress==0.0.1

# cffi
sudo pip install cffi

# pytest
sudo pip install pytest
```

## Job Set Up
There are three Jenkins jobs

- test
	- This runs the unit tests when a commit is pushed to dev
    - Git Pulls from the dev branch
- dev to main
	- Merges the dev branch to main
    - Git pulls from the dev branch
- update aws web
	- rsyncs the new files from main to the server, stop and restarts the waitress process
    - Git pulls from the main branch

![alt text](img/jenkins_jobs.png)

## Git Set Up

![alt text](img/jenkins_git.png)

All three jobs are set up with GitHub, the test and merge job pull from the dev branch. And the aws update from main.

## Webhook on the test job

![alt text](img/jenkins_webhooks.png)

The first jobs, test, will be triggered by a webhook, so that it runs when a commit to the dev branch is made.

The webhook on GitHub should be configured like this:

![alt text](img/github_webhook.png)

## Test job script

![alt text](img/jenkins_test_script.png)

This script runs within the test job, it just runs the pytest tests within the tests directory.

```bash
cd tests
python3 -m pytest
```

## Merge Job

![alt text](img/jenkins_merge.png)

The merge job is triggered after the test, and merges the dev branch into the main branch.

## AWS Update

![alt text](img/jenkins_aws.png)

The AWS Update job runs after the merge, it rsyncs the code across (rsync only transfers the files with changes), then SSH'es into the web server, kills the waitress process and then restarts it, making sure to set the correct enviroment variables.

```bash
rsync -avz -e "ssh -o StrictHostKeyChecking=no" . ubuntu@ec2-18-200-242-172.eu-west-1.compute.amazonaws.com:~/repo/northwind_python_app-main

ssh -o "StrictHostKeyChecking=no" ubuntu@ec2-18-200-242-172.eu-west-1.compute.amazonaws.com <<EOF
	cd repo/northwind_python_app-main/
    pkill waitress
    export FLASK_APP=northwind_web.py
    #export DB_CONNECTION_URI=mysql+pymysql://group_2:password@3.255.110.130:3306/northwind
    export DB_CONNECTION_URI=mysql+pymysql://group_2:password@3.254.199.133:3306/northwind
    
    nohup waitress-serve --port=5000 northwind_web:app > waitress_output.log 2>&1 &
```
