# Pen Testing the Cloud Environment

In this scenario, I acted as a cloud architect that has been tasked with setting up an ELK server to gather logs for the Incident Response team.
Before I hand over the server to the IR team, my senior architect has asked that I verify the ELK server is working as expected and pulling both logs and metrics from the pen-testing web servers.
I have three tasks:
  - Generate a high amount of failed SSH login attempts and verify that Kibana is picking up this activity.
  - Generate a high amount of CPU usage on the pen-testing machines and verify that Kibana picks up this data.
  - Generate a high amount of web requests to your pen-testing servers and make sure that Kibana is picking them up.

# SSH Barrage
To start, I attempted to login with an incorrect username `ssh sysadmin@10.0.0.9` multiple times.  Then, I monitored the failed attempts through the Kibana log stream as seen below.
![SSH_Barrage.png](https://github.com/w0lfboy/Cloud-Security/blob/main/Pen%20Testing%20the%20Cloud%20Environment/SSH%20barrage.png)

Then, to make it more interesting, I barraged all three Web VMs at once.
![SSH_Barrage2.png](https://github.com/w0lfboy/Cloud-Security/blob/main/Pen%20Testing%20the%20Cloud%20Environment/SSH%20barrage2.png)

we can use a `for` loop to make this quicker and easier.
```
$ for i in 10.0.0.9 10.0.0.8 10.0.0.13; do ssh fakelogin@$i; done
```
*Based off of our above logs, Kibana is effectively monitoring failed ssh attempts*

# Linux Stress
To start I load and attach to my ansible container.
``` 
$ sudo docker start angry_engelbart
$ sudo docker attach angry_engelbart
```
SSH into `Web-1 VM` `ssh redadmin@10.0.0.9`
Run `$ sudo apt install stress` to acquire the stress program
Run `$ sudo stress --cpu 1` 

I then moved to the Metric page in Kibana.  After waiting 2 minutes, we can see that the system has the CPU usage has increased to nearly 100% by the user as seen below.
![linux_stress.png](https://github.com/w0lfboy/Cloud-Security/blob/main/Pen%20Testing%20the%20Cloud%20Environment/linux%20stress.png)

We can do the same stress test on all three Web VM's.  See below for evidence via Kibana metrics on Web-2 and Web-3.
![linux_stress_web2](https://github.com/w0lfboy/Cloud-Security/blob/main/Pen%20Testing%20the%20Cloud%20Environment/linux%20stress%20web2.png)
![linux_stress_web3](https://github.com/w0lfboy/Cloud-Security/blob/main/Pen%20Testing%20the%20Cloud%20Environment/linux%20stress%20web3.png)

# wget-DoS
We can use the wget command to provide a DoS attack to our Web VM's.  
Using a simple `for` loop to wget all of the VM's at once.
```
$ for i in 10.0.0.9 10.0.0.8 10.0.0.13; do wget $i; done
```
*Note, this creates an insane amount of index.html files on our desktop.  To remove them simply use the command `rm index*`*

We can use the `-O` option to send our information to a specific file instead of creating a massive amount index.html files.  
```
$ for i in 10.0.0.9 10.0.0.8 10.0.0.13; do wget -O logfile $i; done
```

We can see the network traffic shows our DoS attack due to the increased volume on the network.
![wget_dos](https://github.com/w0lfboy/Cloud-Security/blob/main/Pen%20Testing%20the%20Cloud%20Environment/wget%20dos.png)

And that completes our penetration test!

