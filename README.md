# Ansible

## What is Ansible?
Ansible is an open-source software provisioning, configuration management (CM) and application deployment tool enabling infrastructure as code. Ansible is AGENTLESS, temporary connecting remotely via SSH or Windows Remote Management to do its tasks.

### Use case
When we launch an EC2 instance on AWS, for example, we need to click a lot of things in order to set the instance up. By using Ansible and IaC we could have a script of instruction to do these steps -> Automation. 
Note: Ansible can be used for on premise, hybrid or cloud configurations

### Benefits
- Open-source (free, a lot of documentation available)
- Efficient
- Powerful
- Agentless (no need to install other software or separate management structure, agent nodes do not need Ansible installed on them)
- Simple to set up

## What is Infrastructure as Code (IaC)?
IaC manages and provisions the infrastructure through code instead of manual processes. This makes it easier to edit and distribute configurations.
IaC has two parts:
1. Configuration Management (Ansible, Puppet, CHEF): track and control the changes in a software
2. Orchestration (Ansible, Terraform, Kubernetes): same but automated

### Push-Pull configuration
The tools that work with IaC have either push or pull configuration. The main difference is the manner in which the servers are told how to be configured. In the pull method the server to be configured will pull its configuration from the controlling server. In the push method the controlling server pushes the configuration to the destination system.

![alt text](https://github.com/ioanan11/sre_iac_ansible/blob/main/Screenshot%202021-09-14%20082531.png)

### Yet Another Markup Language (YAML)
YAML is a data serialization language that is often used for writing configuration files. Depending on whom you ask, YAML stands for yet another markup language or YAML ain’t markup language (a recursive acronym), which emphasizes that YAML is for data, not documents. 

YAML is a popular programming language because it is human-readable and easy to understand. It can also be used in conjunction with other programming languages.


### Playbooks
Ansible playbooks are essentially frameworks, which are prewritten code developers can use ad-hoc or as starting template. 

Ansible playbooks help IT staff program applications, services, server nodes, or other devices without the manual overhead of creating everything from scratch. And Ansible playbooks—as well as the conditions, variables, and tasks within them—can be saved, shared, or reused indefinitely.

Note: Playbooks start with 3 dashes
	---

### Ad hoc commands
Used to find out stuff about the agent machines. They are designed to achieve a very specific task very quickly. 
Command structure:

ansible + all or "machine name" + -m "module name" + "CLI command"

Example:
' ansible all -a "uname -a" ' To find the systems' information 


**To put it simply, ad hoc commands are one line in a Linux shell and playbooks are the shell script.**

# How to use Ansible on prem?

![alt text](https://github.com/ioanan11/sre_iac_ansible/blob/main/Screenshot%202021-09-14%20075300.png)

## Step 1: Vagrantfile Configuration
We need to configure 3 different Vagrant Machines. In our example, we will use Web (which will contain node app), db (database running on MongoDB) and Controller (Ansible controller). 
This repo containing the Vagrantfile on GitHub was cloned: https://github.com/khanmaster/eng89_ansible_controller
The Vagrantfile should look like this:

	# -*- mode: ruby -*-
	# vi: set ft=ruby :

	# All Vagrant configuration is done below. The "2" in Vagrant.configure
	# configures the configuration version (we support older styles for
	# backwards compatibility). Please don't change it unless you know what

	# MULTI SERVER/VMs environment 
	#	
	Vagrant.configure("2") do |config|

	# creating first VM called web  
  	config.vm.define "web" do |web|
    
    	web.vm.box = "bento/ubuntu-18.04"
   	# downloading ubuntu 18.04 image

    	web.vm.hostname = 'web'
    	# assigning host name to the VM
    
    	web.vm.network :private_network, ip: "192.168.33.10"
    	#   assigning private IP
    
    	#config.hostsupdater.aliases = ["development.web"]
    	# creating a link called development.web so we can access web page with this link instread of an IP   
        
      end
  
	# creating second VM called db
 	 config.vm.define "db" do |db|
    
    	db.vm.box = "bento/ubuntu-18.04"
    
    	db.vm.hostname = 'db'
    
   	db.vm.network :private_network, ip: "192.168.33.11"
    
    	config.hostsupdater.aliases = ["development.db"]     
  	end

 	# creating are Ansible controller
 	config.vm.define "controller" do |controller|
    
    	controller.vm.box = "bento/ubuntu-18.04"
    
    	controller.vm.hostname = 'controller'
    
    	controller.vm.network :private_network, ip: "192.168.33.12"
    
    	#config.hostsupdater.aliases = ["development.controller"] 
    
      end

    end

## Step 2: Vagrant up
Vagrant up and SSH into each machine at a time

	vagrant ssh web
	vagrant ssh db
	vagrant ssh controller

Run the following commands:

	sudo apt-get update -y
	sudo apt-get upgrade

## Step 3: Install Ansible 
Stay in the cotroller machine and follow these commands: 

	sudo apt-get install software-properties-common -y
	sudo apt-add-repository ppa:ansible/ansible
	sudo apt-get install ansible -y

## Step 4: SSH into agent machines
Use the command: 

	ssh vagrant@"machine ip"

In our case the IPs are:

- web: 192.168.33.10 
- db:  192.168.33.11

## Step 5: Defining the machines

In ansible configuration directory we need to edit the hosts file by adding our IPs 

	cd /etc/ansible/
	ansible all -m ping

- for web: 192.168.33.10 ansible_connection=ssh ansible_user=vagrant ansible_ssh_pass=vagrant
- for db: 192.168.33.11 ansible_connection=ssh ansible_user=vagrant ansible_ssh_pass=vagrant

Run ' ansible all -m ping ' again.

## Installing nginx using a playbook in Ansible

- create the nginx playbook 
	
	sudo nano nginx_playbook.yml

- it needs to contain:

	#Create a playbook to install nginx web server on web machine
	#web 192.168.33.10
	#Lets add 3 dashes to start the YAML
	---
	#add the name of the host
	- hosts: web

	#gather facts about the installation steps
	gather_facts: yes

	#we need admin access 
	become: true

	#add instructions to install nginx on web machine
	tasks:
	- name: Install Nginx
	  apt: pkg=nginx state=present


- Run the playbook

	ansible-playbook nginx_playbook.yml


# How to use Ansible on hybrid configuration?
## Launching an EC2 instance on AWS using Ansible playbook

### Step 1: Install the following dependencies

- Python 3 and above
- Boto3
- pip3

Run these commands, either one by one or from a script:

	
	sudo apt install python
        alias python=python3
	sudo apt install python-pip
	sudo pip install --upgrade pip
	sudo pip install boto boto3

### Step 2: set up Ansible Vault


### Step 3: make sure all keys are available
Copy the pem file into Ansible controller to SSH into the EC2 instance


We need to install some dependencies:

- AWS keys
- sre_key.pem
- sre_key and sre_key.pub
- ansible vault configuration
- an aws account
- iam role with access to EC2

Step 4: Playbook

```
---

- hosts: localhost
  connection: local
  gather_facts: True
  become: True
  vars:
   key_name: sre_key
   region: eu-west-1
   image: ami-0943382e114f188e8
   id: SRE ioana Ansible EC2
   sec_group: sg-0373f88dbd1c68d3e
   subnet_id: subnet-0429d69d55dfad9d2

   ansible_python_interpreter: /usr/bin/python3

  tasks:
   - name: Facts
     block:

     - name: Get instance facts
       ec2_instance_facts:
         aws_access_key: "{{aws_access_key}}"
         aws_secret_key: "{{aws_secret_key}}"
         region: "{{ region }}"
       register: result

   - name: Provisioning EC2 instances
     block:

     - name: Upload public key to AWS
       ec2_key:
         name: "{{key_name}}"
         key_material: "{{lookup('file', '~/.ssh/{{key_name}}.pub') }}"
         region: "{{ region }}"
         aws_access_key: "{{aws_access_key}}"
         aws_secret_key: "{{aws_secret_key}}"

     - name: Provision instance(s)
       ec2:
         aws_access_key: "{{aws_access_key}}"
         aws_secret_key: "{{aws_secret_key}}"
         assign_public_ip: true
         key_name: "{{key_name}}"
         id: "{{ id }}"
         vpc_subnet_id: "{{ subnet_id }}"
         group_id: "{{ sec_group }}"
         image: "{{ image }}"
         instance_type: t2.micro
         region: "{{ region }}"
         wait: true
         count: 1
         instance_tags:
           Name: sre_ioana_ansible_ec2

      tags: ['never', 'create_ec2']

```



Create an EC2 instance in Ansible controller

- ami
- type of instance: t2micro
- vpc id
- subnet id
- SG id
- key_name
- public ip


