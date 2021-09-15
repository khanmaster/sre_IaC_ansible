# Infrastructure as Code (Iac)
## Iac - Configuration Management and Orchestration
### Which tools are used for Push config and push config

- What is Ansible and benefits of it?
- Why should we use Ansible
- Create a Diagram for Ansible on prem, Hybrid and public arthitecture
- Step by STep guide for Installating and settting up Ansible controller with 2 agent nodes - include commands in your readme.md
- What is the default Directory structure for Ansible
- What is the Inventory/hosts file and the purpose of it
- What should be added to hosts file to establish secure connection between Ansible controller and agent nodes?(include code block)
- What are Ansible `Ad-hoc commands`
- add a structure of creating adhoc commands `ansible all -m ping`
- Inlcude all the adhoc commands we have used today in this documentation


### Vagrantfile code 
```vagrant

# -*- mode: ruby -*-
 # vi: set ft=ruby :
 
 # All Vagrant configuration is done below. The "2" in Vagrant.configure
 # configures the configuration version (we support older styles for
 # backwards compatibility). Please don't change it unless you know what
 
 # MULTI SERVER/VMs environment 
 #
 Vagrant.configure("2") do |config|
 # creating are Ansible controller
   config.vm.define "controller" do |controller|
     
    controller.vm.box = "bento/ubuntu-18.04"
    
    controller.vm.hostname = 'controller'
    
    controller.vm.network :private_network, ip: "192.168.33.12"
    
    # config.hostsupdater.aliases = ["development.controller"] 
    
   end 
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
     
     #config.hostsupdater.aliases = ["development.db"]     
   end
 
 
 end
 ```
 - `vagrant up`

### Ansible Playbooks
- Ansible playbooks are completely different way to use ansible than ad-hoc task execuetion 
- Ansible playbooks are a .yml files written in YAML
- Yet Another Markup Language (YMAL)

- playbooks start with `---` 3 dashes
```
  # Create a playbook to install nginx web server on web machine
# web 192.168.33.10
# Let's add the 3 dashes to start the YAML 
---
# add the name of the host
- hosts: web

# gather facts about the installation steps 
  gather_facts: yes

# we need admin access
  become: true 
    
# add instruction to install nignx on web machine
  task:
  - name: Install Nginx
    apt: pkg=nginx state=present
# ensure the nginx server is running
```

- Let's check if the playbook worked for us
- `ansible web -a "sudo systemctl status nginx"`
- enter the web ip in your browser and you should nginx default page!
 
 ### Moving onto Configuring Mongodb in our db machine
 ```
 #mongodb.yml
# This is a YAML file to install nginx onto oue web VM using YAML
---

---

- hosts: awsdb

  gather_facts: yes

  become: true

  tasks:
  - name: install mongodb
    apt: pkg=mongodb state=present

  - name: Remove mongodb file (delete file)
    file:
      path: /etc/mongod.conf
      state: absent

  - name: Touch a file, using symbolic modes to set the permissions (equivalent to 0644)
    file:
      path: /etc/mongod.conf
      state: touch
      mode: u=rw,g=r,o=r


  - name: Insert multiple lines and Backup
    blockinfile:
      path: /etc/mongod.conf
      backup: yes
      block: |
        "storage:
          dbPath: /var/lib/mongodb
          journal:
            enabled: true
        systemLog:
          destination: file
          logAppend: true
          path: /var/log/mongodb/mongod.log
        net:
          port: 27017
          bindIp: 0.0.0.0"


```
  
### Let's get node configuration completed
```
# Install node js and NPM

- hosts: web
  gather_facts: true
  become: true



  tasks:
  - name: Install nodejs
    apt: pkg=nodejs state=present

  - name: Install NPM
    apt: pkg=npm state=present

  - name: download latest npm + Mongoose
    shell: |
      npm install -g npm@latest
      npm install mongoose -y
# Downloading pm2
  - name: Install pm2
    npm:
      name: pm2
      global: yes


  - name: seed + run app
    shell: |
      cd app/
      npm install
  #node seeds/seed.js
      pm2 kill
      pm2 start app.js
#     environment:
# # This is where you enter the environment variable to tell the app where to look for the db
#       DB_HOST: mongodb://ubuntu@<ENTER DB IP HERE>:27017/posts?authSource=admin
   become_user: root
   ```
- Ensure to set up env var before seeding 

### Importing playbooks 
```
---
# Run Mongodb Playbook
- name: Running MongoDB Playbook
  import_playbook: mongodb.yml

# Run Nginx Playbook
- name: Running Nginx Playbook
  import_playbook: nginx_proxy.yml
```
- Diagram to be added for Hybrid infrastructure here

### Let's move onto Hybrid 
- We need to install dependencies to set up Ansible Vault to secure our AWS access and secret keys
- Let's install the them using the script 
```
!#/bin/bash
sudo apt update -y
sudo apt-get install tree -y
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt install ansible -y
sudo apt install python3-pip

pip3 install awscli 
pip3 install boto boto3 -y
sudo apt-get upgrade -y
```  
- Checking installation
- `aws --version`
- Outcome should be as `aws-cli/1.20.40 Python/3.6.9 Linux/4.15.0-151-generic botocore/1.21.40`

- Let's change python to use python3 
- `alias python=python3`

#### Let's create Ansible vault file to secure our AWS keys
- In `/etc/ansible` create a folder `mkdir /group_vars/all` 
- create `pass.yml` with `ansible-vault create pass.yml`
- `aws_access_key` and `aws_secret_key` copy your keys
- `ansible db -m ping --ask-vault-pass`
- `sudo chmod 666 pass.yml`
