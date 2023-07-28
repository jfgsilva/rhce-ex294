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