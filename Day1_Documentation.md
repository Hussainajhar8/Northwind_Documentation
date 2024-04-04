# Manually setting up the database

1. Get an EC2 created manually using a Ubuntu 20.04 image
2. Use the same configurations as previous database vms
3. Create a security group with 80, 22 and 3306 port access
4. Ensure it's outbound rules allowed 'all' traffic so it can access packages etc for installation
   
*We ran the below commands manually to get the database working*





# Automating the process

We then used a terraform script to launch an instance with the configurations needed (as detailed above)

1. When SSH'd into the EC2 we began to put a script together using our manual commands
   *The first script was as follows:*

![alt text](<Manual Commands for DB.png>)

![alt text](<First DB automation script.png>)

2. We then used 'sudo nano' to create a file to run the script. 
3. Used the 'chmod' command so it was ready to run.
4. Ran the script and got an error at the end for getting MySQL to run so that's the main focus for tomorrow.
