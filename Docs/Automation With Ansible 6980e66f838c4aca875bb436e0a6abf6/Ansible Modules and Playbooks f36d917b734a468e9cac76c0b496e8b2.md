# Ansible Modules and Playbooks <!-- omit in toc -->

## 🥡 Table of Contents <!-- omit in toc -->

- [🌤️ Ansible Modules](#️-ansible-modules)
  - [⚙️ The Setup Module](#️-the-setup-module)
  - [📂 The File Module](#-the-file-module)
  - [🇨🇴 Color notations used during execution and idempotent operations](#-color-notations-used-during-execution-and-idempotent-operations)
  - [🔃 The copy Module](#-the-copy-module)
  - [🧭 The command Module](#-the-command-module)
  - [🧷 Fetch Module[doc]](#-fetch-moduledoc)
  - [📃 Ansible-doc](#-ansible-doc)
- [🇬🇾 `YAML` - The Playbook Language](#-yaml---the-playbook-language)
  - [🕵️ Python Interprets YAML](#️-python-interprets-yaml)
- [▶️ Ansible PlayBook - Introduction](#️-ansible-playbook---introduction)
  - [🃏 Ansible Playbooks, Breakdown of Sections](#-ansible-playbooks-breakdown-of-sections)
  - [🐾 Playbooks Introduction](#-playbooks-introduction)
  - [🍾 Message of the day - Playbook Example](#-message-of-the-day---playbook-example)
  - [🧛🏾 Variables in Playbook](#-variables-in-playbook)
  - [🦾 Handlers in Playbook](#-handlers-in-playbook)
  - [🧇 Decision Making in playbooks (`when`)](#-decision-making-in-playbooks-when)
  - [📈 Challenge: Playbook](#-challenge-playbook)
- [👩‍✈️Ansible Playbooks, Variables](#️ansible-playbooks-variables)
- [🏭 Ansible Playbooks, Facts](#-ansible-playbooks-facts)
  - [🏵️ The `setup` module, and `ansible_facts`](#️-the-setup-module-and-ansible_facts)

## 🌤️ Ansible Modules

### ⚙️ The Setup Module

- The module automatically called when using playbooks, to gather useful information as variables, about remote targets, etc.. which is used at the execution time.
- This module can be called directly to find out the variables, available to a host.
- It can be also used in windows,
- Ansible provides many `facts`(info. provided by the setup module called facts), about target system automatically.
- The setup is now (as of ansible 2.10) is a 'builtin' plugin, referenced by `setup` or `ansible.builtin.setup`, which is in the `ansible.builtin` collection.
- Docs:- [https://docs.ansible.com/ansible/latest/collections/ansible/builtin/setup_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/setup_module.html)

run the `setup` in centos1,

```bash
ansible centos1 -m setup

# That gives a hell lot of infos, pipe through more
ansible centos1 -m setup | more
```

Understanding certain info from facts, is another process → using some other modules too.

### 📂 The File Module

- Sets/removes files, symlinks and directories. sets attributes for files
- Using the file module with the combination of `path` and `state`, will full-fill the same outcome as using a `touch` in UNIX (used to create a `0 len` file or update existing file timestamp)

```bash
# creating a test file in all hosts
ansible all -m file -a 'path=/tmp/test state=touch'

# check on client
ls -altrh /tmp/test

# on remote
ssh centos2 ls -altrh /tmp/test
```

### 🇨🇴 Color notations used during execution and idempotent operations

- RED 🟥- Failure
- GREEN 🟩 - Success with No Changes
- YELLOW 🟨 -  Success with changes

By re-running the `ansible all -m file -a 'path=/tmp/test state=touch'` command, it will give a 'yellow' out as the file timestamps changed,

![unix-permission.png](Ansible%20Modules%20and%20Playbooks%20f36d917b734a468e9cac76c0b496e8b2/unix-permission.png)

```bash
# set the file permission to 600
ansible all -m file -a 'path=/tmp/test state=file mode=600'
```

yellow out all the hosts files modes changed

- rerun it, then gets green that the operation is successful but nothing changed,

🪃 **Idempotent Operations**

> An operation idempotent, if the result of performing it once, is exactly the same as the result of performing it repeatedly without any intervening actions.
>

 This one is crucial as the hosts can be in variant states,

### 🔃 The copy Module

- Copy files from the local or remote target, to a location on the remote target. Use the `[fetch]` module, to copy files from a remote target, to a local target,

    If created a file `/tmp/copy_test.txt`, in `ubuntu-c` and need to copy this one to all hosts, file will be copied to all hosts with a "yellow" change indication, and a green success will be displayed on `ubuntu-c` as the end-state already satisfied.

    ```bash
    # create a file
    touch /tmp/copy_test.txt

    # copy it
    ansible all -m copy -a 'src=/tmp/copy_test.txt dest=/tmp/copy_test.txt'
    ```

    copy module uses "checksum"s to validate whether copy needed or not,

- To copy files on the remote system to the remote system, include `remote_src=yes`,

    ```bash
    # copy files from path x -> y in remote hosts
    ansible all -m copy -a 'remote_src=yes src=/tmp/x dest=/tmp/y'
    ```

- If needed variable interpolation in copied files use the `template` module.
- `win_copy` module for windows.
- Can be referenced using `copy` or `ansible.builtin.copy`, [documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html).

### 🧭 The command Module

- The `command` module takes the command name followed by a list of space-delimited arguments, the given command will be executed on all selected nodes.
- ***note:- It is not processed through the shell, so variables like `$HOME` and operations like `<, >, |, ;, &` will not work. Use the `[shell]` module if needed them.***

    No need to specify any module names as `command` is the default module,

- run a command to display `hostname` of every hosts,

    ```bash
    ansible all -a 'hostname' -o
    ```

- `create`s and `removes` variable that can be used for idempotent operations,

    ```bash
    ansible all -a 'touch /tmp/copy_test2.txt creates=/tmp/copy_test2.txt'
    ```

    It gives a warning as `toch` is supported naively with `file` module, to ignore warning `command_warning` flag to cnf,

    if same command run again as `creates` specified it will skip the command execution(file already exists), instead if `removes` specified only executes the command if the file exists,

    ```bash
    ansible all -a 'rm /tmp/copy_test2.txt removes=/tmp/copy_test2.txt'
    ```

- `[win_command]` module for windows
- Referenced using `command` or `ansible.builtin.command`, [documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/command_module.html).

### 🧷 Fetch Module[[doc](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/fetch_module.html)]

- Test,

    ![2021-09-08_08-24.png](Ansible%20Modules%20and%20Playbooks%20f36d917b734a468e9cac76c0b496e8b2/2021-09-08_08-24.png)

    ```bash
    # create a file with req. permissions using file
    ansible linux -m file -a 'path=/tmp/test_modules.txt state=touch mode=600' -o

    # using fetch
    ansible linux -m fetch -a 'src=/tmp/test_modules.txt dest=/tmp/test_module_files'

    # The final file dest will on local 'ubuntu-c'
    /tmp/test_module_files/centos1/tmp/test_modules.txt
    ```

It will create a directory for each of the hostnames,

### 📃 Ansible-doc

- **Online documentation**

All the modules and there use cases documented here [https://docs.ansible.com/ansible/latest/collections/ansible/](https://docs.ansible.com/ansible/latest/collections/ansible/)

- Inbuilt documentation with `ansible-doc`, (manual pages for ansible)

    ```bash
    ansible-doc file
    ansible-doc fetch
    ```

    How a module works behind the scene, with source code locations etc..

## 🇬🇾 `YAML` - The Playbook Language

- `YAML` is a data-oriented language,

**Contents**

1. Structure
2. Indentation
3. Quotes
4. Multi line Values
5. Boolean (True/False)
6. Lists and Dictionaries

Ansible Playbooks are the native way of running logic as scripts in ansible, `yaml` is the language utilised in ansible playbooks (YAML ain't markup language), (JSON is also used in playbooks)

- It is a human readable, data-serialisation language
- Easy to use, easy to read, great for collaboration.
- Read and write in YAML is supported by most programming language
- Yaml files comes with `.yml` or `.yaml` extension, `.yaml` is the recommended extension.

### 🕵️ Python Interprets YAML

Files written in `yaml`, can be interpreted in a pythonic way(as you guessed in a python dictionary) , also this is how ansible looks `yaml` as ansible written in python,
Can use a one liner script for that

```bash
python3 -c 'import yaml,pprint;pprint.pprint(yaml.load(open("test.yaml").read(), Loader=yaml.FullLoader))'

# yaml - python yaml module
# pprint - Pretty Print
```

This will read the contents of the `test.yaml`, and pretty prints the output,

`test.yaml`

```yaml
# Every YAML file should start with three dashes
---


# Every YAML file should end with three dots
...
```

Python will interpret this as a `None` Type, (nothing in the file , only comments)

`test.yaml` with contents

```yaml
# Every YAML file should start with three dashes
---

no_quotes: newline chrtr\n
single_quotes: 'newline chrtr\n'
# interpreted as {'new_line2': 'newline chrtr\\n'}
# Extra escaping is provided as python interprets this as a part of string
double_quotes: "newline chrtr\n"
# interpreted as {'new_line1': 'newline chrtr\n'}
# correctly formated as intended

# multiline strings with "|" character - interpret as multiline
example_key_1: |
 this is an
 example of
 multiline string

# {'example_key_1': 'this is an \nexample of\nmultiline string\n'}

# multiline strings with ">" character - interpret as singleline
example_key_1: >-
 this is an
 example of
 multiline string

# {'example_key_1': 'this is an  example of multiline string'}
# extra "-" is added with ">" to remove the end '\n' charater that will occur otherwise

# Integers treated directly
example_integer: 1 # As integer {'example_integer': 1}
example_integer: "1" # As String {'example_integer': "1"}

# versatile boolean handling
bool : False, false, no, No, NO, off, OFF, n ..
bool : True, true, yes, Yes, on, ON, y

# Every YAML file should end with three dots
...
```

- List/Dictionary of items in `yaml`, using `-` character,

```yaml
---
- item 1
- item 2
- item 3

# interpreted as [item1, item2, ...]

example_1: value_1
example_2: value_2

# {'example_1': 'value_1', 'example_2': 'value_2'}

# or explicitly specify as a dictionary or list
{example_1: value_1, example_2: value_2}
[item_1, item_2 ...]
...
```

- Key-value and list block in-line is Invalid

```yaml
---
example_1: value_1
- list_entry_1
...
```

- Indentation in Yaml - 2 character indentation - commonly used

```yaml
---
example_1:
  sub_example_1: value_1

example_2:
  sub_example_2: value_2

# This is now a python dict. of dict
# {'example_1': {'sub_example_1': 'value_1'},
# 'example_2': {'sub_example_2': 'value_2'}}

# Dict with list of values in yaml
example_1:
  - item_1
  - item_2
  - item_3

example_2:
  - item_4
  - item_5
  - item_6

# {'example_1': ['item_1', 'item_2', 'item_3'],
# 'example_2': ['item_4', 'item_5', 'item_6']}
...
```

- Concat these data structures to get the desired level of interpretation

```yaml
---
example_dictionary_1:
  - example_dictionary_2:
    - 1
    - 2
    - 3
  - example_dictionary_2:
    - 4
    - 5
    - 6
  - example_dictionary_4:
    - 7
    - 8
    - 9

# python interprets
# {'example_dictionary_1': [{'example_dictionary_2': [1, 2, 3]},
#                           {'example_dictionary_2': [4, 5, 6]},
#                           {'example_dictionary_4': [7, 8, 9]}]}
...
```

## ▶️ Ansible PlayBook - Introduction

### 🃏 Ansible Playbooks, Breakdown of Sections

- Understanding common sections
- Create a message of the day - playbook sample
- Use variables and override them from the command line.
- Use Handlers for post task execution,
- Update playbook to target both CentOS and Ubuntu, with when directive

### 🐾 Playbooks Introduction

The playbook contains a list of plays, with each play being a dictionary

Major Sections in a Playbook,

- **Hosts**: where our play will run and options it will run with
- **Vars**: variables that will apply to the play, on all target systems
- **Tasks**: the list of tasks that will be executed within the play, this section, can also be used for pre and post tasks
- **Handlers**: the list of handlers that are executed as a notify key from a task
- **Roles**: list of roles to be imported into the play

List of plays can be included in a single playbook.

### 🍾 Message of the day - Playbook Example

The aim is to deliver a message to all hosts,

- Have an `ansible.cfg` file (where inventory and options listed)
- A file contains message to deliver,
- `hosts` file, (from the old hosts file built)
- Finally the `mot_playbook.yaml` file, that contains

    ```yaml
    ---
    -
      # Hosts: where our play will run and options it will run with
      hosts: centos
      user: root

      # Vars: variables that will apply to the play, on all target systems

      # Tasks: the list of tasks that will be executed within the playbook
      tasks:
        - name: Configure a MOTD (message of the day)
          copy:
            src: centos_motd
            dest: /etc/motd

      # Handlers: the list of handlers that are executed as a notify key from a task

      # Roles: list of roles to be imported into the play
    ...
    ```

    We've got one play, for host group `centos` user `root`, assign the task with a friendly name, using the copy module with `src` and `dest` given as key-values, (at the time of execution these dictionary gets passed in as an argument to the copy module)

Execute the playbook using `ansible-playbook` command, with the yaml file specified,

```bash
ansible-playbook motd_playbook.yaml
```

Now it is run the colours indicates whether changes being made/not, along with the `copy` module specified the `settings` module will also run with the playbook to gather facts

indicated in `ok=2,`

log in to centos childs using [web interface](http://localhost:1000/), when login to centos the message of the day can be seen,

- To get how much time the playbook execution takes

    ```bash
    time ansible-playbook motd_playbook.yaml

    PLAY RECAP ******************************************************

    centos1 : ok=2    changed=0    unreachable=0    failed=0    skipped=0
    centos2 : ok=2    changed=0    unreachable=0    failed=0    skipped=0
    centos3 : ok=2    changed=0    unreachable=0    failed=0    skipped=0

    real    0m4.314s # Time took
    user    0m2.259s
    sys     0m0.834s
    ```

- The `facts` gather can be turned off in playbook - file,

    ```yaml
    hosts: centos
    user: root
    gather_facts: False

    real    0m2.396s
    # The execution time now gets halved
    ```

### 🧛🏾 Variables in Playbook

- The message of the day not always need to be in a file, it can be passed directly or saved and passed with a variable assigned

    ```yaml
    # Vars: variables that will apply to the play, on all target systems
    vars:
      motd: "Welcome to CentOS Linux - Ansible is Good\n"

    # Tasks: the list of tasks that will be executed within the playbook
    tasks:
      - name: Configure a MOTD (message of the day)
        copy:
          content: "{{ motd }}"
          dest: /etc/motd
    ```

    The defined variable expressed using `{{<variable>}}` , these are part of `jinja` template syntax used. (jinja2 templating system will be discussed later)

- Variable can be passed to ansible via command line,

    ```bash
    ansible-playbook motd_playbook.yaml -e 'motd="Testing playbook\n"'
    ```

### 🦾 Handlers in Playbook

- Act as a notify key from tasks

    ```yaml
    handlers:
      - name: MOTD changed
        debug:
          msg: The MOTD was changed
    ```

    Handlers are only executed one after the tasks when there is a change.

    Here the `debug` module is used, which helps to add debug context to the playbook.

    ```bash
    RUNNING HANDLER [MOTD changed] *****
    ok: [centos1] => {
        "msg": "The MOTD was changed"
    }
    ...
    ```

  - The ok tasks `ok=2`, one for the change and one for the `handler`
  - If rerun again, there is no change so the handler doesn't gets notified,

### 🧇 Decision Making in playbooks (`when`)

- To control the execution of tasks, state `when` a task need to be run, it can be based to logic or gathered facts
- Here going the `motd` change targeted only to `ubuntu` hosts, to check if a host is using ubuntu/centos, using help of the facts gathered by `facts` module,
- Take a look at the facts available on the centos system,

    ```bash
    ansible all -i centos2, -m setup | more
    # centos2, -> Requires a list of hosts comma seperated
    ```

- There is a lot of facts, find a fact that can be used to distinguish between `ubuntu` and `centos` hosts, there is one, verify it by running on ubuntu system

    ```bash
    "ansible_distribution": "CentOS"

    # on ubuntu
    ansible all -i centos2,ubuntu2 -m setup | grep ansible_distribution

    "ansible_distribution": "Ubuntu"
    ```

- Now each of the system can be given different motd, based on this

    ```yaml
    ---
    -
      hosts: linux

      vars:
        motd_centos: "Welcome to CentOS Linux - Ansible Rocks\n"
        motd_ubuntu: "Welcome to Ubuntu Linux - Ansible Rocks\n"

      tasks:
        - name: Configure a MOTD (message of the day)
          copy:
            content: "{{ motd_centos }}"
            dest: /etc/motd
          notify: MOTD changed
          when: ansible_distribution == "CentOS"

        - name: Configure a MOTD (message of the day)
          copy:
            content: "{{ motd_ubuntu }}"
            dest: /etc/motd
          notify: MOTD changed
          when: ansible_distribution == "Ubuntu"

      handlers:
        - name: MOTD changed
          debug:
            msg: The MOTD was changed
    ...
    ```

    The `when: ansible_distribution == "CentOS"`, does the task only when the condition met, also the hosts is now `Linux` group.

    ![when-playbook-run.png](Ansible%20Modules%20and%20Playbooks%20f36d917b734a468e9cac76c0b496e8b2/when-playbook-run.png)

### 📈 Challenge: Playbook

- **Step1**:- Setup the challenge using the `ansible.cfg, hosts` from the previous challenge, and `playbook.yaml` from first, need to target only the `ubuntu` hosts for this.
- **Step2**:- Add a handler to debug if there is any change
- **Step3**:- Create a task, that copies the file in the current directory, `60-ansible-motd`, to `/etc/update-motd.d/60-ansible-motd` , of all hosts,
- **Step4**:- Use the option `mode`,  for copy to set the permission to preserve (preserve file permissions), Add a notification option to the task to inform the changes.
- Answer

    ```yaml
    ---
    -
      hosts: ubuntu

      tasks:
        - name: Copy The Message of the Day File to /etc/update-motd.d/
          copy:
            src: 60-ansible-motd
            dest: /etc/update-motd.d/60-ansible-motd
            mode: preserve
          notify: Debug Handler

      handlers:
        - name: Debug Handler
          debug:
            msg: The MOTD Content Changed
    ...
    ```

- **Extra Step**:- Delete the copied file in the hosts using Ansible command Line
- Answer

    ```bash
    ansible ubuntu -m file -a 'path=/etc/update-motd.d/60-ansible-motd state=absent' -o
    ```

## 👩‍✈️Ansible Playbooks, Variables

- Simple yaml variable,

    ```yaml
    # variable
    vars:
     example_key: example_value

    # Display
    tasks:
     - name: Test dictionary key value
      debug:
       msg: "{{ example_key }}"

    # dictonary as variable and accessing values
    vars:
     dict:
      dict_key: dictionary_value

    # Whole dictionary
    msg: "{{ dict }}"

    # dict value with "." notation or python dict notation
    msg: "{{ dict.dict_key }}"
    msg: "{{ dict['dict_key'] }}"

    # in a similar way yaml lists can be accessed
    msg: "{{ var_list.0 }}"
    msg: "{{ var_list[0] }}" # Accessing the first item
    ```

- Variables in a file, here variables are mentioned in another `yaml` file,

    ```yaml
    vars_files:
     - external_vars.yaml # containes all the variables mentioned above
    ```

    The variables can be accessed in a similar way ,

- Prompt the user to input a variable such as username and password etc use `vars_prompt`,

    ```yaml
    # For username
    vars_prompt:
     - name: username
      private: False

    # Retrive
    msg: "{{ username }}"

    # For password
    vars_prompt:
     - name: password
      private: True

    # with private-> True message typin cannot be shown
    ```

- Accessing `hostvars`,

    Use `hostvars` to get ansible_port,

    ```yaml
    msg: "{{ hostvars[ansible_hostname]['ansible_port'] }}"
    ```

    If any hosts have not set the accessed `hostvars`, it will give an error, as the variables don't exists,

    We can use a `jinja2` filter with a default value to bypass the error,

    ```yaml
    msg: "{{ hostvars[ansible_hostname]['ansible_port'] | default('22') }}"

    # So if the variable not present, it will defaults to port 22
    ```

- Accessing `groupvars`,

    we've set the `ansible_user` group vars, to access it using ansible

    ```yaml
    msg: "{{ ansible_user }}"

    # result For centos
    ok: [centos1] => {
        "msg": "root"
    }
    ```

    For ubuntu hosts there is no `groupvars` set,  so it will give error

    Also, ansible automatically assigns `groupvars` to `hostvars` section, so a `groupvar` can be accessed from there too,

    ```yaml
    msg: "{{ hostvars[ansible_hostname]['ansible_user'] }}"
    ```

- Passing Extra variables when executing a playbook
  - using `ini` format,

        ```bash
        ansible-playbook variables_playbook.yaml -e extra_vars_key="extra vars value"
        ```

  - In `json` format

        ```bash
        ansible-playbook variables_playbook.yaml -e {"extra_vars_key": "extra vars value"}
        ```

  - in `yaml` format

        ```bash
        ansible-playbook variables_playbook.yaml -e {extra_vars_key: extra vars value}
        ```

  - Passing an entire file of extra variables, also as json/or ini files

        ```bash
        ansible-playbook variables_playbook.yaml -e @extra_vars_file.yaml
        ```

## 🏭 Ansible Playbooks, Facts

The `setup` module runs automatically at every playbook executions that brings up the facts,

- Filters to get specific facts
- Creation and execution of custom facts (satisfy requirements that comes extra)

### 🏵️ The `setup` module, and `ansible_facts`

- Ansible doc for [setup-module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/setup_module.html). (gather all information about connected hosts)
- From docs `gather_subset` with `setup` module, If supplied, restrict the additional facts collected to the given subset. Possible values: `all, min, hardware, network, virtual, ohai, and facter`.

    ```bash
    ansible centos1 -m setup -a 'gather_subset=network' | more

    # This still includes a lot of info(although network subset used)
    # By default a min selection and all selecton included, exclude them

    ansible centos1 -m setup -a 'gather_subset=!all,!min,network' | more

    # can check the decrese in no. of results returned using `wc -l` instead of more
    ansible centos1 -m setup -a 'gather_subset=!all,!min,network' | wc -l
    369 --> 218
    ```

- Filtering Output with wildcard, the output can be filtered further with heads/wildcards,

    ```bash
    ansible centos1 -m setup -a 'filter=ansible_memory_mb'
    ansible centos1 -m setup -a 'filter=ansible_mem*'
    ```

- The `ansible_facts`, ***Ansible idiom,***

    Any module returning a dictionary of `ansible_facts`, is get added to the root of the facts namespace, so `ansible_facts` is the all in one keeper of all facts.

    With a playbook,

    ```yaml
    ---
    -
      hosts: all
      tasks:
        - name: Show IP Address
          debug:
            msg: "{{ ansible_default_ipv4.address }}"
    ...
    ```

    Here there is no mention of `ansible_facts`, but direct mention of 2nd and 3rd level,

**Custom Facts**

Although there is enough default system info, there is always need - own user defined facts of custom facts, that will be gathered from the hosts with gather facts process,

- Any scripts that returns a JSON, INI or any structure, can be a custom facts.
- By default ansible looks for custom fact in `/etc/ansible/facts.d`, (which is ok if running as root user, but that's not most cases,)

-

For eg, a simple `bash` script that gives date o/p in `json` format/ `ini` format,

```bash
#!/bin/bash
echo {\""date\"" : \""`date`\""}

# output
{"date" : "Fri Nov 26 16:56:28 UTC 2021"}
```

```bash
#!/bin/bash
echo [date]
echo date=`date`

# out,
[date] # A category is must required
date=Fri Nov 26 16:57:48 UTC 2021
```

create `/etc/ansible/facts.d` and copy the files to,

```bash
sudo mkdir -p /etc/ansible/facts.d
sudo cp * /etc/ansible/facts.d/

ansible ubuntu-c -m setup -a 'filter=ansible_local' | more

# out
"ansible_facts": {
 "ansible_local": {
            "getdate1": {
                "date": "Fri Nov 26 17:06:11 UTC 2021"
            },
            "getdate2": {
                "date": {
                    "date": "Fri Nov 26 17:06:11 UTC 2021"
                }
```

In playbook, it can be accessed like,

```bash
msg: "{{ ansible_local.getdate1.date }}"
# or via hostvars
msg: "{{ hostvars[ansible_hostname].ansible_local.getdate1.date }}
```

To copy the facts to host servers,

```yaml
tasks:
    - name: Make Facts Dir
      file:
        path: /etc/ansible/facts.d
        recurse: yes
        state: directory

    - name: Copy Fact 1
      copy:
        src: /etc/ansible/facts.d/getdate1.fact
        dest: /etc/ansible/facts.d/getdate1.fact
        mode: 0755

    - name: Refresh Facts
      setup:

    - name: Show IP Address
      debug:
        msg: "{{ ansible_default_ipv4.address }}"

    - name: Show Custom Fact 1
      debug:
        msg: "{{ ansible_local.getdate1.date }}"
```

- To remove the copied facts, and use directory other than `/etc/ansible/facts.d`, for this,

```bash
# remove the files
ansible linux -m file -a 'path=/etc/ansible/facts.d/getdate1.fact state=absent'
```

- Copy using a playbook the facts exists in local `fact.d` folder in control host, then reload the setup module to load facts without su privilages, or root path

```yaml
tasks:
    - name: Make Facts Dir
      file:
        path: /home/ansible/facts.d
        recurse: yes
        state: directory
        owner: ansible

    - name: Copy Fact 1
      copy:
        src: facts.d/getdate1.fact
        dest: /home/ansible/facts.d/getdate1.fact
        owner: ansible
        mode: 0755

  # When reloading facts implicitly specify the location for the setup module to look for
  - name: Reload Facts
      setup:
        fact_path: /home/ansible/facts.d
```

- Remove the created junks from all hosts,

```yaml
ansible linux -m file -a 'path=/home/ansible/facts.d state=absent'
```
