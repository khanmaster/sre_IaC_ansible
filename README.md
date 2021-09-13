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
 
  




