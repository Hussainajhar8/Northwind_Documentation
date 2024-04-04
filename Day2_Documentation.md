# Automation of Database script 

* To get more insight into what the error meant and why the script wouldn't run we sat on a call together with a shared screen whilst one person manually ran the commands from the script until we hit an error. 

* The first blocker came from us not being able to run the below commands from the terminal like it would in the script:
 ![alt text](<img/Blocker - sudo -e.png>)

* We had to actually enter into my SQL and run the commands without the sudo -e, we then created a user, database and granted privileges 

* We were then running into an issue where we did have to manually enter the password after we'd quit MySQL and tried to run the next steps in the terminal which would obviously cause an issue for automation.

* We later found that the '-e' in the script version is what would allow us to bypass having to manually enter the password when running the final script 

* We also realised we could now remove the 'sleep' command as it wasn't needed.