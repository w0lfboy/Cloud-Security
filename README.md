# Cloud-Security


Automated ELK Stack Deployment
The files in this repository were used to configure the network depicted below.

Cloud_Security_ELK.png


These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of playbooks may be used to install only certain pieces of it, such as Filebeat, Metricbeat, etc.
Metricbeat Playbook
---
  - name: Install metric beat
    hosts: webservers
    become: true
    tasks:
      # Use command module
    - name: Download metricbeat
      command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.6.1-amd64.deb


      # Use command module
    - name: install metricbeat
      command: dpkg -i metricbeat-7.6.1-amd64.deb


      # Use copy module
    - name: drop in metricbeat config
      copy:
        src: /etc/ansible/files/metricbeat-config.yml
        dest: /etc/metricbeat/metricbeat.yml


      # Use command module
    - name: enable and configure docker module for metric beat
      command: sudo metricbeat modules enable kibana


      # Use command module
    - name: setup metric beat
      command: sudo metricbeat setup


      # Use command module
    - name: start metric beat
      command: sudo service metricbeat start


      # Use systemd module
    - name: enable service metricbeat on boot
      systemd:
        name: metricbeat
        enabled: yes


Filebeat Playbook
---
  - name: installing and launching filebeat
    hosts: webservers
    become: yes
    tasks:
    - name: download filebeat deb
      command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb


    - name: install filebeat deb
      command: dpkg -i filebeat-7.6.1-amd64.deb


    - name: drop in filebeat.yml
      copy:
        src: /etc/ansible/files/filebeat-config.yml
        dest: /etc/filebeat/filebeat.yml


    - name: enable and configure system module
      command: sudo filebeat modules enable system


    - name: setup filebeat
      command: sudo filebeat setup


    - name: start filebeat service
      command: sudo service filebeat start


    - name: enable service filebeat on boot
      systemd:
        name: filebeat
        enabled: yes
Install-Elk Playbook
---
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: redadmin
  become: true
  tasks:
    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        force_apt_get: yes
        name: docker.io
        state: present


      # Use apt module
    - name: Install python3-pip
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present


      # Use pip module (It will default to pip3)
    - name: Install Docker module
      pip:
        name: docker
        state: present


      # Use command module
    - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144


      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: '262144'
        state: present
        reload: yes


      # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        # Please list the ports that ELK runs on
        published_ports:
          -  5601:5601
          -  9200:9200
          -  5044:5044


      # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes
DVWA Playbook
---
- name: DVWA Playbook
  hosts: webservers
  become: true
  tasks:
  - name: Install docker.io  (state=present is optional)
    apt:
       update_cache: yes
       name: docker.io
       state: present


  - name: Install python3-pip  (state=present is optional)
    apt:
       name: python3-pip
       state: present
  - name: Install docker  (state=present is optional)
    pip:
       name: docker
       state: present


  - name: Install cyberxsecurity/dvwa  (state=present is optional)
    docker_container:
       name: dvwa
       image: cyberxsecurity/dvwa
       state: started
       restart_policy: always
       published_ports: 80:80


This document contains the following details:
  - Description of the Topology
  - Access Policies
  - ELK Configuration
    - Beats in Use
    -  Machines Being Monitored
  - How to Use the Ansible Build
  - Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.
Load balancing ensures that the application will be highly available, in addition to restricting an overload of traffic to the network.
Our load balancer allows traffic to redirect between the three machines hosting the site to provide availability and security as needed to our site.  This is a crucial part of the CIA triad that can be done simply with the load balancer.
With our jump box, we can provide scaling and segmentation with ease.  Since all of our playbooks are located in the jump-box, we can easily create new machines to run whichever playbook we prefer.  By having one central location such as our jump box, we are also minimizing access controls.  
Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the data and system logs.
Why install Filebeat and Metricbeat?
Filebeat allows us to monitor, locate, and ship log data to a central location (Kibana) where we can analyze it with ease.
Metricbeat allows us to output metrics and statistics to the same central location (Kibana) where we can again analyze it with ease.  
By combining both of these beats we can monitor security to prevent and respond to security incidents.
Automated ELK Stack Deployment
The files in this repository were used to configure the network depicted below.


