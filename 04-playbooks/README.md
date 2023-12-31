# Playbooks
## Syntax
```yaml
---   # three dashes declare it's a playbook
# playbook name starts with dash and col = 0
- name: task name here  
  become: yes
  hosts: all # you can use limit clause
  tasks:
# tasks start with - name: and aligned with tasks keyword
  - name: install httpd
    dnf:
      name: httpd
      state: present
  
  - name: enable service
    service:
      name: httpd
      state: started
      enabled: yes
```
## basic structure
playbook contains plays, plays contains tasks. The above playbook only contains one play.
* Plays start with a hyphen
* Each play has a name, target hosts and list of properties and tasks
* Tasks are more indented then play and also use a hyphen before the name
* Use the fqdn for the module used with the task
* Tasks arguments are more indented than the task level
* Use spaces for indentation

this playbook can be executed with
`$ ansible-playbook -i inventory install-httpd.yaml --limit node2`

## Playbook required keywords
| level | keyword | usage |
| ----- | ------- | ----- |
| play | hosts | target hosts for the playbook|
| play | name | The name of the play |
| play | tasks | One or more tasks to be executed |
| tasks | name | task name |
| tasks | module name | module to be used in task |
| task | arguments | arguments to be used in task |

### lists
```yaml
---
- name: task name here
  become: yes
  hosts: all # you can use limit clause
  tasks:
  - name: install httpd
    dnf:
      name: httpd
      state: present
  
  - name: enable service
    service:
      name: httpd
      state: started
      enabled: yes
```      
```yaml
- name: Remove several packages
  hosts: node2
  tasks:
  - name: Remove httpd and mysql and tmux
    dnf:
      name:
        - httpd
        - mysql
        - tmux
      state: absent
```
## tips
* syntax check with
`$ ansible-playbook remove-several.yaml --syntax-check`
* perform dry runs with
`$ ansible-playbook remove-several.yaml --check --diff`
or `ansible-navigator run install-several.yaml --check --diff`

### multiline files
```yaml
---
- name: Create a file with multiline content
  hosts: node2
  tasks:
  - name: Create a file with several lines inside using pipe symbol
    copy:
      content: |
        this is line1
        this is line2
      dest: /home/vagrant/multiline.file
  - name: Create a file with concatenating lines inside using > symbol
    copy:
      content: >
        this is line1
        this is line2
      dest: /home/vagrant/multiline2.file
```

## multiplay playbooks
### tips
* use includes instead of large playbooks
* uselocal host to verify accessibility of services on managed hosts
* use -vv or -vvv flag to increase verbosity to debug

