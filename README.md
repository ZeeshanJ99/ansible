# Infrastructure as code (Iac)
Infrastructure as Code (IaC) automates the provisioning of infrastructure, enabling your organization to develop, deploy, and scale cloud applications with greater speed, less risk, and reduced cost.

-----------------------------------------------------------------

## IAC - configuration management and orchestration

**Configuration management**: Configuration management is the process of tracking and controlling the changes in a software with respect to its requirement, design, function, and development of a product. There are two types of configuration management approaches.

**Orchestration**: Orchestration is the automated configuration, management, and coordination of computer systems, applications, and services. Orchestration helps IT to more easily manage complex tasks and workflows. IT teams must manage many servers and applications, but doing so manually isn’t a scalable strategy.

-----------------------------------------------------------------------------------------

### Which tools are used for push and pull config
**Pull Model**: The nodes are dynamically updated with the configurations that are present in the server.

**Push Model**: Centralized server pushes the configurations on the nodes.

----------------------------------------------------------------------------

## What is Ansible and benefits of it?
Ansible is a software tool that provides simple but powerful automation for cross-platform computer support. It is intended for IT professionals, who use it for application deployment, updates on workstations and servers, cloud provisioning, configuration management, intra-service orchestration etc.

Its heavily used in the industry and cloud independent

-----------------------------------------------------------------------------------------------

## Why should we use it?
Because it brings huge time savings when we install packages or configure large numbers of servers. Its architecture is simple and effective. It works by connecting to your nodes and pushing small programs to them.

------------------------------------------------------------------------

## Ansible on prem, Hybird and public architecture
![image](https://user-images.githubusercontent.com/88186084/133142027-9695e725-7491-487b-bb63-7e70555d2bde.png)

--------------------------------------------------------------------------------------------

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

# Installing Ansible

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

## Install nginx with an ansible playbook
![image](https://user-images.githubusercontent.com/88186084/133139977-b4478c0b-cfab-494c-b878-e5b3bb7f52e1.png)

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

## Create a playbook to install/configure node and reverse proxy on the web machine
This code will create a new playbook. You can use `sudo nano web.yml` or any other suitable name to enter the text editor to paste this in.
      ---

    - hosts: web
      become: true

      tasks:
      - name: Clone the repository with the app file
        git:
        repo: https://github.com/ZeeshanJ99/SRE_cloud_computing
        dest: /home/vagrant/cloud_computing
        clone: yes
        update: yes

      - name: Setting the environmental variable of the DB
        shell: "echo $DB_HOST"
        environment:
          DB_HOST: mongodb://vagrant@192.168.33.11:27017/posts?authSource=admin

      - name: Echo the environment to make sure its set
        shell: "echo $DB_HOST"

      - name: Install python-software-properties
        apt: pkg=software-properties-common state=present

      - name: Add nodejs apt key
        apt_key:
          url: https://deb.nodesource.com/gpgkey/nodesource.gpg.key
          state: present

      - name: Add nodejs
        apt_repository:
          repo: deb https://deb.nodesource.com/node_13.x bionic main
          update_cache: yes

      - name: Install nodejs
        apt: pkg=nodejs state=present

      - name: install nodejs-legacy
        command: "apt-get install nodejs-legacy"

      - name: Change directory to app and install npm
        command: "npm install pm2 -g"
        args:
          chdir: "/home/vagrant/cloud_computing/app/app"

      - name: Run npm install
        command: "npm install"

---------------------------------------------------------------------

## Create a playbook to install/configure mongodb in the db machine
This code will create a new playbook. You can use `sudo nano db.yml` or any other suitable name to enter the text editor to paste this in.

    # Create a playbook to install and configure mongodb on the db machine
    # db 192.168.33.11
    ---
    - hosts: db
      gather_facts: yes
      become: true

      tasks:
      - name: install mongodb
        apt: pkg=mongodb state=present

      - name: remove mongod.conf file
        file:
          path: /etc/mongod.conf
          state: absent

      - name: touch file and set permissions
        file:
          path: /etc/mongod.conf
          state: touch
          mode: u=rw,g=r,o=r

      - name: insert contents for mongod.conf
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
-----------------------------------------------------------------------------

## Run the commands to start the app
In the controller machine, in /etc/ansible folder, run the following:

    ansible-playbook nginx_playbook.yml
    ansible-playbook mongodb_playbook.yml
    ansible-playbook node_playbook.yml

Then type the following into your browser: `192.168.33.10`, and then `192.168.33.10/posts`

To import another playbook at the end/beginning of a playbook

    - name: running the next playbook
      import_playbook: 


------------------------------------------------------

## Ansible Vault
Ansible Vault is an Ansible feature that helps you encrypt confidential information without compromising security. To install ansible on your controller machine use the following to provision ansible and all its dependencies:

    sudo apt update -y
    sudo apt-get install tree -y
    sudo apt-add-repository --yes --update ppa:ansible/ansible
    sudo apt install ansible -y
    sudo apt install python3-pip
    pip3 install awscli
    pip3 install boto boto3
    sudo apt-get upgrade -y

- check installation by using `aws--version`

- change python to python 3 if python is still 2 - `alias python=python3`

- go to etc/ansible - `sudo mkdir group_vars` then cd into group_vars

- `sudo mkdir all` cd into the all directory

- Use the command `sudo ansible-vault create pass.yml` this will then ask to create a password for the vault which will not show when you type it,

### vim commands
Once the password is set you will be sent into vim. Here are some vim commands to help you start.

- Press `ii` to Insert
- press `esc : wq` to save and exit 

![image](https://user-images.githubusercontent.com/88186084/133487464-ac8a4695-20ee-425a-a747-3b2599bc0ff9.png)



When in the vim vault paste this:

      aws_access_key: put in aws access key
      aws_secret_key: aws secret key

and exit

`sudo cat pass.yml` 
This will show us a encrypted version of the key which will show it has worked, **you should never share your keys anywhere.**

`sudo ansible db -m ping --ask-vault-pass` - redirects command to encrypted key we made. will ask for password pass we made. 

Like 2 factor authentication ^ - hardening the security using ansible vault

------------------------------------------------------------------------------------------------------------------

## Hybrid cloud infrastructure and Ansible Playbooks
Ansible playbooks provide another way to use Ansible to automate tasks. Ansible playbooks are `.yaml` or `.yml` files written in Yet Another Markup Language (YAML). They always begin with `---` at the top of the file. 

![image](https://user-images.githubusercontent.com/88186084/133486908-b23f4f1a-f6ef-4de3-9831-bbfc50bcb62c.png)

---------------------------------------------------------------------------------------------------------------

## Launch an EC2 instance on AWS using Ansible playbook
within /etc/ansible use `sudo nano create_ec2.yml` and paste the following: 

      # this playbook will launch an Ec2 instnace

      - hosts: localhost
        connection: local
        gather_facts: True
        become: True
        vars:
          key_name: enter your key name
          region: eu-west-1
          image: ami id here
          id: "SRE Zeeshan Ansible EC2"
          sec_group: "sg id here"
          subnet_id: "subnet-id here"
    # add the following line if ansible by default uses python 2.7
          ansible_python_interpreter: /usr/bin/python3
        tasks:


          - name: Facts
            block:

            - name: Get instances facts
              ec2_instance_facts:
                aws_access_key: "{{aws_access_key}}"
                aws_secret_key: "{{aws_secret_key}}"
              register: result

          - name: Provisioning EC2 instances
            block:

            - name: Upload public key to AWS
              ec2_key:
                name: "{{key_name}}"
                key_material: "{{ lookup('file', '~/.ssh{{ key_name }}.pub') }}"
                region: "{{ region }}"
                aws_access_key: "{{aws_access_key}}"
                aws_secret_key: "{{aws_secret_key}}"

             - name: Provision instance(s)
               ec2:
                 aws_access_key: "{{aws_access_key}}"
                 aws_secret_key: "{{aws_secret_key}}"
                 assign_public_ip: true
                 key_name: "{{ key_name }}"
                 id: "{{ id }}"
                 vpc_subnet_id: "{{ subnet_id }}"
                 group_id: "{{ sec_group }}"
                 image: "{{ image }}"
                 instance_type: t2.micro
                 region: "{{ region }}"
                 wait: true
                 count: 1
                 instance_tags:
                   Name: sre_zeeshan_ansible_ec2-app

               tags: ['never', 'create_ec2']


- Save and close
- Run `sudo ansible-playbook create_ec2.yml --ask-vault-pass --tags create_ec2`
- Edit `hosts` file in /etc/ansible: paste in the following (and place your instance's IP in)

      [local]
      localhost ansible_python_interpreter=/usr/local/bin/python3

      [aws]
      ec2-instance ansible_host=ec2-ip ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/sre_key.pem

- ssh into the machine - if this fails, then add `sudo` to the start of the ssh command, then exit

- then do `sudo ansible aws -m ping --ask-vault-pass`
