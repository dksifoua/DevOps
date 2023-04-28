# Introduction to Ansible

As a system engineer or IT admin, we are probably involve doing a lot of repetitive tasks in our environment such as 
infra provisioning, configuration management, continuous delivery, application deployment or security and compliance. 
All these very repetitive tasks involve execution of hundred of commands on hundred of different servers while 
maintaining the right sequence of events with system reboots and whatnot in between.

Some people develop scripts to automate these tasks. But that requires coding skills and regular maintenance of these 
scripts, and a lot time in putting these scripts together on the first place. That's where ansible helps. Ansible is a 
powerful IT automation tool that can be learn quickly. It's simple enough for everyone in IT, yet powerful enough to 
automate even the most complex deployments.

In the past, something that took a complex scripts, now takes just a few lines of instructions in a ansible automation 
playbook. Whether we want to make that happen on our localhost or on all of our databases for example, all it takes is 
just modifying one line of code.

**Script**

```shell
#!/bin/bash
# Script to add a user to Linux system
if [ $(id -u) -eq 0 ]; then
  $username=johndoe
  read -s -p "Enter password: " password
  egrep "^$username" /etc/passwd > /dev/null
  if [ $? -eq 0 ]; then
    echo "$username exists!"
    exit 1
  else
    useradd -m -p $password $username
    [ $? -eq 0 ] && echo "User has been added to the system!" || echo "Failed to add the user!"
  fi
fi
```

**Ansible Playbook**

```yaml
- hosts: all_servers
  tasks:
    - user:
        name: johndoe
```

## Use case example - simple

Let's say we have a number of hosts in our environment that we would to restart in a particular order. Some of them are 
web servers and others are database servers. So we would like to power down the web server first, followed by the 
database servers, and then  power up the database servers and then the web servers. We can an ansible playbook to get 
this done in matter of minutes and simply invoke the invoke the ansible playbook every time we wish to restart our app.

## Use case example - complex

Let's say we are setting up a complex infra that spans across public and private clouds and that includes hundreds of 
virtual machines. With ansible, we can provision vms on public cloud like amazon and as well as private cloud 
environments like VMWare, and move on to configuring applications on those and setting up communication between them, 
such as modifying configuration files, installing applications on them, configuring firewall rules, etc. 

There are a lot of builtin modules available in ansible that supports these kind of operations. We can easily integrate 
ansible with the rest of our environment so that we can pull information to be used in the automation process, such as 
data from a configuration management database to get the list of vms we want to target, or we can configure ansible to 
trigger automation automatically from tools like ServiceNow when a workflow get approved.
