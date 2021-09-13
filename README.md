# Infrastructure as code (Iac)
## IAC - configuration management and orchestration
### Which tools are used for push and pull config

## What is Ansible and benefits of it?
Ansible is a software tool that provides simple but powerful automation for cross-platform computer support. It is intended for IT professionals, who use it for application deployment, updates on workstations and servers, cloud provisioning, configuration management, intra-service orchestration etc.

## why should we use it

## Create a diagram for ansible on prem, Hybird and public architecture

# Ansible controller and agent nodes set up guide
![image](https://user-images.githubusercontent.com/88186084/133131903-d4f81f87-0fe7-45bd-9dbd-82a366d66563.png)

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

- `vagrant up`

-------------------------------------------------------

### Are the VM's up to date
- `vagrant status`
- ssh into each machine `vagrant ssh "machine name"`
- `Sudo apt-get update -y` 
- `Sudo apt-get upgrade -y`
- `ping web` `ping db` from the controller machine to test connection 

----------------------------------------

## Installing Ansible

-----------------------------------

### Step 1 Install dependencies
- `sudo apt-get install software-properties-common -y`
- `sudo apt-add-repository ppa:ansible/ansible`

--------------------------------------------

### Step 2 Run update
- `sudo apt-get update -y`

-------------------------------------------------

### Step 3 Install Ansible
- `sudo apt-get install ansible -y`
- `ansible --version` - checks if its installed

----------------------------------------------

## Set up ssh passwords

------------------------------------

### SSH into a machine
- `ssh vagrant@ip`
- Enter a password, it will not show on the terminal as it is protected when you write it

--------------------------------------------------------------------------

## Hosts file
In ansible, host files are those files that are used for storing information about remote nodes information, which we need to manage. The location is `/etc/ansible/hosts`. This is known as the **Default directory structure of Ansible**. You can `nano` into the hosts file and add the following for our two machines:

    [web]
    192.168.33.10 ansible_connection=ssh ansible_user=vagrant ansible_ssh_pass=vagrant

    [db]
    192.168.33.11 ansible_connection=ssh ansible_user=vagrant ansible_ssh_pass=vagrant

To check if they connect you can use the command `ansible all -m ping`

-------------------------------------------------------------------

## Run any shell commands through Ansible Ad-hoc commands
Ad-hoc commands in Ansible allow you to execute simple tasks at the command line against one or all of your hosts. An ad-hoc command consists of two parameters; the host group that defines on what machines to run the task against and the Ansible module to run. This makes things a lot easier as you can get an outcome without having to ssh inside. When on site this can be very effective when one has to deal with a high number of machines. The syntax for this is:

    ansible all -a "command"
    
`all` refers to all of the servers in the Hosts file and can be replaced by any specific machine name such as `db` or `web`. Here are a few examples of ad hoc commands that can be used.

    ansible all -a "ls -a"
    ansible db -a "pwd"
    ansible web -a "uname -a"

![image](https://user-images.githubusercontent.com/88186084/133139806-72375b70-ae81-45d2-8e1b-18b8ee797e72.png)


---------------------------------------------------------------------

## Ansible playbooks
Ansible playbooks are a completely different way to use ansible than ad-hoc task execution. Ansible playbooks are a .yml files written in YAML (Yet Another Markup Language). Playbooks record and execute Ansible’s configuration, deployment, and orchestration functions. They can describe a policy you want your remote systems to enforce, or a set of steps in a general IT process. NOTE: Indentation is really important in Playbooks.

![image](https://user-images.githubusercontent.com/88186084/133136727-bc541b31-83d6-4204-b2e2-84ea16a0e3bd.png)

- Playbooks start with 3 dashes '---'

--------------------------------------------

### Install nginx with an ansible playbook
In /etc/ansible create a new **.yml** file called `nginx_playbook.yml`. In this file you can `nano` inside to use these instructions:
      
    # Create a playbook to install nginx web server on web machine
    # web 192.168.33.10
    # Lets add 3 dashes to start the YAML
    ---
    # add the name of the host
    - hosts: web

    # gather facts about the installation steps
    gather_facts: yes

    # we need admin access 
    become: true

    # add instructions to install nginx on web machine
    tasks:
    - name: Install Nginx
      apt: pkg=nginx state=present

- Run playbook: `ansible-playbook nginx_playbook.yml`
- Check whether it works with the ad-hoc command: `ansible web -a "sudo systemctl status nginx"`

--------------------------------------------------------------------

## Copy app from controller to web
 


