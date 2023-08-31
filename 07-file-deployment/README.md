# File Deployment

## Modules stat and file

Several modules can be used to depoly file, depending on the objectives.
| module | Usage |
| ------ | ----- |
| copy | copy to remote host |
| fetch | gets files from remote host |
| file | manages files and properties |
| find | finds files based on properties |
| lineinfile | manages lines in textfiles |
| blockinfile | manages blocks of text in files |
| replace | replaces strings in files based on regex |
| synchronize | rsync based tasks |
| stat | gets stats on file | 

### stat - manage file attributes

stat modules gets files status and is mainly used to get infofrom the file and is used in tandem with the register and when keywords in ansible

```yaml
---
- name: playbook demonstrating stat usage
  hosts: localhost
  tasks:
  - name: using stat
    stat:
      path: /etc/hosts
    register: content
  - name: diplaying contents 
    debug:
      msg: "{{ content.stat }}"
    when:
      content.stat.exists
```

you can also fail controlling the flow

```yaml
---
- name: playbook demonstrating stat usage and controling the flow
  hosts: localhost
  tasks:
  - name: create a file
    command:
      touch /tmp/myfile
  - name: using stat
    stat:
      path: /tmp/myfile
    register: content
  - name: diplaying contents 
    debug:
      msg: "{{ content.stat }}"
  - fail:
      msg: "user and/or permissions are incorrect. owner is {{ content.stat.pw_name }} and permissions are {{ content.stat.mode }}"
    when:
      content.stat.pw_name != "root" and content.stat.mode != "0600"
```

### file

file module is used to change file attributes, works great with stat module to conditionally change something. You can also use it to create files and dirs, or remove them.

```yaml
---
- name: playbook demonstrating stat and file module
  hosts: localhost
  become: true
  tasks:
  - name: create a file
    command:
      touch /tmp/myfile
  - name: using stat
    stat:
      path: /tmp/myfile
    register: content
  - name: diplaying contents 
    debug:
      msg: "{{ content.stat }}"
  - file:
      path: /tmp/myfile
      group: jfcgsilva
      mode: '0600'
      owner: root 
    when:
      content.stat.pw_name != "root" and content.stat.mode != "0600"
```

* creating and removing files


## File contents

### lineinfile

using regex to change a line in some file

```yaml
---
- name: lineinfile usage example
  hosts: localhost
  tasks:
  - name: change resolv.conf
    lineinfile:
      path: /etc/resolv.conf
      regexp: '^nameserver'
      line: 'nameserver 8.8.8.8'
    notify: restart_network
  - name: configure sshd
    lineinfile:
      path: /etc/ssh/sshd_config
      regexp: '^PermitRootLogin'
      line: 'PermitRootLogin no'
    notify:
      restart_sshd

  handlers:
  - name: restart_sshd
    service:
      name: sshd
      state: restarted
  - name: restart_network
    service:
      name: NetWorkManager
      state: restarted
```

### blockinfile

blockinfile inserts a block delimited by markers. These markers can be changed with *marker*
Also, using the "|" char after block appends the newline char at de end of the line. Using ">" would concatenate the different lines in a single one in the resulting file.

```yaml
---
- name: lineinblock usage example
  hosts: localhost
  vars_files:
    vars/lineinblock
  tasks:
  - name: copy /etc/hosts
    copy:
      src: /ect/hosts
      dest: /home/{{ user }}/hosts
  - name: managing /etc/hosts copy example
    # don't forget to add the pipe and permissions are {{ content.stat    .permissions }}
    blockinfile:
      path: /home/{{ user }}/hosts
      block: |
        192.168.0.23 host1.rhce-ex294.com
        192.168.0.24 host2.rhce-ex294.com
        192.168.0.25 host3.rhce.ex294.com
      state: present
```
### fetch

Using fetch it's easy to get log data from nodes and process it in control node for evaluation. Also node the usage of {{ range(x,y)|list }} in the loop section to generate a integer sequence


```yaml
---
- name: using fetch to get log from slave nodes
  hosts: all
  tasks:
  - name: fetching log data
    fetch:
      dest: /tmp
      src: /var/log/wtmp
- name: conciliating the binaries in a single readable file
  hosts: localhost
  vars:
    prefix: /tmp/node
    suffix: /var/log/wtmp
    conciliated_file: allnodes-last.out
  tasks:
  - name: ouput the log into file
    shell: last -f {{ prefix }}{{ item }}{{ suffix}} >> /tmp/{{ conciliated_file }}
    loop: "{{ range(1,5)|list }}" # also use - 1\n - 2\n - 3\n - 4\n

```
