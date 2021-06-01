# Pen Testing the Cloud Environment

In this scenario, I acted as a cloud architect that has been tasked with setting up an ELK server to gather logs for the Incident Response team.
Before I hand over the server to the IR team, my senior architect has asked that I verify the ELK server is working as expected and pulling both logs and metrics from the pen-testing web servers.
I have three tasks:
  - Generate a high amount of failed SSH login attempts and verify that Kibana is picking up this activity.
  - Generate a high amount of CPU usage on the pen-testing machines and verify that Kibana picks up this data.
  - Generate a high amount of web requests to your pen-testing servers and make sure that Kibana is picking them up.

# SSH Barrage
To start, I attempted to login with an incorrect username `ssh sysadmin@10.0.0.9` 
