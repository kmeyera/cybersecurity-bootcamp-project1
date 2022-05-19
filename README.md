## Automated ELK Stack Deployment

This repository describes a Columbia Engineering Cybersecurity Bootcamp project in which virtual networks were designed and deployed to Azure, and then provisioned with an Elasticsearch Logstash Kibana (ELK) Stack, and with Damn Vulnerable Web Application (DVWA) software. Configuration and playbook files used to implement the deployment are included in this repo.

This REAMDME file contains the following details:
- Azure Cloud Resource Topology
- Azure Cloud Access Policies
- ELK Configuration
- Target Machines & Beats
- Ansible Installation and Deployment

### Azure Cloud Resource Topology

Two virtual networks were constructed for this project. The first is a network with a Jump Box, a flexible number of Web Servers running DVWA, a Load Balancer and basic firewalls implemented through Azure Network Security Rules. This network has a simple but standard architecture which limits SSH access to a single machine (the Jump Box), and provides a secure, robust and scalable presentation of web pages. 

The second network consists of a single, firewalled Application Server running an ELK Stack which is used to monitor Web Server activity. Elasticsearch is the heart of the ELK Stack and is a search and analytics engine which receives Web Server data from ELK Stack components Logstash and Beats, and allows querying and visualization of the harvested data via the browser based Kibana application.    

The configuration details of each of the four Virtual Machines (VMs) as as shown here:

| Name         | Function  | IP Address | Operating System |
|--------------|-----------|------------|------------------|
| Jump Box     | Gateway   | 10.1.0.4   | Ubuntu 18.04     |
| Web Server 1 | Runs DVWA | 10.1.0.6   | Ubuntu 20.04     |
| Web Server 2 | Runs DVWA | 10.1.0.7   | Ubuntu 20.04     |
| ELK Stack    | Monitors  | 10.0.0.4   | Ubuntu 18.04     | 

### Azure Cloud Access Policies

The Web Servers are not exposed to the public Internet except through the Load Balancer. For this exercise, Azure has been configured so that the Web Servers can only be accessed from a Home PC. Normally, Web Servers such as these would be accessible from all but blacklisted IP addresses.

The ELK Stack Kibana front end has also been configured to be accessible only from a Home PC. A more typical configuration would allow anyone within a corporate network to access Kibana.

The Jump Box is used for administration and can only be accessed from a Home PC in this implementation. Normally it would be accessible within a corporate network by system administrators. The Jump Box has SSH access to all other VMs.

A summary of the access policies in place can be found in the table below.

| Name         | Has a public IP addr?       | Access Method | Port |
|--------------|-----------------------------|---------------|------|
| Jump Box     | Yes                         | Terminal      | 22   |
| Web Server 1 | Uses Load Balancer's IP addr| Browser       | 80   |
| Web Server 2 | Uses Load Balancer's IP addr| Browser       | 80   | 
| ELK Stack    | Yes                         | Browser       | 5601 | 

### ELK Configuration

Ansible is provisioning software which was used for this project to deploy software onto the Web Server and ELK Stack VMs. It uses configuration files to specify the IP addresses and ports, and the commands to run on the VMs to get them up and running the software of choice. Use of configuration files for automated playback is advantageous because the files can be debugged, versioned and reused. 

The Ansible playbook files used for ELK configuration implements the following tasks:
- Install docker.io and python3-pip using apt-get
- Install docker using pip3
- Increase memory in the ELK VM in order to accommodate the ELK Stack
- Download and launch a docker ELK container, specifying the three ELK Stack ports (5601, 9200 and 5044)
- Enable the docker service with systemd

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

### Target Machines & Beats
This ELK server is configured to monitor the two Web Servers in the Web Server pool, Web-1 (10.1.0.6) and Web-2 (10.1.0.7). ELK software Elasticsearch, Logstash and Kibana run on the ELK server itself.

While not in the ELK acronym, Beats are a commonly used fourth component of the ELK Stack. Beats are lightweight agents which run on the monitored servers and ship specified data to the ELK Server. Two Beats were installed for this project, Filebeat and Metricbeat.

In the Filebeat configuration, Filebeat collects all logs under /var/log, data which could be queried from within Kibana in order to detect anomalous log activity. In the Metricbeat configuration, CPU usage, Load, Memory usage and Network traffic are collected. 

The screenshot below shows a Kibana visualization of Metricbeat data collected from Web-2 while attacking it with a barrage of failed SSH login attempts and a DoS web attack, and by overloading its CPU with the Linux stress command. 

### Ansible Installation and Deployment
Steps to installing, running and entering into an Ansible container on the Jump Box are as follows:
- sudo apt install docker.io
- sudo systemctl start docker
- sudo docker pull cyberxsecurity/ansible
- sudo docker run -ti cyberxsecurity/ansible:latest bash (this is for the first time you run it)
- sudo docker start <container_name> (this is for every subsequent time)
- sudo docker attach <container_name>

Once inside the Ansible container, take these steps to configure Ansible to run ELK software on a specified server
- cd /etc/ansible
- modify the hosts file to include these lines:
  - [elk]
  - 10.0.0.4 ansible_python_interpreter=/usr/bin/python3
- ansible-playbook install-elk.yml

Inside the Ansible container, take these steps to install beats on the the web servers
- modify the hosts file to include these lines:
  - [webservers]
  - 10.1.0.6 ansible_python_interpreter=/usr/bin/python3
  - 10.1.0.7 ansible_python_interpreter=/usr/bin/python3
- cd roles
- ansible-playbook filebeat-playbook.yml
- ansible-playbook metricbeat-playbook.yml

Verify that the playbooks executed as expected by navigating to http://<public_ip_addr_of_ELK_Stack_Server>:5601/app/kibana
