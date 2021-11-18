## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

Project 1 Diagram of Elk.PNG

![Project Diagram of Elk VM](https://user-images.githubusercontent.com/88859779/141648701-d191b487-e6e7-418c-98d4-6789db64fc82.png)


These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the _Ansible and YML_ file may be used to install only certain pieces of it, such as Filebeat.

<p>
<details>
  <summary>ELK Installation Playbook</summary>
  
<pre><code>---
- name: Configure ELK
  hosts: ELK
  remote_user: sysadmin
  become: True
  tasks:
  - name: use more memory
    sysctl:
      name: vm.max_map_count
      value: '262144'
      state: present
      reload: yes

  - name: docker.io
    apt:
      update_cache: yes
      name: docker.io
      state: present

  - name: Install pip3
    apt:
     force_apt_get: yes
     name: python3-pip
     state: present

  - name: install python module
    pip:
      name: docker
      state: present

  - name: elk container
    docker_container:
      name: elk
      image: sebp/elk:761
      state: started
      restart_policy: always
      published_ports:
        - 5601:5601
        - 9200:9200
        - 5044:5044
        
  - name: Enable Service docker on boot
    systemd:
      name: docker
      enabled: yes</code></pre>

  </details>
  </p>

<p>
<details>
  <summary>VM with Docker Installation</summary>
  
  <pre><code>
  ---
- name: Config Web VM with Docker
  hosts: webservers
  become: true
  tasks:
  - name: docker.io
    apt:
      force_apt_get: yes
      update_cache: yes
      name: docker.io
      state: present

  - name: Install pip3
    apt:
      force_apt_get: yes
      name: python3-pip
      state: present

  - name: Install Docker python module
    pip:
      name: docker
      state: present

  - name: download and launch a docker web container
    docker_container:
      name: dvwa
      image: cyberxsecurity/dvwa
      state: started
      published_ports: 80:80

  - name: Enable docker service
    systemd:
      name: docker
      enabled: yes</code></pre>
  </details>
  </p>

<p>
<details>
  <summary>Filebeats Playbook</summary>
  
  <pre><code>---
- name: Installing and Launch Filebeat
  hosts: webservers
  become: yes
  tasks:
    # Use command module
  - name: Download filebeat .deb file
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

    # Use command module
  - name: Install filebeat .deb
    command: dpkg -i filebeat-7.4.0-amd64.deb

    # Use copy module
  - name: Drop in filebeat.yml
    copy:
      src: /etc/ansible/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

    # Use command module
  - name: Enable and Configure System Module
    command: filebeat modules enable system

    # Use command module
  - name: Setup filebeat
    command: filebeat setup

    # Use command module
  - name: Start filebeat service
    command: service filebeat start

    # Use systemd module
  - name: Enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes</code></pre>
  </details>
  </p>
  
<p>
<details>
  <summary>Metricbeats Playbook</summary>
  
  <pre><code>---
- name: Install Metricbeat
  hosts: webservers
  become: true
  tasks:
    # Use command module
  - name: Download metricbeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb

    # Use command module
  - name: install metricbeat
    command: dpkg -i metricbeat-7.4.0-amd64.deb

    # Use copy module
  - name: drop in metricbeat config
    copy:
      src: /etc/ansible/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

    # Use command module
  - name: enable and configure docker module for metric beat
    command: metricbeat modules enable docker

    # Use command module
  - name: setup metric beat
    command: metricbeat setup

    # Use command module
  - name: start metric beat
    command: service metricbeat start

    # Use systemd module
  - name: Enable service metricbeat on boot
    systemd:
      name: metricbeat
      enabled: yes</code></pre>
  </details>
  </p>

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.
Load balancing ensures that the application will be highly _available_, in addition to restricting _inbound access_ to the network.

What aspect of security do load balancers protect? 

_Load balancers protect the Availability, Web Traffic, and Web Security. Load balancers protects the machine from DDoS attacks by rerouting live traffic._

What is the advantage of a jump box? 

_The advantage of a jump box is that it allows automation, improves security, audtis traffic of segmented networks, and allows for ease of access control by using a single point to be secured and monitored._


Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the _logs_ and system _metrics and statistics_.

What does Filebeat watch for? 

_Filebeat monitors for any changes in information and when the changes took place._

What does Metricbeat record?

_Metricbeat records metrics and statistical data which is then sent out to a specified output: Elasticsearch or logistach for indexing._

The configuration details of each machine may be found below.

| Name          | Function       | IP Address                     | Operating System |
|---------------|----------------|--------------------------------|------------------|
| Jump Box      | Gateway        | 52.249.250.19/10.0.0.4         | Linux            |
| DVWA-VM1      | Server         | 10.0.0.5/Static IP             | Linux            |
| DVWA-VM2      | Server         | 10.0.0.6/Static IP             | Linux            |
| ELK-server    | Monitoring     | 20.112.100.232/10.1.0.4        | Linux            |
| Load Balancer | LB             | Static IP                      | Linux            |
| User          | Access Control | External IP                    | Windows 10       |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the _Jump Box Provisioner_ machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:

_The local host IP address: 52.249.250.19._

Machines within the network can only be accessed by _Jump Box Provisioner._

Which machine did you allow to access your ELK VM? What was its IP address? 

_Jump Box Provisioner Private IP: 10.0.0.4 via ssh port 22 and User Public IP via TCP port 5601._

A summary of the access policies in place can be found in the table below.

| Name                 | Publicly Accessible | Allowed IP Addresses                    |
|----------------------|---------------------|-----------------------------------------|
| Jump Box Provisioner | Yes                 | User Public IP via ssh port 22          |
| Elk-Server           | No                  | User Public IP via ssh TCP port 5601    |
| DVWA-Web1            | No                  | 10.0.0.4 via ssh port 22                |
| DVWA-Web2            | No                  | 10.0.0.4 via ssh port 22                |
| Load Balancer        | No                  | User Public IP via HTTP port 80         |   
   

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...

What is the main advantage of automating configuration with Ansible?

_- Ansible allows you to deploy multiple applications using a single playbook into your servers._

The playbook implements the following tasks:

- _Machine groups and remote user specifications:_

```
- name: Configure Elk VM with Docker
  	  hosts: elk
  	  remote_user: sysadmin
 	  become: true
  	  tasks:
```

- _Increase system memory:_
```
- name: Use more memory
      	  sysctl:
       	  name: vm.max_map_count
          value: "262144"
          state: present
          reload: yes
```

- _Install the following packages:_
	- _Docker.io_
	- _Python3-pip_
	- _Docker elk container_

- _Launch docker container with these ports:_
	- _5601:5601_
	- _9200:9200_
	- _5044:5044_

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

_Elk Server_

![Elk docker ps](https://user-images.githubusercontent.com/88859779/141648723-882227ce-22c8-4521-a094-df0c5f4fa1ad.png)

_Web-1_

![Web-1 docker ps](https://user-images.githubusercontent.com/88859779/141648732-1c7e4bc3-f227-42e1-b9d3-dd7ba5c99a59.png)

_Web-2_

![Web-2 docker ps](https://user-images.githubusercontent.com/88859779/141648736-7f19f795-81d6-49b0-93ab-47db01535420.png)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:

- _Web-1: 10.0.0.5_
- _Web-2: 10.0.0.6_


-We have installed the following Beats on these machines:

_- Elk Server, Web-1, and Web-2_

These Beats allow us to collect the following information from each machine:

- _Filebeat allows us to collect log events._ 
	- _Ex: Files generated by Apache or MS Azure tools._

- _Metricbeat allows us to collect the metrics and statistics._
	- _Ex: CPU usage._



### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:

- Copy the _Filebeat Playbook_ file to _/etc/ansible_.

![etc ansible](https://user-images.githubusercontent.com/88859779/141648775-1631eed9-c0ae-43f3-8f42-df14a638f1a1.png)

- Update the _/etc/ansible/hosts_ file to _include Web-1, Web-2, and Elk-server._

![machine specification in hosts](https://user-images.githubusercontent.com/88859779/141648782-6a2a6cc7-4c7d-4de8-b737-a484503d8f73.png)

- Run the playbook, and navigate to _the Elk VM via Kibana URL_ to check that the filebeat installation worked as expected.

![Verify Installation Playbook](https://user-images.githubusercontent.com/88859779/141648789-d77284ca-f13a-499c-ab4e-7f27ae353f6d.PNG)

_**Repeat the steps with Metricbeat using the metricbeat config.yml file and the metricbeat playbook file**._ 

- Run the playbook, and navigate to _the Elk VM via Kibana URL_ to check that the metricbeat installation worked as expected.

![Verify Metricbeat](https://user-images.githubusercontent.com/88859779/142044240-2fc83cb4-ac7b-42d4-9c93-87b96c8a8bcc.png)

Which file is the playbook? _Filebeat-playbook.yml_

Where do you copy it? _Into the_ `/etc/ansbile/files`

Which file do you update to make Ansible run the playbook on a specific machine? 

`/etc/ansible/hosts` _file and add the IP address of the VM's under [webserver]._

How do I specify which machine to install the ELK server on versus which to install Filebeat on?

_By specifying two groups in the_ `/etc/ansible/hosts` _file, labled [webservers] for filebeat, and [ELK] for Elk installation._

Which URL do you navigate to in order to check that the ELK server is running?

_**http:[Elk.VM.IP]:5601/app/kibana**_

As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files

<details><summary>Click for command list</summary>
------Filebeat------

-First ssh into your ansible container by running the following command:
	`:~$ ssh -i [name of keygen] [user@ipaddress]`

- To create the filebeat-config.yml file run the following command:

`ansiblecontainer:~$ cd /etc/ansible/files`
`/etc/ansible/files# curl https://gist.githubusercontent.com/slape/5cc350109583af6cbe577bbcc0710c93/raw/eca603b72586fbe148c11f9c87bf96a63cb25760/Filebeat > /etc/ansible/filebeat-config.yml`


- Edit the file to servers info by running the following command:
	
	`/etc/ansible/files# nano filebeat-config.yml`

		output.elasticsearch: 
		hosts: ["10.1.0.4:9200"] #<- edit to ELK IP address
		username: "elastic"
		password: "changeme"
				&
		setup.kibana:
		host: "10.1.0.4:5601" #<- edit to ELK IP address

- To create the filebeat-playbook.yml file run the following command:
	`/etc/ansible# nano filebeat-playbook.yml`

- Configure the file to download the .deb file, install the .deb file, run the filebeat modules enable system, run the filebeat setup command, run the service filebeat start command, and to enable the filebeat service on boot. The configuration of the playbook should resemble:

---
		- name: Installing and Launch Filebeat
  		  hosts: webservers
  		  become: yes
  		  tasks:
    		# Use command module
  		- name: Download filebeat .deb file
    		  command: curl -L -O 		https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

    		# Use command module
  		- name: Install filebeat .deb
    		  command: dpkg -i filebeat-7.4.0-amd64.deb

    		# Use copy module
  		- name: Drop in filebeat.yml
    		  copy:
      		src: /etc/ansible/files/filebeat-config.yml
      		dest: /etc/filebeat/filebeat.yml

    		# Use command module
 		 - name: Enable and Configure System Module
   		   command: filebeat modules enable system

   		 # Use command module
 		 - name: Setup filebeat
  		   command: filebeat setup

   		 # Use command module
 		 - name: Start filebeat service
  		   command: service filebeat start

   		 # Use systemd module
  		- name: Enable service filebeat on boot
  		  systemd:
     		    name: filebeat
    		    enabled: yes


- To run this playbook navigate to the directory the file is at and run the following command:

	`/etc/ansible# ansible-playbook filebeat-playbook.yml`


------Metricbeat------

- To create the metricbeat-config.yml file run the following command:

	`ansiblecontainer:~$ cd /etc/ansible/files`

	`/etc/ansible/files# cp filebeat-config.yml metricbeat-config.yml`
- No edits to the copied contents required for the config file.

- To create the metricbeat-playbook.yml file navigate to where the filebeat-playbook file is located and run the following command:

	`/etc/ansible# cp filebeat-playbook.yml metricbeat-playbook.yml`

	`/etc/ansible# nano filebeat-playbook.yml`

- Change all "filebeat" to "metricbeat". Change the command URL path to metric beat path. The configuration of the playbook should resemble:

		---
		- name: Installing and Launch Metricbeat
  		  hosts: webservers
  		  become: yes
  		  tasks:
    		# Use command module
  		- name: Download metricbeat .deb file
    		  command: curl -L -O 		https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.7.1-amd64.deb

    		# Use command module
  		- name: Install metricbeat .deb
    		  command: dpkg -i metricbeat-7.4.0-amd64.deb

    		# Use copy module
  		- name: Drop in metricbeat.yml
    		  copy:
      		src: /etc/ansible/files/metricbeat-config.yml
      		dest: /etc/metricbeat/metricbeat.yml

    		# Use command module
 		 - name: Enable and Configure System Module
   		   command: metricbeat modules enable system

   		 # Use command module
 		 - name: Setup metricbeat
  		   command: metricbeat setup

   		 # Use command module
 		 - name: Start metricbeat service
  		   command: service metricbeat start

   		 # Use systemd module
  		- name: Enable service metricbeat on boot
  		  systemd:
     		    name: metricbeat
    		    enabled: yes

- To run this playbook navigate to the directory the file is at and run the following command:

	`/etc/ansible# ansible-playbook metricbeat-playbook.yml`
</details>


