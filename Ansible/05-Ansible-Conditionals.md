# Ansible Conditionals

```yaml
- name: Install nginx
  hosts: all
  tasks:
    - name: Install nginx on debian
      apt:
        name: nginx
        state: present
      when: ansible_os_family == 'Debian' and ansible_distribution_version == '16.04'
    - name: Install nginx on redhat
      yum:
        name: nginx
        state: present
      when: ansible_os_family == 'RedHat' or ansible_os_family == 'SUSE'
```

**Conditional loops**

```yaml
- name: Install nginx
  hosts: all
  vars:
    packages:
      - name: nginx
        required: True
      - name: mysql
        required: True
      - name: apache
        required: False
  tasks:
    - name: Install "{{ item.name }}" on debian
      apt:
        name: "{{ item.name }}"
        state: present
      when: item.required == True
      loop: "{{ packages }}"
```

**Conditionals & Register**

Use a conditional with the output of the previous task

```yaml
- name: Check the status of a service and email if it's down
  hosts: localhost
  tasks:
    - command: service httpd status
      register: result
    - mail:
        to: email@company.com
        subject: Service Alert
        body: Httpd service is down
      when: result.stdout.find('down') != -1
```