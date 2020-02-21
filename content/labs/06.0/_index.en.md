---
title: "6.0 - Managing Secrets with Ansible Vault"
weight: 60
---

In this lab we are going to practice encryption in Ansible playbooks. It assumes your working directory is always `/home/ansible/techlab/`.

### Task 1

- Create a simple playbook called `secretservice.yml` which creates a file `MI6` in the `/etc/` directory on node1 and node2. Use the `template` module and a template named `nsa.j2`. Don’t encrypt anything yet and use the inventory hosts from the earlier labs.
- The content of the file `MI6` should be:
  ```
    username: jamesbond
    password: miss_moneypenny
  ```
- Run the playbook and check if the file `/etc/MI6` has been deployed on the nodes.

### Task 2

- Make the playbook `secretservice.yml` use a variable file named `secret_vars.yml` with the content:
  ```
    var_username: jamesbond
    var_password: miss_moneypenny
  ```
- Rewrite the `nsa.j2` template to use the variables from the `secret_vars.yml` file. Nothing is encrypted yet.
- Rerun the playbook and remember nothing has been encrypted yet.

### Task 3

- Create a file named `vaultpassword` containing the unencrypted string "goldfinger".
- Encrypt the `secret_vars.yml` file by using `ansible-vault` with the password *goldfinger*.

{{% notice tip %}}
You don’t have to set a label when encrypting the file.
{{% /notice %}}

- Rerun the playbook providing the password for decrypting `secret_vars.yml` at the command prompt.
- Rerun the playbook providing the password for decrypting `secret_vars.yml` from the file `vaultpassword`.

{{% notice tip %}}
Since the password is in cleartext in the file vaultpassword, you shoul never ever push it to a git repository or similar. Also doublecheck that only the necessary permissions are set.
{{% /notice %}}


### Task 4

- Configure your environment to always use the `vaultpassword` file as the vault file.
- Rerun the playbook without providing the password or the passwordfile at the commandline.

### Task 5

- Decrypt the file `secret_vars.yml`.
- Encrypt the values of the variables `username` and `password` and put them into the `secret_vars.yml` file.

{{% notice note %}}
Look for an option to ansible-vault to give the name of the variable while encrypting the value. This makes it easyier to copy-paste the output later!
{{% /notice %}}

### Task 6

- Remove the `/etc/MI6` file on the nodes using an ad-hoc command.

### Task 7

- Encrypt another file secret_vars2.yml. Ensure its encrypted with your vault password file `vaultpassword`
- Change the encryption of the file: encrpyt it with another password provided at the commandline.

{{% notice note %}}
 Don't do this by decrypting & reencrypting but rather by using the `rekey` option. 
 There's a trap hidden here. Doublecheck if everything worked as you expected.
{{% /notice %}}

- Check the content of `secret_vars2.yml` by viewing it and providing the password at the command line.

### TASK 8

- What can you do, to avoid Ansible to print out sensitive data at runtime?

{{% notice tip %}}
Take a look at [docs.ansible.com](https://docs.ansible.com)
{{% /notice %}}

## Solutions

{{% collapse solution-1 "Solution 1" %}}
```bash
$ cat nsa.j2 
username: jamesbond
password: miss_moneypenny

$ cat secretservice.yml 
---
- hosts: node1, node2
  become: yes
  tasks:
    - name: put template
      template:
        src: nsa.j2
        dest: /etc/MI6

$ ansible-playbook secretservice.yml
```
{{% /collapse %}}


{{% collapse solution-2 "Solution 2" %}}

```bash
$ cat secret_vars.yml 
---
var_username: jamesbond
var_password: miss_moneypenny

$ cat nsa.j2 
username: {{ var_username }}
password: {{ var_password }}

$ cat secretservice.yml 
---
- hosts: node1, node2
  become: yes
  vars_files:
    - secret_vars.yml
  tasks:
    - name: put template
      template:
        src: nsa.j2
        dest: /etc/MI6

$ ansible-playbook secretservice.yml
```
{{% /collapse %}}

{{% collapse solution-3 "Solution 3" %}}
```bash
$ cat vaultpassword 
goldfinger

$ ansible-vault encrypt secret_vars.yml --vault-id vaultpassword

$ ansible-playbook secretservice.yml --vault-id vaultpassword
```
{{% /collapse %}}

{{% collapse solution-4 "Solution 4" %}}

Make sure you recieve the following output in your terminal:

```bash
$ grep ^vault /home/ansible/techlab/ansible.cfg 
vault_password_file = /home/ansible/techlab/vaultpassword

$ ansible-playbook secretservice.yml
```

{{% /collapse %}}

{{% collapse solution-5 "Solution 5" %}}

```bash
$ ansible-vault decrypt secret_vars.yml
$ echo "---" > secret_vars.yml
$ ansible-vault encrypt_string jamesbond -n var_username >> secret_vars.yml
$ ansible-vault encrypt_string miss_moneypenny -n var_password >> secret_vars.yml
```

Content of secret_vars.yml
```yaml
---
var_username: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          65336662623334393265373462616231323635623034653534393861666637333232383438393534
          3264376362633566313337333835313832376566343362330a636639346263323961636232306134
          35393462343935653031353430636666326232343565383330386339646436376265316264376366
          3336326566663033300a396666316461356336313564323236333138623465373439343032333930
          6664
var_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          33366638383438373238333335663933323663326630356564626139323135306563343335613331
          6534373239393234366431656234386232373331316634660a646665303838636465303638316366
          63393034643639393764666634303338636130326164366339303634643264646235323637326661
          3633393039613263390a653062383834323661386661313733393662393935663263633565396133
          3931
```
```bash
$ ansible-playbook secretservice.yml
```

{{% /collapse %}}

{{% collapse solution-6 "Solution 6" %}}
```bash
$ ansible node1,node2 -i inventory/hosts -b -a "rm /etc/MI6"
```

{{% notice tip %}} 
Note that the `command` module is the `default` module and therefore has not to be specified here.
{{% /notice %}}

{{% /collapse %}}

{{% collapse solution-7 "Solution 7" %}}

```bash
$ ansible-vault encrypt secret_vars2.yml
```

Be sure to use `rekey` with `--new-vault-id`. By using `--vault-id`, ansible-vault would use the value from the vaultpasswordfile and not the one asked for by using `@prompt`. This could be quite misleading... You can check the same unexpected behaviour when trying to view an encrpyted file with providing a wrong password at cmdline. Giving a wrong password after `ansible-vault view secret_vars2.yml --vault-id @prompt` still results in showing the decrypted content of the file when ansible.cfg points to the correct vaultpasswordfile.

There is an open [issue](https://github.com/ansible/ansible/issues/33831) about this topic on github.

```bash
$ ansible-vault rekey secret_vars2.yml --new-vault-id @prompt
$ ansible-vault view secret_vars2.yml --vault-id @prompt
```

{{% /collapse %}}

{{% collapse solution-8 "Solution 8" %}}
{{% notice tip %}}
See [Ansible Docs: Logging](https://docs.ansible.com/ansible/devel/reference_appendices/logging.html) and [Ansible Docs: FAQ - How do I keep secret data in my playbook?](https://docs.ansible.com/ansible/devel/reference_appendices/faq.html#keep-secret-data)
{{% /notice %}}

```bash
no_log: true
```

{{% /collapse %}}
