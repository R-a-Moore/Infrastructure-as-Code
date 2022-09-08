# Ansible

![image](https://user-images.githubusercontent.com/47668244/188827373-e78ef9cc-8306-422c-b9b1-bb35636d06ac.png)

Ansible is a suite of open-source automation tools. Which includes software provisioning, configuration management, and application deployment.

Ansible facilitates the purpose of Infrastructure as Code (IaC).

ansible is agentless?

you don't need to install Ansible on every single instance machine you're controlling, if you have let's say 100 machines. That's why it's so powerful.

whilst there are firewalls on your cloud provider, and on your local machine, there is no protection in transit between the two. ansible vault authentication is where this comes in and you can store and maintain ssh keys etc, to connect securely between the two.

### why ansible

## Configuration Management

![IaC, with Ansible](https://user-images.githubusercontent.com/47668244/188429003-adc6b912-e34c-489a-8195-d2164b0db1f0.png)

default directory /etc/ansible
hosts - conf - roles
hosts - inventory

How does Ansible know who is the host/agent? get host name & ip, and feed it into the hosts file in /etc/ansible

### Dependancies

- Ansible is built using python, so you will need a minimum of Python 2.7 in your agent machines.

- Ansible repo link

- install Ansible itself

- make sure to update & upgrade your vagrant machines when they're launched

`sudo apt-get update -y`

`sudo apt-get upgrade -y`

`sudo apt-get install python3`

`sudo apt-get install software-properties-common`

## Using Ansible

add repository for latest ansible version. goes to ansible.com wherever the repository you define is and installs its resources. in other words `git add`
`sudo apt-add-repository ppa:ansible/ansible` press `ENTER` when prompted

install ansible, now that we've got all of the dependencies `sudo apt-get install ansible -y` then check it `ansible --version`

Step 2. Navigate to the ansible directory; `v` check you have the 'hosts' file (ls)

install tree. an additional package for us to view file trees
`sudo apt install tree`
show filepath tree
`tree`

ping your agent machines; `ping 192.168.33.10` web, `ping 192.168.33.11` db

ssh access from controller vm into agent vm; `ssh vagrant@AGENT IP` "are you sure?" `yes`, password; `vagrant`

place the existence of our agent hosts into ansible repo; `sudo nano /etc/ansible/hosts`; insert into the file...

```
[web]
192.168.33.10 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_pass=vagrant
```
gives name, ip, connection type, machine type, and password of the agent host you want to connect to, to ansible (CTRL+X, then y, then ENTER, to save and exit the nano).

Let's do this again for the db machine.
```
[db]
192.168.33.11 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_pass=vagrant
```

ping our agent hosts using ansible instead of linux; (must be in our /etc/ansible location of your controller) `ansible web -m ping`, "are you sure?" `yes`

![ping pong connection](https://user-images.githubusercontent.com/47668244/188447626-bb974646-7e29-4473-9cc3-02971e0ec62e.png)

### Ad-Hoc Commands

An Ansible ad hoc command uses the /usr/bin/ansible command-line tool to automate a single task on one or more managed nodes. ad hoc commands are quick and easy, but they are not reusable.

for example, if you wanna get the os of our web agent;
`ansible web -a "uname -a"`

or if you wish to do it with all of our host agents;
`ansible all -a "uname -a"`

![ansible adhoc example](https://user-images.githubusercontent.com/47668244/188456979-5870d34d-6afc-45ad-96aa-9fafa4376de4.png)

This is extremely powerful, as it means we don't need to ssh into each individual machines, but can still communicate to one or more of them via a secure connection.

`ansible all -a "ls"`

if you need to check the date of a machine; `ansible all -a "date"`. And then, according to the responses, you can decided based on the times. Or if you want to check if they have python installed `ansible all -a "python --version"`.

google ad-hoc command to cp test.txt from controller to web;
`ansible web -m ansible.builtin.copy -a "src=/etc/hosts/test.txt dest=/home/vagrant"` or `ansible web -m copy -a "src=/etc/ansible/test.txt dest=/home/vagrant"`


## YAML

language used in Ansible

make a yaml file `sudo nano nginx-play.yml` it uses .yml

```
# create a playbook to install nginx web server inside we machine
# --- three dashes at the start of the file of YAML

---

# add hosts or name of the host server
- hosts: web

# indentation is EXTREMELY IMPORTANT
# gather live info
  gather_facts: yes

# we need admin access
  become: true

# add the instructions 
# install nginx in web server
  tasks:
    - name: Install Nginx
      apt: pkg=nginx state=present

# the nginx server status is running
```

`---` starts of the yaml commands

`-` makes a new array

A UNIVERSAL RULE FOR YAML SCRIPT!: When making indentation, do not use tab, use two spaces to create indentation.

present means it's ensuring it's active/running, absent means it's ensuring it's not working.

`ansible-playbook nginx-play.yml` play the yaml commands

![YAML playhook example](https://user-images.githubusercontent.com/47668244/188465356-d2f533f8-02fd-4508-bed7-2448f9fcab53.png)

Use an adhoc command to check that nginx has been installed:
![YAML playhook status](https://user-images.githubusercontent.com/47668244/188465382-0adfca24-2e7e-4688-9dc1-c002ba64e3da.png)

check it out live (via ip):
![YAML playhook example show](https://user-images.githubusercontent.com/47668244/188465397-178e27b9-348f-4a0e-8e9d-89ce70e5d877.png)

### Playbook for Reverse Proxy

for this we're going to copy/replace files. though there are a couple ways to do it.

first you need to make a file which will replace the default, I named it 'default' and made a 'rev_proxy' directory to put it in, so i know what it does when i come back later;

```
server {
        listen 80 default_server;
        listen [::]:80 default_server;



       root /var/www/html;




        index index.html index.htm index.nginx-debian.html;



       server_name _;



       location / {
                proxy_pass http://localhost:3000;
        }



}
```

this is what you're playbook should look like now?

```
---
# names script
- name: rev porxy for web

# stakes which agent machine is being ordered around
  hosts: web

  # provides admin access (will kick up about /sites-available being non-writable overwise)
  become: yes
  tasks:
  # names task
  - name: to replace the default file in nginx, to allow for reverse proxy

    # copies files (replacing them)
    copy:

      # source file to copy
      src: /etc/ansible/rev_proxy/default

      # destination to copy (replace)
      dest: /etc/nginx/sites-available/default
```

### Playbook for configuring Node

```
---
# connecting to web
- hosts: web
  
  # admin access
  become: yes

  task:
    - name: purge old node.js
      shell: sudo apt-get purge nodejs npm -y
      when: nodejs_installed|failed

    - name: get node version from source
      shell: curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
      when: nodejs_installed|failed

    - name: install node
      shell: sudo apt-get install -y nodejs npm
      when: nodejs_installed|failed
```

### Playbook for app sync

```
---

- hosts: web

  become: yes

  tasks:
  - name: to add app file to web

    # copies files (replacing them)
    copy:

      # source file to copy
      src: /home/vagrant/app

      # destination to copy (replace)
      dest: /home/vagrant/
```

### Playbook for npm 

```
---

- hosts: web

  become: yes

  tasks:

  # get to app file

  # navigate to app
  - name: 
    shell: cd app/app

  # install npm
  - name: install npm
    shell: npm install express
```

#### Deploy npm

```
---

- hosts: web

  become: yes

  tasks:

  # navigate to app
  - name: 
    shell: cd app/app

  # run npm
  - name: start npm
    start: npm start&
```

#### Installing Mongodb
```
--- 

#host name
- hosts: db
  gather_facts: yes

  # admin access
  become: yes

  # add set of instructions
  tasks:
  - name: set up Mongodb in db server
    apt: pkg=mongodb state=present

  # install mongodb
  - name: remove mongodb file (delete file)
    file:
      path: /etc/mongodb.conf
      state: absent

  - name: touch a file, using symbolic modes to set the permissions (equivalent to 06044)
    file:
      path: /etc/mongodb.conf
      state: touch
      # ensures can read/write etc
      mode: u=rw,g=r,o=r
      # g for group, o for any other users

  - name: insert multiple lines and backup
    blockinfile:
      path: /etc/mongodb.conf
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

  # ensure it's running

```

`ansible-playbook PLAYBOOK_NAME --syntax-check` you can add this into the syntax (at the end) when executing a yml playbook, and it will present any syntax errors present in your playbook, such as indentation errors.

`block:` allows you to write multiple shell lines in a YAML file.

executing the playbook:

![yaml mongodb db playbook](https://user-images.githubusercontent.com/47668244/188826534-74549d4a-0339-4f54-bb8f-63990e6cb0ce.png)

and if we cat the .conf file of our mongodb we should be able to see that it's changed to allow for all IPs (0.0.0.0) on port 27017...
![yaml mongodb db playbook 2](https://user-images.githubusercontent.com/47668244/188826544-a52403ea-4687-4643-959f-9e8ac600dc24.png)

# Ansible with AWS

Now let's set up ec2 instances, using Ansible.

dependancies:

- Ansible-vault 
- python3.7 
- pip3 
- awscli
- ansible vault folder structure /etc/ansible/group_vars/all/file.yml <- this file will store the aws secret access keys.


Now that we're using ec2 instances instead of local virtual machines, the command for running playbooks is different `sudo ansible-playbook PLAYBOOK_NAME.yml --ask-vault-pass --tags ec2_create` 

We'll need to automate the ssh key access

copy file.pem as well as generate another keypair called eng122. In the playbook copy the .pub file to ec2

### Making an Ansible Vault

`sudo mkdir group_vars/all`

`sudo ansible-vault create pass.yml` create an ansible vault

give it a password (`vagrant` in this case).

press `i` to enter insert mode

give it your access & secret access keys (`access_key: XXXXXXXXXXXXXXXXXXX`) on each line, i.e., if should look something like...

```
access_key: XXXXXXXXXXXXXXXXXXXXXXXX
secret_access_key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXX
~
~
~
```

To exit the editor, press ESC (you should see the `INSERT` at the bottom disappear), then type `:wq!` (or `wq!` either seems to work for me), then press `ENTER`, and you should be placed back onto the terminal.

`sudo ansible-vault edit pass.yml` so you can edit the file

`sudo chmod 600 pass.yml` gives it permission

`ll`

### Install Python & Pip

`sudo apt install python3-pip -y`

`pip3 install boto boto3` you will need to do this in the ansible directory you're using (i.e., `/etc/ansible`)

### Install AWSCLI

`pip3 install awscli`

### Make keypair

make directory `.ssh` in /etc/ansible

then in .ssh make a keypair `ssh-keygen -t rsa -b 4096` or `sudo ssh-keygen -t rsa -b 4096` give it a name.

Next, cat your .pem key file. And paste it into a copy in your .ssh file in ansible.

### Playbook to Launch EC2

[Source](https://medium.datadriveninvestor.com/devops-using-ansible-to-provision-aws-ec2-instances-3d70a1cb155f)

vars:    ansible_python_interpreter: /usr/bin/python3  
tasks:    - debug: var=ansible_host

```
---
- hosts: localhost
  connection: local
  gather_facts: True
  become: True
  vars:
    key_name: eng122
    region: eu-west-1
    image:  ami-07b63aa1cfd3bc3a5
    id: "eng122_christian_ansible_test_3"
    sec_group: "sg-0369e1048d7b954fc"
    subnet_id: "subnet-0429d69d55dfad9d2"
# add the following line if ansible by default uses python 2.7
    ansible_python_interpreter: /usr/bin/python3
  tasks:

    - name: Facts
      block:

      - name: Get instances facts
        ec2_instance_facts:
          aws_access_key: "{{ec2_access_key}}"
          aws_secret_key: "{{ec2_secret_key}}"
          region: "{{ region }}"
        register: result


    - name: Provisioning EC2 instances
      block:

      - name: Upload public key to AWS
        ec2_key:
          name: "{{ key_name }}"
          key_material: "{{ lookup('file', '/etc/ansible/.ssh/{{ key_name }}.pub') }}"
          region: "{{ region }}"
          aws_access_key: "{{ec2_access_key}}"
          aws_secret_key: "{{ec2_secret_key}}"


      - name: Provision instance(s)
        ec2:
          aws_access_key: "{{ec2_access_key}}"
          aws_secret_key: "{{ec2_secret_key}}"
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
            Name: eng122_christian_ansible_test_3

      tags: ['never', 'create_ec2']
```

Make note that everytime you spin up an instance, you will need to give it a neew instance tag name (this example currenlty being - `eng122-christian-ansible-app`), as well as the id, security group tag, etc. Including the ami and other configurations that are relevant to the instance you're currently spinning up.

`sudo ansible-playbook --ask-vault-pass playbook.yml --tags INSTANCE_TAG` to execute the playbook using the pass.yml access keys and password. Must insert the tag for the current instance that you've set in the playbook (for this example `create_ec2`).

### Accessing your Instance in Ansible

Once you've made your instance using the playbook, you'll want to access it, using your controller.

To do this, first you need to add it as a group into your /hosts file.

`sudo nano hosts` to edit hosts file

insert your ec2 machines with the key connection that you've just placed into your new .ssh file...

```
[local]
localhost ansible_python_interpreter=/usr/local/bin/python3

[aws]

ec2-instance ansible_host=ec2-ip ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/eng122.pem

```

### Connecting through SSH

escalate .pem ssh key so you can actually ssh into it; `chmod 400 .pem key`

Is you cannot use your typical ssh command to connect into your instance, try replacing your `.pem` file with your private ssh key (if you have one). I.e., rather than `ssh -i "eng122_christian_pair.pem" ubuntu@ec2-54-247-16-22.eu-west-1.compute.amazonaws.com` instead do `ssh -i "eng122_christian_pair" ubuntu@ec2-54-247-16-22.eu-west-1.compute.amazonaws.com`

172.31.16.107

