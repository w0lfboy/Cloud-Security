# Cloud-Security
## This document contains the following details:
  - Description of the Topology
  - Access Policies
  - ELK Configuration
    - Beats in Use
    - Machines Being Monitored
  - How to Use the Ansible Build





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
![docker_ps.png](https://github.com/w0lfboy/Cloud-Security/blob/main/docker%20ps.png)
