# Ansible Roles

The primary purpose of ansible roles is to make our work reusable. 

```yaml
# tasks.yaml
tasks:
 - ...
```

They also help in organizing code within ansible. 
Roles introduce a set of best practices that must be followed to organize all tasks into a task directory, all variables
 used by these tasks in the vars directory, any default value goes to the default directory, all handlers go into the 
handler directory, any template used by the playbooks go into the template directory.

Roles also help in sharing our code with others in the ansible community. Ansible Galaxy is one such community where we 
can find thousands of roles for almost any task we can think of.

```shell
$ anisble-galaxy init [role-name]
--- [role-name]
 |- README.md
 |- templates
 |- tasks
 |- handlers
 |- vars
 |- defaults
 |- meta
```

```yaml
# playbook.yaml
- name: [task-name]
  hosts: [host-list]
  roles: # They must be either in the same directory as the playbook.yaml file or in the default role path
    - [role-name-1]
    - role: [role-name-2]
      become: yes
      vars:
        [var-name]: [var-value]
```

The default role path is define in the ansible config file

```text
# /etc/ansible/ansible.cfg
roles_path = /etc/ansible/roles
```

Once our role is created, we may share it with the ansible community (ansible galaxy) or in a github repo.

```shell
$ ansible-galaxy search [role-name]
$ ansible-galaxy install [role-name] < -p [dest-path] >
$ ansible-galaxy list
$ ansible-config dump | grep ROLE # get the default role path
```

