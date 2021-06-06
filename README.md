# cybersecurity_bootcamp_assignments
Azure Virtual Networks and Bash Script Assignments
## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

!https://drive.google.com/file/d/1y453JHMueYdbgc4fw260zyvYudiYt6xi/view?usp=sharing

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the _____ file may be used to install only certain pieces of it, such as Filebeat.

  ---
 - name: ELK Provisioner
   hosts: ELK
   become: True
   tasks:

     - name: Set VM Map Count
       sysctl:
         name: vm.max_map_count
         value: '262144'
         sysctl_set: yes

     - name: Install Docker.io
       apt:
         update_cache: yes
         name: docker.io
         state: present

     - name: Install pip3
       apt:
         force_apt_get: yes
         name: python3-pip
         state: present

     - name: Install Python Docker Module
       pip:
         name: docker
         state: present

     - name: Download and Launch Docker-ELK Container
       docker_container:
         name: elk
         image: sebp/elk:761
         state: started
         restart_policy: always
         published_ports: 5601:5601, 9200:9200, 5044:5044

     - name: Enable Docker Service
       systemd:
          name: docker
          enabled: yes


This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting access to the network.
- The load balancer, when paired with multiple instances of the DVWA (in this case 3 Web-VM's), improves availability.  The load balancer 
exposes one static-public IP address for the DVWA application/Web-VM's and it distributes traffic between the three DVWA Web-VM's as needed. If a 
health probe determines that one or more Web-VM is unavailable, then the load balancer switches traffic to an available Web-VM.  
- The Jump Box Provisioner distributes and, as needed, redistributes a Docker Container for the DVWA application to the three Web-VM's.  The Jump
Box Provisioner utilitzes Ansible playbooks to accomplish the installation and VM and network configurations.  Although the benefits of this
Ansible-Docker strategy is not so very apparent with only three subject VM's, the system is scalable to allow for the automated deployment of an 
unlimited number of additional Web-VM's. 

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the logs and system services.
- Filebeat aggregates log data on the 3 Web-VM's and sends this data to the ELK server.  Once aggregated in the ELK server, the log data can be
stored and analyzed with the Logstash and Elasticsearch tools.
- Metricbeat aggregates and exports system service data in the Web-VM's.  This data also gets delivered to the ELK server for storage and analysis.

The configuration details of each virtual machine may be found below.

| Name               | Function                                                   | IP Address                       | Operating System |
|--------------------|------------------------------------------------------------|----------------------------------|------------------|
| JumpBoxProvisioner | Provisioner/Gateway                                        | Pub: 13.68.138.134 Prv: 10.0.0.4 | Ubuntu 18.04-LTS |
| Web-1              | DVWA Server                                                | Prv: 10.0.0.5                    | Ubuntu 18.04-LTS |
| Web-2              | DVWA Server                                                | Prv: 10.0.0.6                    | Ubuntu 18.04-LTS |
| Web-3              | DVWA Server                                                | Prv: 10.0.0.7                    | Ubuntu 18.04-LTS |
| ELK                | ELK Server                                                 | Pub: 20.96.32.184 Prv: 10.1.0.4  | Ubuntu 18.04-LTS |
| LoadBalancer_1     | HTTP Traffic Switching and Single Pub IP Add. For Web-VM's | Pub: 52.147.194.8                |                  |

### Access Policies

Access Policies are mostly determined in the Azure Portal through Network Security Groups.

There are two network security groups, Jumpboxprovisoner-nsg and ELK-nsg.  The Jumpboxprovisioner-nsg includes the Jumpboxprovisioner VM and 3 Web-VM's.
The ELK-nsg includes the ELK server VM only.  Azure Network Security Group rules are divided between Inbound and Outbound sets.  Each rule, inbound or outbound
is designated as either a Deny rule or an Allow rule.  Furthermore, each rule is assigned a semi-arbitrary number.  Lower numbered rules take precedence
over higher numbered rules.  

The Jumpboxprovisioner-nsg's highest numbered (therefore, lowest priority) rule disallows all inbound traffic.  This rule exists by default in the NSG. 
Therefore, subsequent lower numbered rules are created by the administrator to permit desired traffic.  For our Jumpboxprovisioner-nsg, a rule was added
to permit SSH connections from the Jumpboxprovisioner VM to any other VM on the Virtual Network.  This rule is necessary to allow the Jumpboxprovisioner VM's
Ansible application to do its job of provisioning the DVWA application on the Web-VM's.  Ansible operates over port 22.  Another rule was added to allow
SSH connections specifically from the administrator's remote IP address to the Jumpboxprovisioner VM.  This is necessary to adminstrate the provisioner VM.  
This rule does not allow access to any other remote IP addresses.

The single load balancer on the project, named LoadBalancer_1 provides IP masquerading for the 3 Web-VM's, which together make up the load balancer's 
"backend pool."  LoadBalancer_1 has a static public IP address.  The load balancer only allows HTTP traffic between itself and the three Web-VM's.
The Web-VM's are further isolated by Jumpboxprovisioner-nsg rules.  Other than the load balancer, only the Jumpboxprovisioner VM and the ELK server
permit connections fromt eh internet.  And these outside connections are restricted to whitelisted IP addresses.


### Elk Configuration

Ansible was valuable for the deployment of the Web-Server VM's because these vm's may need to be reset.  Furthermore, Ansible grants scalability, 
allowing us to add additional Web-Server VM's after adding IP addresses to the configuration file and placing the public ssh key in a new server vm.
On the other hand, the value of utilizing Ansible for the ELK server deployment and configuration is only the value of a further exercise in using 
Ansible.  It would have taken less work to set up the ELK server on the ELK-Server VM than to configure the deployment on the Provisioner.   

The steps in the Ansible Playbook for deploying the ELK-Server include the following:
- The first step is to increase the virtual memory allowance of the ELK-Server VM.
- Install docker.io via apt.
- Install pip via apt-get.
- Install the python docker-module via pip.
- Download and launch the Docker-ELK container with port configurations.
- Configure systemd to enable the Docker service on vm startup.

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

https://drive.google.com/file/d/1czfiDzOmMkDzwYLTT2a3w2nHl8CrcUQR/view?usp=sharing

### Target Machines & Beats
The ELK server is configured to monitor the three Web-VM's.  Filebeat and Metricbeat are installed on each Web-VM.  Filebeat aggregates
log data and sends it to the ELK server for storage and analysis.  Metricbeat sends the ELK server monitoring data regarding the operating system
and services running on each Web-VM.

The default Filebeat settings record the following logs on the three Web-VM's: Syslog, Sudo Commands, SSH Logins, and New Users and Groups.


The default Metricbeat dashboard that we installed provides metrics on CPU usage, Disk IO, Memory Usage, and Network IO.


### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
The installations and checks worked after some tweaks.  

_TODO: Answer the following questions to fill in the blanks:_
- _Which file is the playbook? Where do you copy it?_ I have multiple playbooks as per the instructions.  I named them ELK_provisioner.yml, docker_pip_dvwa_install.yml,
metricbeat_install.yml, and filebeat-playbook_provided.yml.  They are stored in the Ansible Container in my Provisioner VM in a directory called "playbooks", which is 
located in the home directory of the root user.
- _Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on?_
In the /etc/ansible directory the "hosts" file resides.  In the hosts file IP addresses of target machines are added.  The IP addresses are grouped together with a square-bracketed
group name (i.e. [webservers] and [ELK] in this case.)  The playbooks designate IP addresses according to their group names. So, all three Web-VM's are targeted by a playbook
by referencing the "webservers" group.
- I navigate to the ELK server's public IP address and port 5601 from my home workstation to verify that it is running.  Then I go to the Filebeat and Metricbeat dashboards
to verify that they are receiving data from the Web-VM's.
