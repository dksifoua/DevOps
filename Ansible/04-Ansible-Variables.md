# Ansible Variables

Just like any other scripting or programming language, variables are used to store values that varies with different 
items. For example, let's say we are trying to perform the same operation of applying patches to hundred of servers, we 
only need a single playbook for all hundred servers. However, it's the variables that store information about the 
different hostnames, usernames, or passwords that are different for each server.

We've already used variables inside the inventory file. We define variable inside a playbook.

```yaml
# playbook-name.yaml
- name: [play-name]
  hosts: [hosts]
  vars:
    [name]: [value]
  tasks:
    - ...
```
We can also have variable in a file dedicated for it.

```yaml
variable1: value1
variable2: value2
```

**How to use variables?**

- Declare them:

Either

```ini
# Inventory file
Web http_port=8081 smp_port=161-162 inter_ip_range=192.0.2.0
```

Or

```yaml
# web.yaml
http_port: 8081
smp_port: 161-162
inter_ip_range: 192.0.2.0
```

- Use them:

```yaml
# playbook-name.yaml
- name: Set firewall conf
  hosts: web
  tasks:
    - firewalld:
        service: https
        permanent: true
        state: enabled
    - firewalld:
        port: '{{ http_port }}'/tcp
        permanent: true
        state: disabled
    - firewalld:
        port: '{{ smp_port }}'/udp
        permanent: true
        state: disabled
    - firewalld:
        port: '{{ inter_ip_range }}'/24
        zone: internal
        state: enabled
```