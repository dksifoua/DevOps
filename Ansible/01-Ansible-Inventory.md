# Ansible Inventory

Ansible can work with one or multiple systems in our infra at the same time. In order to work with multiple servers, 
ansible needs to establish connectivity to those servers. This is done using ssh for linux and powershell remoting for 
windows. That's what makes ansible agentless meaning that we don't need to install any additional software on the target
 machine to be able to work with ansible. A simple ssh connectivity would suffice ansible's needs.

One of the major disadvantages of most other orchestration tools is that we are required to configure an agent on the 
target systems before we can invoke any kind of automation. Now, information about these target systems is stored in an 
**inventory** file. If we don't create a new inventory file, ansible used the default inventory file located at 
`/etc/ansible/hosts` location.

The inventory file is in an `ini` like format. It's simply a number of servers listed, one after the other. We can also 
group different servers together.

```ini
<server-alias> ansible_host=<server-hostname> ansible_connection=<ssh|winrm> ansible_port=<22(default)> ansible_user=<user> ansible_ssh_pass=<passwrd>

localhost ansible_connection=localhost

web ansible_host=server1.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Pwd#3
server2.company.com

[mail]
server3.company.com
server3.company.com
```

`$ ansible [target-machine] -m ping -i [inventory-file] # Test Ansible controller can reach a target machine`
