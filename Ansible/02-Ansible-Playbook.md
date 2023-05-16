# Ansible Playbooks

Ansible's playbooks are ansible orchestration language. It is in playbooks where we define what we want ansible to do. It 
is a set of instructions we provide ansible to work its magic. 

For example, it can be as simple as running a series of commands on different servers in a sequence, and restarting 
those servers in a particular order.

```yaml
# simple ansible playbook
- Run command 1 on server 1
- Run command 2 on server 2
- Run command 3 on server 2
- Restarting server 1
- Restarting server 2
- Restarting server 3
```

Or it could be as complex as deploying hundreds of vms in a public and private cloud infra, provisioning storage to vms,
 setting up their network and cluster configurations, configuring application on them such as web server or database 
server, setting up load balancing, setting up monitoring components, installing and configuring backup clients and 
updating configuration database with information about the new vms, etc.

```yaml
# complex ansible playbook
- Deploy 50 vms on public cloud
- Deploy 50 vms on private cloud
- Provision storage on all vms
- Setup network config in private vms
- Setup cluster configuration
- Configure web servers on 20 public vms
- Configure db servers on 20 private vms
- Setup load balancing between web server vms
- Setup monitoring components
- Install and configure backup clients
- Update cmdb with new vm config
```

Let's take a closer look at how playbooks are written (yaml format).

```yaml
- Playbook - A single yaml file
  - Play - Defines a set of activities (tasks) to be run on hosts
    - Task - An action to be performed on the host. Can be:
      - Execute command
      - Run a script
      - Install package
      - Shutdown/Restart
```

Let's take a look at an actual playbook

```yaml
# playbook-name.yaml
- name: Play 1
  hosts: localhost # Must be present in the inventory file. Can be also a group
  tasks:
    - name: Execute the command `date`
      command: date
    - name: Execute a script on server
      script: test_script.sh
- name: Play 2
  hosts: localhost
  tasks:
    - name: Install httpd service
      yum:
        name: httpd
        state: present
    - name: Start the web server
      service:
        name: httpd
        state: started
```

The different actions run by tasks are called **modules**. In this case `command`, `script`, `yum`, and `service` are 
ansible modules. There are hundreds of other modules available out of the box. Information about these modules is 
available in the ansible documentation website, or we can simply run the command `$ ansible-doc -l`.

Once we successfully build the ansible playbook, we execute the ansible playbook by running the command:

```shell
$ ansible-playbook playbook-name.yaml # Run the ansible playbook
$ ansible-playbook-help # Get to know some additional parameters available for the ansible-playbook command
```