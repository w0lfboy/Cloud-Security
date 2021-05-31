# Cloud-Security
## This document contains the following details:
  - Description of the Topology
  - Access Policies
  - ELK Configuration
    - Beats in Use
    - Machines Being Monitored
  - How to Use the Ansible Build

## Cloud Network Diagram
The files in this repository were used to configure the network depicted below.

![Cloud_Security_ELK.png](https://github.com/w0lfboy/Cloud-Security/blob/main/Diagrams/Cloud%20Security%20ELK.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of playbooks may be used to install only certain pieces of it, such as Filebeat, Metricbeat, ELK installation, and DVWA playbooks.
  - [Metricbeat Playbook](https://github.com/w0lfboy/Cloud-Security/blob/main/Ansible/Metricbeat-Playbook.yml)
  - [Filebeat Playbook](https://github.com/w0lfboy/Cloud-Security/blob/main/Ansible/Filebeat-Playbook.yml)
  - [Install-Elk Playbook](https://github.com/w0lfboy/Cloud-Security/blob/main/Ansible/Install-Elk.yml)
  - [DVWA Playbook](https://github.com/w0lfboy/Cloud-Security/blob/main/Ansible/DVWA-Playbook.yml)

# Description of the Topology
The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting an overload of traffic to the network.

Our load balancer allows traffic to redirect between the three machines hosting the site to provide availability and security as needed to our site.  This is a crucial part of the CIA triad that can be done simply with the load balancer.

With our jump box, we can provide scaling and segmentation with ease.  Since all of our playbooks are located in the jump-box, we can easily create new machines to run whichever playbook we prefer.  By having one central location such as our jump box, we are also minimizing access controls. 

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the data and system logs.

Why install Filebeat and Metricbeat?

Filebeat allows us to monitor, locate, and ship log data to a central location (Kibana) where we can analyze it with ease.
Metricbeat allows us to output metrics and statistics to the same central location (Kibana) where we can again analyze it with ease.  

By combining both of these beats we can monitor security to prevent and respond to security incidents.
The configuration details of each machine may be found below. 

|         Name         |     Function    | IP Address (private) | IP Address (public) |   OS  |
|:--------------------:|:---------------:|:--------------------:|---------------------|:-----:|
|       Jump Box       |     Gateway     |       10.0.0.4       |    20.185.199.138   | Linux |
|         Web-1        |      Server     |       10.0.0.9       |     40.121.94.31    | Linux |
|         Web-2        |      Server     |       10.0.0.8       |     40.121.94.31    | Linux |
|         Web-3        |      Server     |       10.0.0.13      |     40.121.94.31    | Linux |
| Django VM  Elk Stack | Monitor servers |       10.1.0.4       |    52.251.117.56    | Linux |

# Access Policies
The machines on the internal network are not exposed to the public Internet.
Only the Jump Box Provisioner can accept connections from the Internet. Access to this machine is only allowed from the IP address 71.115.3.39

Machines within the network can only be accessed by the Jump Box Provisioner.

The only machine allowed to access the ELK stack server is the Jump Box Provisioner (10.0.0.4).

A summary of the access policies in place can be found in the table below.
|         Name        | Publicly Accessible |                    Allowed IP Addresses                   |
|:-------------------:|:-------------------:|:---------------------------------------------------------:|
|       Jump Box      |          No         |                 71.115.3.39 on SSH port 22                |
|        Web-1        |          No         | 10.0.0.4 on SSH port 22 and 71.115.3.39 via Load Balancer |
|        Web-2        |          No         | 10.0.0.4 on SSH port 22 and 71.115.3.39 via Load Balancer |
|        Web-3        |          No         | 10.0.0.4 on SSH port 22 and 71.115.3.39 via Load Balancer |
| Django VM ELK Stack |          No         |  10.0.0.4 on SSH port 22 and 71.115.3.39 on TCP port 5601 |
|    Load Balancer    |          No         |                71.115.3.39 on HTTP port 80                |

# ELK Configuration

