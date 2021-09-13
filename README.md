# Infrastructure as code (Iac)
## IAC - configuration management and orchestration
### WHich tools are used for push and pull config

- what is ansible and benefits of it?
- why should we use it
- Create a diagram for ansible on prem, Hybird and public architecture
- step by step guide Installation and setting up ansible controller with 2 agent nodes - include commands in your readme
- what is the default directory structure for ansible
- what is the inventory/hosts file and the purpose of it
- What should be added to hosts file to establish secure connection between Ansible controller and agent nodes (include code block)
- What are Ansible Ad-hoc commands
- add a structure of creating ad hoc commands 'Ansible all -m ping' - basically what is each thing written in the code
- include all the ad-hoc commands weve used today in this documentation (in notepad)



# Ansible controller and agent nodes set up guide
- Clone this repo and run `vagrant up`
- `(double check syntax/intendation)`


## We will use 18.04 ubuntu for ansible controller and agent nodes set up 
### Please ensure to refer back to your vagrant documentation

- **You may need to reinstall plugins or dependencies required depending on the OS you are using.**

### vagrant file codes

```vagrant 
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
    
    config.hostsupdater.aliases = ["development.web"]
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
    
    config.hostsupdater.aliases = ["development.controller"] 
    
  end

end
```

- 'vagrant up'

### Ansible playbooks
- Ansible playbooks are a completely different way to use ansible than ad-hoc task execution
- ansible playbooks are a .yml files written in YAML
- Yet Another Markup Language (YAML)

- playbooks start with '---' 3 dashes
- 


