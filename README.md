# Cloud-Security
## This document contains the following details:
  - Description of the Topology
  - Access Policies
  - ELK Configuration
    - Beats in Use
    - Machines Being Monitored
  - How to Use the Ansible Build
  - ![Pen Testing the Cloud Environment](https://github.com/w0lfboy/Cloud-Security/tree/main/Pen%20Testing%20the%20Cloud%20Environment)
  - Additional Commands Used


# Cloud Network Diagram
The files in this repository were used to configure the network depicted below.

![Cloud_Security_ELK.png](https://github.com/w0lfboy/Cloud-Security/blob/main/Diagrams/Cloud%20Security%20ELK.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of playbooks may be used to install only certain pieces of it, such as Filebeat, Metricbeat, ELK installation, and DVWA playbooks.
  - [Metricbeat Playbook](https://github.com/w0lfboy/Cloud-Security/blob/main/Ansible/Metricbeat-Playbook.yml)
  - [Filebeat Playbook](https://github.com/w0lfboy/Cloud-Security/blob/main/Ansible/Filebeat-Playbook.yml)
  - [Install-Elk Playbook](https://github.com/w0lfboy/Cloud-Security/blob/main/Ansible/Install-Elk.yml)
  - [DVWA Playbook](https://github.com/w0lfboy/Cloud-Security/blob/main/Ansible/DVWA-Playbook.yml)

# Description of the Topology
The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

*Load balancing ensures that the application will be highly available, in addition to restricting an overload of traffic to the network.*

Our load balancer allows traffic to redirect between the three machines hosting the site to provide availability and security as needed to our site.  This is a crucial part of the **CIA triad** that can be done simply with the load balancer.

With our jump box, we can provide scaling and segmentation with ease.  Since all of our playbooks are located in the jump-box, we can easily create new machines to run whichever playbook we prefer.  By having one central location such as our jump box, we are also minimizing access controls. 

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the data and system logs.

*Why install Filebeat and Metricbeat?*

**Filebeat** allows us to monitor, locate, and ship log data to a central location (Kibana) where we can analyze it with ease.
**Metricbeat** allows us to output metrics and statistics to the same central location (Kibana) where we can again analyze it with ease.  

By combining both of these beats we can monitor security to prevent and respond to security incidents.
The configuration details of each machine may be found below. 

|         **Name**     |   **Function**  | IP Address (private) | IP Address (public) | **OS**|
|:--------------------:|:---------------:|:--------------------:|---------------------|:-----:|
|       Jump Box       |     Gateway     |       10.0.0.4       |    20.185.199.138   | Linux |
|         Web-1        |      Server     |       10.0.0.9       |     40.121.94.31    | Linux |
|         Web-2        |      Server     |       10.0.0.8       |     40.121.94.31    | Linux |
|         Web-3        |      Server     |       10.0.0.13      |     40.121.94.31    | Linux |
| Django VM  Elk Stack | Monitor servers |       10.1.0.4       |    52.251.117.56    | Linux |

# Access Policies
**The machines on the internal network are not exposed to the public Internet.**
Only the Jump Box Provisioner can accept connections from the Internet. Access to this machine is only allowed from the IP address 71.115.3.39

Machines within the network can only be accessed by the Jump Box Provisioner.

*The only machine allowed to access the ELK stack server is the Jump Box Provisioner (10.0.0.4).*

A summary of the access policies in place can be found in the table below.
|       **Name**      | Publicly Accessible |                  **Allowed IP Addresses**                 |
|:-------------------:|:-------------------:|:---------------------------------------------------------:|
|       Jump Box      |          Yes        |                 71.115.3.39 on SSH port 22                |
|        Web-1        |          No         | 10.0.0.4 on SSH port 22 and 71.115.3.39 via Load Balancer |
|        Web-2        |          No         | 10.0.0.4 on SSH port 22 and 71.115.3.39 via Load Balancer |
|        Web-3        |          No         | 10.0.0.4 on SSH port 22 and 71.115.3.39 via Load Balancer |
| Django VM ELK Stack |          No         |  10.0.0.4 on SSH port 22 and 71.115.3.39 on TCP port 5601 |
|    Load Balancer    |          No         |                71.115.3.39 on HTTP port 80                |

# ELK Configuration
Ansible was used to automate configuration of the ELK machine. By creating playbooks to suit our needs, we were able to quickly construct vm's that we would quickly install by running a given playbook.  Not only was this helpful for creating the ELK machine and the DVWA vm's, but we could create many more with our playbooks very quickly, or take them down quickly if for some reason one of the vm's is compromised.  Our ansible container also allowed us to monitor the status of all of our vm's at the same time.

*as a prerequisite, the virtual machine must first be created with the appropriate memory parameters (minimum of 4gb) and added to the ansible hosts file in a new group named "elk"  Once this is complete, we may install the playbook below.*

[Install-Elk Playbook](https://github.com/w0lfboy/Cloud-Security/blob/main/Ansible/Install-Elk.yml) Walkthrough
  - When creating a playbook, we first need to name it, create it `nano Install-Elk.yml`, and create a header.  In this instance, we named it `Install-Elk.yml`
    ```
       ---
         - name: Configure Elk VM with Docker
           hosts: elk
           remote_user: redadmin
           become: true
           tasks:
  - Now tasks are added to the playbook, starting with apt packages `Install docker.io` and `Install python3-pip`
    ```
       - name: Install docker.io
         apt:
           update_cache: yes
           force_apt_get: yes
           name: docker.io
           state: present
           
       - name: Install python3-pip
         apt:
           force_apt_get: yes
           name: python3-pip
           state: present
  -  We also had to install the following `pip` package that is the Python client for Docker.  This is required by Ansible to control the state of Docker containers.
     ```
        - name: Install Docker module
          pip:
            name: docker
            state: present
  -  Next, we had to increase system memory to `262144` using Ansible's `sysctl` module.  We also had to ensure that it will run automatically when the vm is restarted.
     ```
        - name: Increase virtual memory
          command: sysctl -w vm.max_map_count=262144

        - name: Use more memory
          sysctl:
            name: vm.max_map_count
            value: '262144'
            state: present
            reload: yes
  -  After the memory is increased, we actually had to include the Docker `elk` container! 
     ```
        - name: download and launch a docker elk container
          docker_container:
            name: elk
            image: sebp/elk:761
            state: started
            restart_policy: always
            published_ports:
              -  5601:5601
              -  9200:9200
              -  5044:5044
  -  The final step is to ensure that `docker` starts on a system reboot automatically.
     ```
        - name: Enable service docker on boot
          systemd:
            name: docker
            enabled: yes

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.
![docker_ps.png](https://github.com/w0lfboy/Cloud-Security/blob/main/Images/docker%20ps.png)

# Target Machines & Beats
This ELK server is configured to monitor the following machines:
  - Web 1 VM 10.0.0.9
  - Web 2 VM 10.0.0.8
  - Web 3 VM 10.0.0.13

Both *filebeat* and *metricbeat* have been installed on the ELK machine 10.1.0.4

**Filebeat** allows us to monitor, locate, and ship log data to a central location (Kibana) where we can analyze it with ease.
![filebeat_example.png](https://github.com/w0lfboy/Cloud-Security/blob/main/Images/filebeat%20example.png)
**Metricbeat** allows us to output metrics and statistics to the same central location (Kibana) where we can again analyze it with ease.  
![metricbeat_example.png](https://github.com/w0lfboy/Cloud-Security/blob/main/Images/metricbeat%20example.png)

# Using the Playbook
*In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned:*
`SSH` into the control node and follow the steps below:
  - Copy the roles file to `/etc/ansible/roles`
  - Update the `hosts` file to include the webserver IP's and elk IP's (see below for example `hosts` file)
    ```
    [webservers]
    10.0.0.9 ansible_python_interpreter=/usr/bin/python3
    10.0.0.8 ansible_python_interpreter=/usr/bin/python3
    10.0.0.13 ansible_python_interpreter=/usr/bin/python3

    [elk]
    10.1.0.4 ansible_python_interpreter=/usr/bin/python3
  - Run the playbook, and navigate to `http://52.251.117.56:5601/app/kibana` to check that the installation worked as expected. See below on how to run the playbook.
    ```
    ansible-playbook install-elk.yml
*It is important to note that in the `install-elk.yml` we label the hosts as `elk` which references `elk` in our `hosts` file.  This is how we differentiate which vm gets the ELK stack downloaded on it.  We don't want it on our `webservers` group, so that is why they are grouped differently.*

# ![Pen Testing the Cloud Environment](https://github.com/w0lfboy/Cloud-Security/tree/main/Pen%20Testing%20the%20Cloud%20Environment)
In this scenario, I acted as a cloud architect that has been tasked with setting up an ELK server to gather logs for the Incident Response team.
Before I hand over the server to the IR team, my senior architect has asked that I verify the ELK server is working as expected and pulling both logs and metrics from the pen-testing web servers.
I have three tasks:
  - Generate a high amount of failed SSH login attempts and verify that Kibana is picking up this activity.
  - Generate a high amount of CPU usage on the pen-testing machines and verify that Kibana picks up this data.
  - Generate a high amount of web requests to your pen-testing servers and make sure that Kibana is picking them up.

## SSH Barrage
To start, I attempted to login with an incorrect username `ssh sysadmin@10.0.0.9` multiple times.  Then, I monitored the failed attempts through the Kibana log stream as seen below.
![SSH_Barrage.png](https://github.com/w0lfboy/Cloud-Security/blob/main/Pen%20Testing%20the%20Cloud%20Environment/SSH-Barrage/SSH%20barrage.png)

Then, to make it more interesting, I barraged all three Web VMs at once.
![SSH_Barrage2.png](https://github.com/w0lfboy/Cloud-Security/blob/main/Pen%20Testing%20the%20Cloud%20Environment/SSH-Barrage/SSH%20barrage2.png)

we can use a `for` loop to make this quicker and easier.
```
$ for i in 10.0.0.9 10.0.0.8 10.0.0.13; do ssh fakelogin@$i; done
```
*Based off of our above logs, Kibana is effectively monitoring failed ssh attempts*

## Linux Stress
To start I load and attach to my ansible container.
``` 
$ sudo docker start angry_engelbart
$ sudo docker attach angry_engelbart
```
SSH into `Web-1 VM` `ssh redadmin@10.0.0.9`
Run `$ sudo apt install stress` to acquire the stress program
Run `$ sudo stress --cpu 1` 

I then moved to the Metric page in Kibana.  After waiting 2 minutes, we can see that the system has the CPU usage has increased to nearly 100% by the user as seen below.
![linux_stress.png](https://github.com/w0lfboy/Cloud-Security/blob/main/Pen%20Testing%20the%20Cloud%20Environment/Linux-Stress/linux%20stress.png)

We can do the same stress test on all three Web VM's.  See below for evidence via Kibana metrics on Web-2 and Web-3.
![linux_stress_web2](https://github.com/w0lfboy/Cloud-Security/blob/main/Pen%20Testing%20the%20Cloud%20Environment/Linux-Stress/linux%20stress%20web2.png)
![linux_stress_web3](https://github.com/w0lfboy/Cloud-Security/blob/main/Pen%20Testing%20the%20Cloud%20Environment/Linux-Stress/linux%20stress%20web3.png)

## wget-DoS
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
![wget_dos](https://github.com/w0lfboy/Cloud-Security/blob/main/Pen%20Testing%20the%20Cloud%20Environment/wget-DoS/wget%20dos.png)

And that completes our penetration test!

# Additional Commands that were used and are helpful!
| COMMAND |	PURPOSE |
|---------|---------|
| sudo apt-get update | this will update all packages |
| sudo apt install docker.io	| install docker application |
| sudo service docker start	| start the docker application |
| systemctl status docker	| status of the docker application |
| sudo docker pull cyberxsecurity/ansible	| download the docker file |
| sudo docker run -ti cyberxsecurity/ansible bash	| run and create a docker image |
| sudo docker start <image-name>	| starts the image specified |
| sudo docker ps -a	| list all active/inactive containers |
| sudo docker attach <image-name>	| effectively sshing into the ansible |
| ssh-keygen	| create a ssh key |
| ansible -m ping all	| check the connection of ansible containers |
