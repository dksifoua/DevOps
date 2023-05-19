# Ansible Loops

```yaml
- name: Create users
  hosts: localhost
  tasks:
    - user: name="{{ item }}" state=present
      loop:
        - joe
        - george
        - ravi
        - mike
```

```yaml
- name: Create users
  hosts: localhost
  tasks:
    - user: name="{{ item.name }}" state=present uid="{{ item.uid }}"
      loop:
        - name: joe
          uid: 1010
        - name: george
          uid: 1011
        - { name: ravi, uid: 1012 }
        - { name: mike, uid: 1013 }
```

**With_**
 
With_* directives is just like loops

```yaml
- name: Create users
  hosts: localhost
  tasks:
    - user: name="{{ item }}" state=present
      with_items:
        - joe
        - george
        - ravi
        - mike
```

There are many with_* directives:

- with_items
- with_files
- with_url
- with_mongodb
- with_dict
- with_etcd
- with_env
- with_filetree
- with_ini
- with_inventory_hostnames
- with_k8s
- with_manifold
- with_nested
- with_nios
- with_openshift
- with_password
- with_pipe
- with_rabbitmq
- with_redis
- with_sequence
- with_skydrive
- with_subelements
- with_template
- with_together
- with_varnames
- etc.

