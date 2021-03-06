---
title: "4.1 - Ansible Playbooks - Variables and Loops"
weight: 41
---

In this lab we’ll start to use variables and loops.

### Task 1

- In your playbook `webserver.yml` you have two tasks for starting and enabling `httpd` and `firewalld`. Merge these 2 tasks into one.

{{% notice tip %}}
Remember `loop:` or `with_items:`
{{% /notice %}}

### Task 2

- In your playbook `webserver.yml`, ensure that that the yum package `firewalld` is installed. Do the installation of `httpd` and `firewalld` in one task. Do you really need to use a loop? Have a look at the description of the ansible module yum.

### Task 3

- Write a new playbook `motd.yml` which sets the content of `/etc/motd` on all servers to a custom text. Use the variable `motd_content` and the `copy` module with the option `content` containing the variable.

### Task 4

- Using the command line, overwrite the content of `motd` by providing the variable `motd_content` with a different value.
- Modify the content again, but use a `vars.yml` file.

### Task 5

- Set the `motd_content` from Task 4 using `group_vars` for node1 and `host_vars` for node2.
- Make sure you remove the variable definition in `motd.yml`. Reason being it will have a higher priority.
- Limit the run to node1 and node2.

{{% notice tip %}}
  Think about where you have to create the folders for your host and group variables
{{% /notice %}}

### Task 6

- Get a feeling for errors: Remove the quotes around the curly brackets and have a look at the output.

## Solutions

{{% collapse solution-1 "Solution 1" %}}

Delete the 2 tasks "start and enable \[httpd,firewalld\]". Add a new task with the following content:
```yaml
- name: start and enable services
  service:
    name: "{{ item }}"
    state: started
    enabled: yes
  with_items:
    - httpd
    - firewalld
```

{{% notice tip %}}
Make sure your indentations are correct!
Older ansible-versions don’t know the keyword "loop" yet, use "with\_items" instead.
{{% /notice %}}

{{% /collapse %}}

{{% collapse solution-2 "Solution 2" %}}
```yaml
tasks:
  - name: install httpd and firewalld
    yum:
      name:
        - httpd
        - firewalld
      state: installed
```

{{% notice tip %}}
See [Ansible Docs - Yum Module](https://docs.ansible.com/ansible/latest/modules/yum_module.html#yum-module)
{{% /notice %}}

{{% /collapse %}}


{{% collapse solution-3 "Solution 3" %}}
Content of motd.yml:

```yaml
---
- hosts: all
  become: yes
  vars:
    motd_content: "Thi5 1s some r3ally stR4nge teXT!\n"
  tasks:
   - name: set content of /etc/motd
     copy:
       dest: /etc/motd
       content: "{{ motd_content }}"
```
```bash
$ ansible-playbook motd.yml
```

Take a look at what your playbook just did:

```bash
$ ssh -l ansible <node1-ip>
Last login: Fri Nov  1 14:16:08 2019 from 5-102-146-174.cust.cloudscale.ch
Thi5 1s some r3ally stR4nge teXT! # <-- it worked!
[ansible@node1 ~]$
```
{{% /collapse %}}

{{% collapse solution-4 "Solution 4" %}}

```bash
$ ansible-playbook motd.yml --extra-vars motd_content="0th3r_5trang3_TExt"

$ ssh -l ansible <node1-ip>
Last login: Fri Nov  1 14:18:52 2019 from 5-102-146-174.cust.cloudscale.ch
0th3r_5trang3_TExt # <-- it worked
[ansible@node1 ~]$
```

```bash
$ cat vars.yml
---
motd_content: "st1ll m0r3 str4ng3 TexT!"
$ ansible-playbook motd.yml --extra-vars @vars.yml
```

Login via SSH again and check if the new text was set.
{{% /collapse %}}

{{% collapse solution-5 "Solution 5" %}}

Your `motd.yml` should look something like this:

```yaml
---
- hosts: all
  become: yes
  tasks:
    - name: set content of /etc/motd
      copy:
        dest: /etc/motd
        content: "{{ motd_content }}"
```

After creating the new directories and files you should have something similar to this:

```bash
$ cat inventory/group_vars/web.yml
---
motd_content: "This is a webserver\n"
$ cat inventory/host_vars/node2.yml
---
motd_content: "This is node2\n"
```

Run your playbook and check if the text was changed accordingly on the two nodes:

```bash
$ ansible-playbook motd.yml -l node1,node2

$ ansible web,node2 -a "cat /etc/motd"
```
{{% /collapse %}}

{{% collapse solution-6 "Solution 6" %}}
```yaml
---
- hosts: all
  become: yes
  tasks:
    - name: set content of /etc/motd
      copy:
        dest: /etc/motd
        content: {{ motd_content }} #<-- missing quotes here
``` 
{{% /collapse %}}
