# 👩🏾‍🎓 Ansible Playbooks - Deep Dive <!-- omit in toc -->

## 🥡 Table of Contents <!-- omit in toc -->

- [🍿Ansible Playbook Modules](#ansible-playbook-modules)
  - [🏗️ Ansible Inbuilt & Most commonly Used modules](#️-ansible-inbuilt--most-commonly-used-modules)
- [🧨 Dynamic Inventories](#-dynamic-inventories)
  - [🔑 Key Requirement of a dynamic inventory](#-key-requirement-of-a-dynamic-inventory)
  - [🧺 base inventory template in python](#-base-inventory-template-in-python)
- [🤰🏾 Register and When](#-register-and-when)
- [➿ Looping With Ansible](#-looping-with-ansible)
  - [🏇🏻 Asynchronous, Serial and Parallel operations in ansible](#-asynchronous-serial-and-parallel-operations-in-ansible)
- [📝 Task Delegation](#-task-delegation)
  - [🇦🇸 Sample Task to be done](#-sample-task-to-be-done)
  - [🧍‍♀️ Task Delegation to host](#️-task-delegation-to-host)
- [🧙‍♂️ Magic Variables](#️-magic-variables)
- [🧱 Blocks](#-blocks)
  - [⚠️ Error Handling for `blocks`](#️-error-handling-for-blocks)
- [🔏 Ansible Vault](#-ansible-vault)
  - [🚂 Encrypt string with ansible vault](#-encrypt-string-with-ansible-vault)
  - [🔐 Encrypt file with ansible vault](#-encrypt-file-with-ansible-vault)
  - [🔓 Decrypt vault encrypted file](#-decrypt-vault-encrypted-file)

## 🍿Ansible Playbook Modules

Topics Covered,

- Ansible Playbook modules with examples,
- Dynamic Inventories - Creation and Using them
- Registering coming Information and make use of It,
- Using `when` conditional
- Various Looping methods
- Playbook Performance -  Asynchronous, Serial and Parallel execution
- Task Delegation - Delegating tasks to specific targets
- Magic Variables - Special Set of beneficial variables.
- `Blocks` - Structure Tasks to blocks
- Ansible Vault - For securing Information.

### 🏗️ Ansible Inbuilt & Most commonly Used modules

- `set_fact`
- `prompt`
- `assemble`
- `fetch`

- `pause`
- `group_by`
- `wait_for`
- `add_host`

1. `set_fact` : Dynamically add/change facts during playbook execution

    ```yaml
    ---
    - name: Common Ansible modules Learn

      # Hosts: where our play will run and options it will run with
      hosts: ubuntu3,centos3

      tasks:
        - name: Set a fact
          set_fact:
            our_fact: Ansible Rocks!

        - name: Show custom fact
          debug:
            msg: '{{ our_fact }}'
    ```

    - Rewriting an Existing Fact,

        ```yaml
        # Rewriting an Existing Fact
            - name: Rewrite an Existing Fact
              set_fact:
                ansible_distribution: '{{ ansible_distribution | upper }}'

            # Expected: UBUNTU & CENTOS
            - name: Show Modified Existing Fact
              debug:
                msg: '{{ ansible_distribution }}'
        ```

2. The Commonly Used:- `when` and `set_fact` combination

    Which is a commonly used method to set common variables based on certain when conditions first then use this common variables at the later stages of the playbook.

    That can remove certain inventory variable dependencies,

    ```yaml
    tasks:
        - name: Set our installation variables for CentOS
          set_fact:
            webserver_application_port: 80
            webserver_application_path: /usr/share/nginx/html
            webserver_application_user: root
          when: ansible_distribution == 'CentOS'

        - name: Set our installation variables for Ubuntu
          set_fact:
            webserver_application_port: 8080
            webserver_application_path: /var/www/html
            webserver_application_user: nginx
          when: ansible_distribution == 'Ubuntu'

        - name: Show pre-set distribution based facts
          debug:
            msg: "webserver_application_port:{{ webserver_application_port }} webserver_application_path:{{ webserver_application_path }} webserver_application_user:{{ webserver_application_user }}"
    ```

3. The `pause` Module: Pause a playbook for a set amount of time or until a prompt is acknowledged.

    ![The Prompt](%F0%9F%91%A9%F0%9F%8F%BE%E2%80%8D%F0%9F%8E%93%20Ansible%20Playbooks%20-%20Deep%20Dive%202f3e15f8b9ff461386de1359c45f1fb2/Untitled.png)

    The Prompt

    ```yaml
    - name: The Pause Module

      hosts: ubuntu3,centos3

      tasks:
        - name: Pause our playbook for 10 seconds
          pause:
            seconds: 10

      # Pause with Prompt
        - name: Pause with Prompt User to Verify before continue
          pause:
            prompt: Please check that the webserver is running, press enter to continue
    ```

4. The `wait_for` module

    The task with `wait_for` in it waits until the described condition in it gets fulfilled, then only it goes to the next step.

    To check that first lets,

    1. initialize an nginx server using the similer steps as previous(`setip_webserver.yml`),
    2. Then stop the web-server on a host manually,

        ```bash
        ansible centos3 -m service -a "name=nginx state=stopped"
        ```

    3. Run the playbook containing `wait_for` - web-server to come alive as a background task,

        ```yaml
        tasks:
            - name: Wait for the webserver to be running on port 80
              wait_for:
                port: 80
        ```

        Run the playbook as background task using `&`,

        ```bash
        ansible-playbook wait_for_playbook.yaml &
        ```

    4. Finally start the web-server which is manually stopped, and verify the background running playbook now completes its execution.

        ```bash
        ansible centos3 -m service -a "name=nginx state=started"

        # Now the background executing playbook resumes its runing.
        ```

5. The `Assemble` module: Assemble configuration files from fragments,

    when an application/tool requires its configuration as a single file, but you wish to manage it as separate entities,

    Here we are dividing the conf files as host specific,

    in `mkdir conf.d` we got 2 separate conf files for `centos` and a `default` file,

```bash
## Custom for centos1
Host centos1
  User root
  Port 2222
```

```bash
# default
Port 22
Protocol 2
ForwardX11 yes
GSSAPIAuthentication no
```

In the final step we are assembling the contents in `conf.d` to `sshd_config`, `src` to `dest`,

```bash
- name: Ansible Assemble
  hosts: ubuntu-c
  tasks:
    - name: Assemble conf.d to sshd_config
      assemble:
        src: conf.d
        dest: sshd_config
```

After running the playbook a new file `sshd_config` created in playbook directory with

- contents,

    ```bash
    ## Custom for centos1
    Host centos1
      User root
      Port 2222

    ## Defaults

    Port 22
    Protocol 2
    ForwardX11 yes
    GSSAPIAuthentication no
    ```

To SSH connect to a host with that `sshd_config`,

```bash
ssh -F sshd_config centos1
```

So each hosts can be managed separately by source control, but in ansible it  can be then combined to one

1. Ansible `add_hosts` module: Dynamically add target hosts to running playbooks

    This is useful for playbooks with multiple plays,

    - Sample play book

        ```yaml
        ---
        # Play 1
        - name: Ansible modules - assemble and add_hosts
          hosts: ubuntu-c

          tasks:
            # Add exising hosts to groups
            - name: Add centos1 to adhoc_group
              add_host:
                name: centos1
                groups: adhoc_group1, adhoc_group2

        - # Play 2 with different host
          hosts: adhoc_group1

          tasks:
            - name: Ping all in adhoc_group1
              ping:
        ```

2. `group_by` module: Create groups based on facts
    - Create a group based on ansible_distribution

        ```yaml
        ---
        - hosts: all

          tasks:
            # Task to create a group based on ansible_distribution custom_centos/custom_ubuntu
            - name: Create group based on ansible_distribution
              group_by:
                key: 'custom_{{ ansible_distribution | lower }}'

        - hosts: custom_centos
          # By This way we've isolated all centos hosts to one group
          tasks:
            - name: Ping all in custom_centos
              ping:
        ```

    We have created `custom_centos` group and added hosts based on the fact condition `key`, then used this group in next play

3. Ansible `fetch` module: Capture file from hosts
    - Fetch centos release_version file

        ```yaml
        - name: Using Ansible fetch modules
          hosts: centos

          tasks:
            # To fetch centos redhat release version file(src) and put in dest.
            - name: Fetch /etc/redhat-release
              fetch:
                src: /etc/redhat-release
                dest: /tmp/redhat-release
        ```

## 🧨 Dynamic Inventories

The normal inventories are defined in `ansible.cfg` or specified implicitly, which contains a list of hosts,

The variables can be set on `hosts_vars` and `group_vars` and can be override with `-i` flag,

The dynamic inventory is where the inventory file is an executable one, then ansible executes that file and takes its output as inventory (`json` format).

### 🔑 Key Requirement of a dynamic inventory

- It needs to be executable and can be run from command line
- Accepts command line options of `--list` and `--host <hostname>`
- Returns a `json` encoded dictionary of inventory content when used with `--list` or with `host <hostname>`.

### 🧺 base inventory template in python

[inventory.py](%F0%9F%91%A9%F0%9F%8F%BE%E2%80%8D%F0%9F%8E%93%20Ansible%20Playbooks%20-%20Deep%20Dive%202f3e15f8b9ff461386de1359c45f1fb2/inventory.py)

- run `./inventory.py` to get the usage help.

    ```bash
    usage: inventory.py [-h] [--list] [--host HOST]
    ```

- `./inventory.py --list` gives hosts list in a json formatted content of groups and hosts, with corresponding `groupvars` and the `children` specified.
- `./inventory.py --host centos1` gives hostvars for centos1.
- To use it with ansible to show the available hosts in the inventory,

    ```bash
    # usage of dynamic inventories with ansible
    ansible all -i inventory.py --list-hosts
    # That will give the hosts list same as a normal inventory as source

    # ping all hosts
    ansible all -i inventory.py -m ping -o
    ```

- script uses python argparse module to parse command line args, debugger is used as direct printing to stdout will affect the pure json input required for ansible.
- By looking athe deug logs in `/var/tmp/ansible_dynamic_inventory.log`,
    1. Ansible called the script with `--list` and the script gives the list of hosts.
    2. Then for each hosts returned ansible ran the `--host` command with  together with hostvars, this execution in each hosts takes a considerable amount of time.
    3. That is for a 1000 hosts this will slow down the process a lot, that can be verified by creating fake hosts,

        ```bash
        for i in {1..1000}
        do
        echo \'fake${i}\'\,
        # transforms \n to space charactors
        done | tr "\n" " "
        ```

        Copy the output and create a new hosts in `define_inventory` as `"fake": { "hosts": [ <copied_content>] }`, now the hosts modified,

        run the list-hosts with time prefix,

        ```bash
        time ansible all -i inventory.py --list-hosts
        # took around 1 min
        ```

- To make the it more efficient, the option `Inventory(include_hostvars_in_list=True)` set True, that makes the `hostvar` information as part of list output, under a separate key called `_meta`,

    So now ansible didn’t  each host with `—hosts` to get host var, it takes that from `--list` output that saves a considerable amount of time. This enhancement was added in ansible 1.3

Remove the background running tail logs job,

```bash
# Run jobs command
jobs

# kill the 1st job
kill %1
```

For design and considerations of dynamic inventory `aws` dynamic inventory is a best resource,

## 🤰🏾 Register and When

By using a combination of register and when, the amount of logic applicable are countless,

- `register` directive is used to register the `stdout` values & parameters to a variable, filters can be used to manipulate that
- `when` helps to compare the registered output with known facts/values.

Thereby dynamic conditions can be built up using this combination to execute things.

```bash
# to get the hostname
ansible all -a 'hostname -s' -o
```

Simple **use of `register`,**

```yaml
-
  hosts: linux
  tasks:
    - name: Exploring register
      command: hostname -s
   # register a varaible to get the content from executed output
      register: hostname_output
```

Showing off registered data using `debug` module,

```yaml
-
  hosts: linux
  tasks:
    - name: Exploring register
      command: hostname -s
      register: hostname_output

    - name: Show hostname_output
      debug:
        var: hostname_output
```

Sample output

```yaml
"hostname_output": {
        "changed": true,
        "cmd": [
            "hostname",
            "-s"
        ],
        "delta": "0:00:00.035281",
        "end": "2022-05-30 18:22:15.326057",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2022-05-30 18:22:15.290776",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "ubuntu3",
        "stdout_lines": [
            "ubuntu3"
        ]
    }
```

- `rc` - return code

Can use the dot notation to access the inner values of dictionary/json.

```yaml
- name: Show hostname_output
      debug:
        var: hostname_output.stdout # gives the whole stdout
```

Using **`when`,**

```yaml
-
  hosts: linux
  tasks:
    - name: Exploring register
      command: hostname -s
      when: ansible_distribution == "CentOS" and ansible_distribution_major_version == "8"
```

Setup a filter considering `ubuntu`,

```yaml
ansible ubuntu1 -m setup -a filter='ansible_distribution*'
```

So using when with `or`,

```yaml
-
  hosts: linux
  tasks:
    - name: Exploring register
      command: hostname -s
      when: ( ansible_distribution == "CentOS" and ansible_distribution_major_version | int >= "8" ) or
            ( ansible_distribution == "Ubuntu" and ansible_distribution_major_version | int >= "20" )
```

- `int` is a jinja2 filter to convert o/p to `int` value,

Providing a `list` to when automatically converts it to an `and` condition,

```yaml
-
  hosts: linux
  tasks:
    - name: Exploring register
      command: hostname -s
      when:
        - ansible_distribution == "CentOS"
        - ansible_distribution_major_version | int >= 8
```

Finally using register with when,

```yaml
-
  hosts: linux
  tasks:
    - name: Exploring register
      command: hostname -s
      when:
        - ansible_distribution == "CentOS"
        - ansible_distribution_major_version | int >= 8
      register: command_register

  # `changed` bool variable always gives true for matched ones
  # can ensure only centos ones are gonna executed
    - name: Install patch when changed
      yum:
        name: patch
        state: present
      when: command_register.changed

# similar logic for changed and skipped
    - name: Install patch when changed
      yum:
        name: patch
        state: present
      when: command_register is changed

    - name: Install patch when skipped
      yum:
        name: patch
        state: present
      when: command_register is skipped
```

## ➿ Looping With Ansible

Simple example of a loop with specified List,

```yaml
tasks:
    - name: Configure a MOTD (message of the day)
      copy:
        content: "Welcome to {{ item }} Linux - Ansible Rocks!\n"
        dest: /etc/motd
      notify: MOTD changed
      with_items: [ 'CentOS', 'Ubuntu' ]
      when: ansible_distribution == item

  # Handlers: the list of handlers that are executed as a notify key from a task
  handlers:
    - name: MOTD changed
      debug:
        msg: The MOTD was changed
```

- The `{{ item }}` gets iterated with the values specified in `with_items`,
- The total loop runs 12 times, which is 2 x all linux groups,

Another way of execution,

```yaml
  tasks:
    - name: Configure a MOTD (message of the day)
      copy:
        content: "Welcome to {{ item }} Linux - Ansible Rocks!\n"
        dest: /etc/motd
      notify: MOTD changed
      with_items:
        - CentOS
        - Ubuntu
      when: ansible_distribution == item
```

The `with_items`, can be really useful for repeating tasks like creating users,

```yaml
  tasks:
    - name: Creating user
      user:
        name: "{{ item }}"
      with_items:
        - james
        - hayley
        - lily
        - anwen
```

Now the users added for each of the hosts, now to remove them change `state` to absend,

```yaml
tasks:
    - name: Removing user
      user:
        name: "{{ item }}"
        state: absent
      with_items:
        - james
        - hayley
        - lily
        - anwen
```

Can iterate through dictionary using `with_dict`,

```yaml
  tasks:
    - name: Creating user
      user:
        name: "{{ item.key }}"
        comment: "{{ item.value.full_name }}"
      with_dict:
        james:
          full_name: James Spurin
        hayley:
          full_name: Hayley Spurin
        lily:
          full_name: Lily Spurin
        anwen:
          full_name: Anwen Spurina
```

Also remove the users with `state: absend`,

Using `with_subelements`,

```yaml
  tasks:
    - name: Creating user
      user:
        name: "{{ item.1 }}"
        comment: "{{ item.1 | title }} {{ item.0.surname }}"
      with_subelements:
        - family:
            surname: Spurin
            members:
             - james
             - hayley
             - lily
             - anwen
      surname: Darlington
      members:
       - freya
        - members
```

The elements and sub-elements can be a list of dictionary, or dict within dict,

`item.0` → represent main path of content, there it captures `surname` in first iteration

The next iteration will be what inside `members`, for each members inside,

The `jinja2` template filter `title` makes the first letter uppercase

All the users are created but none of them got `passwd` set up, verify that by checking up `/etc/shadow` file

```yaml
ssh root@centos3 tail -8 /etc/shadow

# The !! shows password not set
```

To overcome this and to set passwords, for that use ansible inbuilt functionalities,

```yaml
  tasks:
    - name: Creating user
      user:
        name: "{{ item.1 }}"
        comment: "{{ item.1 | title }} {{ item.0.surname }}"
        # https://docs.ansible.com/ansible/latest/plugins/lookup/password.html
        password: "{{ lookup('password', '/dev/null length=15 chars=ascii_letters,digits,hexdigits,punctuation') | password_hash('sha512') }}"
```

Here it is using the `lookup` plugin that can pick and set passwords,

This is highly secure passwords, we are passing to `/dev/null` as we do not intend to store them,

Set common directories for users,

```yaml
  - name: Creating user directories
      file:
        dest: "/home/{{ item.0 }}/{{ item.1 }}"
        owner: "{{ item.0 }}"
        group: "{{ item.0 }}"
        state: directory
      with_nested:
        - [ james, hayley, freya, lily, anwen, ana, abhishek, sara ]
        - [ photos, movies, documents ]
```

The `with_nested` just do like for each of these do each of these, that is now it got 24 (3 x 8) iterations so `[ photos, movies, documents ]` are going to be created for every users home directory: `/home/<user>`.

The `with_together` groups the same index items from two given lists,

```yaml
  tasks:
    - name: Creating user directories
      file:
        dest: "/home/{{ item.0 }}/{{ item.1 }}"
        owner: "{{ item.0 }}"
        group: "{{ item.0 }}"
        state: directory
      with_together:
        - [ james, hayley, freya, lily, anwen, ana, abhishek, sara ]
        - [ tech, psychology, acting, dancing, playing, japanese, coffee, music ]
```

Use `with_file` , to copy the contents of file to a variable and use it eg,

```yaml
  tasks:
    - name: Create authorized key
      authorized_key:
        user: james
        key: "{{ item }}"
      with_file:
        - /home/ansible/.ssh/id_rsa.pub

# Can give multiple items as with_file
      with_file:
        - /home/ansible/.ssh/id_rsa.pub
    - custom_key.pub
```

It essentially adds the contents of `id_rsa.pub` file to the `.ssh/authorized_keys` file of destination, the content of the file is passed as the variable `{{ item }}`,

To create a custom key file at the current location,

```yaml
ssh-keygen -f custom_key

# Connect with custom keys
ssh -i custom_key centos3 -l james
```

Ansible got another loop that mimics the `for` loop functioning,

```yaml
 tasks:
    - name: Create sequence directories
      file:
        dest: "/home/james/sequence_{{ item }}"
        state: directory
      with_sequence: start=0 end=100 stride=10

# similar result
   file:
        dest: "/home/james/sequence_{{ item }}"
        state: directory
      with_sequence: start=0 end=100 stride=10 format=/home/james/sequence_%d

# Or with hex, octan
      with_sequence: start=0 end=16 stride=1 format=/home/james/hex_sequence_%x
# Hex values start with 0 and end with F
```

That includes key-value pairs of start, stop and step

list directories

```yaml
ssh centos3 -l root ls -altrh /home/james
```

Using `with_sequence` to count,

```yaml
with_sequence: count=5 format=/home/james/count_sequence_%x
```

There is also more operations like `with_random_choice`,

```yaml
      with_random_choice:
        - "google"
        - "facebook"
        - "microsoft"
        - "apple"
```

**The `until` Loop : Useful Loop**

```yaml
 tasks:
    - name: Run a script until we hit 10
      script: random.sh
      register: result
      retries: 100
      until: result.stdout.find("10") != -1
      # n.b. the default delay is 5 seconds
      delay: 1

#-- randon.sh
#!/bin/bash
echo $((1 + RANDOM % 10))
```

The `delay` is overwritten from `1sec` to `5sec`,

### 🏇🏻 Asynchronous, Serial and Parallel operations in ansible

This section explains how one can improve the playbook execution and performance aspects of ansible,

Also, understands the existing pitfalls,

We are creating a set of `sleep` commands, that mimics a slow command execution,

```yaml
 tasks:
    - name: Task 1
      command: /bin/sleep 5

 # Repeat to add different tasks
```

To get the time

```bash
time ansible-playbook slow_playbook.yaml
```

Here ansible strategy is linear, it waits for completing the tasks on **all hosts,** then only move to the next task.

How we can improve it to make the execution faster,

By using `when: ansible_hostname == 'ubuntu3'`, with play book can improve as the other hosts just skipped

Ansible has support for `asynchronous` task execution, that helps execution of long running tasks, the tasks that will exceed the ssh connection timeout,

We can run the tasks then later `poll` for the results,

```yaml
  tasks:
    - name: Task 1
      command: /bin/sleep 5
      when: ansible_hostname == 'centos1'
      async: 10
      poll: 1
    register: result1
```

→ `async: 10` - Wait at least 10 seconds

→ `poll: 1` - Poll for the status every one second

a `poll: 0` means - no polling, ie. fire & forget

If we set `poll: 0`, then the tasks finishes immediately, even if the sleep time is 24 hrs.

but the ssh tasks are still running and can check with `ps -ef | grep ssh`, only after the execution time completes it will terminate

```yaml
{
    "msg": {
        "ansible_job_id": "234294177731.688",
        "changed": true,
        "failed": 0,
        "finished": 0,
        "results_file": "/root/.ansible_async/234294177731.688",
        "started": 1
    }
}
```

This the output of the registered `result1` variable,

There is a started and finished keys for jobs that run, there is one `ansible_job_id`: , that which the ansible uses for identifying the job status when polling,

Now we can step into a more complex task,

→ Define a list of `jobids`

→ Register the results for each task

→ Check whether the `jobid` is present in the result, if exists append to the list

```yaml
  vars:
    jobids: []

  tasks:
    - name: Task 1
      command: /bin/sleep 5
      when: ansible_hostname == 'centos1'
      async: 10
      poll: 0
      register: result1
    # ...
   - name: Capture Job IDs
      set_fact:
        jobids: >
                {% if item.ansible_job_id is defined -%}
                  {{ jobids + [item.ansible_job_id] }}
                {% else -%}
                  {{ jobids }}
                {% endif %}
      with_items: "{{ [ result1, result2, result3, result4, result5, result6 ] }}"

    - name: Show Job IDs
      debug:
        var: jobids
```

Now we can use another module called Asynchronous Status module, wait for all background modules to complete,

```yaml
    - name: 'Wait for Job IDs'
      async_status:
         jid: "{{ item }}"
      with_items: "{{ jobids }}"
      register: jobs_result
      until: jobs_result.finished
      retries: 30
```

It checks the status of the process with job id , until `jobs_result.finished` is `True`, retries a max of 30 times,

Switching to asynchronous saves time,

By default ansible uses linear strategy, also another default of `5` hosts, ie

it will execute 1 task at 5 hosts max. at a time after completing it move to next task,

By setting `fork` no in `ansible.cfg` to `6`, no of parallel executions can be made to 6,

```yaml
[defaults]
inventory = hosts
host_key_checking = False
forks=6
```

If need to run tasks in batches of hosts (eg: for rolling updates to a batch of system), here taking `2` at a time use `serial` , set that in the playbook

```yaml
  hosts: linux
  gather_facts: false
  serial: 2

# Setting serial as incremental batches
  serial:
  - 1
  - 2
  - 3
# When working with large no. of hosts it is better to use %
 serial:
  - 15%
  - 34%
  - 50%
```

Setting `strategy` to a playbook,

```yaml
  hosts: linux
  gather_facts: false
  strategy: free

  tasks:
    - name: Task 1
   # random no. up to 10
      command: "/bin/sleep {{ 10 |random}}"
```

With `strategy: free`,  There is no waiting for one task to finish and move to 2nd task, as far as one task in a host completes it gets the second task,

## 📝 Task Delegation

To delegate specific tasks to specific targets - ie hosts,

To show how to delegate tasks to specific hosts, we will assign some TCP wrappers that it will restrict SSH access so that it works only works from certain hosts,

### 🇦🇸 Sample Task to be done

```yaml
- name: Play-1 - Generate SSH key in ubuntu-c
 # Generate SSH keys in ubuntu-c (running as current ansible user)
  hosts: ubuntu-c
  gather_facts: False

  tasks:
    - name: Generate an OpenSSH keypair for ubuntu3
      openssh_keypair:
        path: ~/.ssh/ubuntu3_id_rsa

- name: Play-2 - Copy OpenSSH keypair with permissions to all linux hosts
  hosts: linux
  gather_facts: False

  tasks:
    - name: Copy ubuntu3 OpenSSH keypair with permissions
      copy:
        owner: root
        src: "{{ item.0 }}"
        dest: "{{ item.0 }}"
        mode: "{{ item.1 }}"
   # groups the same index items from two given lists
      with_together:
        - [ ~/.ssh/ubuntu3_id_rsa, ~/.ssh/ubuntu3_id_rsa.pub ]
        - [ "0600", "0644" ]

- name: Play-3 - Add public key to `authorized_keys` File(allow connecting to ubuntu3)
  hosts: ubuntu3
  gather_facts: False

  tasks:
    - name: Add public key to the ubuntu3 authorized_keys file
      authorized_key:
        user: root
        state: present
        key: "{{ lookup('file', '~/.ssh/ubuntu3_id_rsa.pub') }}"
```

After this playbook run, all `linux` hosts can be connected to `ubuntu-3`, (that all shares a common private key and the pub key is added to `ubuntu-3` s - `authorized_keys` file.

Now, we can have an additional task to check if ssh connection can be made to the host/not.

```yaml
- name: Check SSH connection
  hosts: all
  gather_facts: False

  tasks:
    - name: Check that ssh can connect to ubuntu3 using the ssh tool
      command: ssh -i ~/.ssh/ubuntu3_id_rsa -o BatchMode=yes -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null root@ubuntu3 date
      changed_when: False
      ignore_errors: True
```

Extra parameters used & description,

- `-i` - To specify the private inp. key path which is `~/.ssh/ubuntu3_id_rsa`.
- `-o` - for options, `BatchMode=yes` → If it is enabled SSH will fail if a fingerprint/key is not accepted, `StrictHostKeyChecking=no` → As long as strict host key checking is enabled, the SSH client connects only to SSH hosts listed in the known host list. It rejects all other SSH hosts, if it is disabled new hosts gets added to known hosts as the connection being made, so now ssh wants to write to a `known_hosts` file which we are specifying as `/dev/null` as we no longer need the data by setting `UserKnownHostsFile=/dev/null`,
- we are running the command `date` in the hosts after ssh connects
- `changed_when: False` - As we are more concerned about the SSH access state (open or not), we need to neglect the change caused by the command module, this flag helps in that to shows a green output when the SSH is connected
- `ignore_errors: True` - Run the task as informative only,

***Result***:- All the hosts get connected via ssh and shows green OK.

### 🧍‍♀️ Task Delegation to host

We are gathering facts from 3 hosts `ubuntu-c`, `centos1`, `ubuntu1`, then use that facts to add a line to `/etc/hosts.allow` in `ubuntu-3` with task delegation using `delegate_to: <host_name>` option,

```yaml
- name: Add selected hosts to ubuntu3's /etc/hosts.allow for sshd
  hosts: ubuntu-c, centos1, ubuntu1
  # Serial is important as we are writing to a single file
  # run taks one host at a time
  serial: 1

  tasks:
    - name: Add host to /etc/hosts.allow for sshd
      lineinfile:
        path: /etc/hosts.allow
        line: "sshd: {{ ansible_hostname }}.diveinto.io"
        create: True
      delegate_to: ubuntu3
```

As only allow rules are added, we can still ssh in to it so the ssh test wouldn't complain,

Now we can add some `deny` rules to `ubuntu-3` and check again,

```yaml
- name: Remove connectivity from hosts (other than the ones specified in `/etc/hosts.allow`) by adding entry in `/etc/hosts.deny` for ALL
  hosts: ubuntu3
  gather_facts: False

  tasks:
    - name: Drop SSH connectivity from everywhere else
      lineinfile:
        path: /etc/hosts.deny
        line: "sshd: ALL"
        create: True
```

So now the `centos2,3`, `ubuntu2,3` all gets connection refused and shows `Connection reset by peer`, (the errors gets ignored - so the play wont stops).

Now the clean up tasks, to remove the added entries from `hosts.allow` and `hosts.deny` files, by mentioning  `state: absent` for `lineinfile`.

```yaml
-
  hosts: ubuntu-c, centos1, ubuntu1
  serial: 1

  tasks:
    - name: Remove specific host entries in /etc/hosts.allow for sshd
      lineinfile:
        path: /etc/hosts.allow
        line: "sshd: {{ ansible_hostname }}.diveinto.io"
        state: absent
      delegate_to: ubuntu3

-
  hosts: ubuntu3
  gather_facts: False

  tasks:
    - name: Allow SSH connectivity from everywhere
      lineinfile:
        path: /etc/hosts.deny
        line: "sshd: ALL"
        state: absent
```

## 🧙‍♂️ Magic Variables

[Ansible Documentation](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html),

The magic variables are always set by ansible that reflect internal state.

New releases of ansible introduces changes in available magic variables,

```yaml
- name: Fetch all the variables available to the play
  hosts: all

  tasks:
    - name: Using template, create a remote file that contains all variables available to the play
      template:
        src: templates/dump_variables
        dest: /tmp/ansible_variables

    - name: Fetch the templated file with all variables, back to the control host
      fetch:
        src: /tmp/ansible_variables
        dest: "captured_variables/{{ ansible_hostname }}"
        flat: yes

    - name: Clean up left over files
      file:
        name: /tmp/ansible_variables
        state: absent
```

`cat templates/dump_variables` gives

```yaml
PLAYBOOK VARS (Ansible vars):

{{ vars | to_nice_yaml }}
```

`to_nice_yaml` filter converts to readable o/p, then copy the file again for inspection,

The output of each host dump is a large file with 10000+ lines,

The important one to look at it are `hostvars`, `groups`, `group_names`, `inventory_hostname` .. `inverntory_dir` .. etc

## 🧱 Blocks

Blocks allow us to group a number of tasks in to groups, also provides other features such as error handling

```yaml
- name: Practicing with block
  hosts: linux

  tasks:
    - name: A block of modules being executed
      block:
        - name: Example 1
          debug:
            msg: Example 1

        - name: Example 2
          debug:
            msg: Example 2

        - name: Example 3
          debug:
            msg: Example 3
```

<aside>
👉🏻 `name` inside `block` is an Ansible 2.3 feature,

</aside>

Each of the example tasks inside a block executed at a time for hosts, then move to next

Can use blocks with other keywords like `when` , `with_items` etc, within the block and for the block too.

```yaml
- name: Blocks practice
  hosts: linux

  tasks:
    - name: A block of modules being executed
      block:
        - name: Example 1 CentOS only
          debug:
            msg: Example 1 CentOS only
          when: ansible_distribution == 'CentOS'

        - name: Example 2 Ubuntu only
          debug:
            msg: Example 2 Ubuntu only
          when: ansible_distribution == 'Ubuntu'

        - name: Example 3 with items
          debug:
            msg: "Example 3 with items - {{ item }}"
          with_items: ['x', 'y', 'z']
```

### ⚠️ Error Handling for `blocks`

The Error analysis allows one to use functionalities similar to `try - except - finally` which are present in the python language.

```yaml
- name: Error Handling for blocks
  hosts: linux

  tasks:
    - name: Install patch and python-dns
      block:
        - name: Install patch
          package:
            name: patch

        - name: Install python-dnspython
          package:
            name: python-dnspython

      rescue:
        - name: Rollback patch
          package:
            name: patch
            state: absent

        - name: Rollback python-dnspython
          package:
            name: python-dnspython
            state: absent

      always:
        - debug:
            msg: This always runs, regardless
```

The `python-dnspython` module is not available for centos hosts so, the second task in the block will fail for centos hosts, this is where the `rescue` option helps to rollback the patch if the tasks failed,

## 🔏 Ansible Vault

For securing sensitive information in ansible, ansible vault can be used to

- Encrypting/Decrypting variables
- Encrypting/Decrypting files
- Re-Encrypting Data
- Can use multiple vaults

We are removing `ansible_become_pass` entry from hosts - `group_vars/<group>`, now the password gonna supplied through vault by creating a `vault` entry for `ansible_become_pass` by using the `ansible-vault` tool,

### 🚂 Encrypt string with ansible vault

```bash
ansible-vault encrypt_string --ask-vault-pass --name 'ansible_become_pass' 'password'
```

Need to encrypt the string, prompting to ask vault password (if not set set one) , then the key and value to encrypt[’vaultPass’]

After encryption add the hash value to `group_vars/ubuntu` ,

```yaml
ansible_become_pass: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          64383033353265356439353866616439626639633530316135306561646631353638643934323863
          3433663137356562393738656562626330333761373565340a653432643434666338313064613831
          39303961333861306161383534323038356534323363333461373336333662303534363435306534
          6136383564316364350a336166633835356566623063323837303766333564316664303365623338
          3138
```

It need to specify in command to use the vault information, that can be done using `--ask-vault-pass`, like

```bash
ansible --ask-vault-pass ubuntu -m ping -o
# Enter the password when prompted
```

### 🔐 Encrypt file with ansible vault

```bash
ansible-vault encrypt <file_name>
```

Now the enter the password then the vault content get replaced with encrypted values,

Use the var file as a normal variable file,

```yaml
- name: Use vault encrypted variable file
  hosts: linux
  vars_files:
     - external_vault_vars.yaml

  tasks:
    - name: Show external_vault_var
      debug:
        var: external_vault_var
```

```bash
ansible-playbook --ask-vault-pass vault_playbook.yaml
```

### 🔓 Decrypt vault encrypted file

```bash
ansible-vault decrypt <file_name>
```

For password change ansible got the option for `rekey`,

```bash
ansible-vault rekey <file_name>
```

Enter old new passwords,

To view encrypted files use,

```bash
ansible-vault view <encrypted_file_name>
```

To specify the vault password through a file,

```bash
echo vault_password > psw_file

ansible-vault view --vault-password-file psw_file <encrypted_file_name>
```

Another way to ask password is to prompt via ID,

```bash
ansible-vault view --vault-id @prompt <encrypted_file_name>

# --vault-id is more powerfull like
ansible-vault view --vault-id <name_of_vault>@<password_file> <encrypted_file_name>
```

So, **To encrypt a file as `named vault` use**,

```bash
ansible-vault encrypt --vault-id <name_for_vault>@prompt <file_name>.yml
# Eg
ansible-vault encrypt --vault-id vars@prompt ext_vars.yml
```

the vault name `vars` also get added to encryption,

Now to encrypt password,

```bash
ansible-vault encrypt_string --vault-id ssh@prompt 'ansible_become_pass' 'password'
```

Now the playbook relies on 2 different vault names and vault passwords, to manage them, (one for ext_vars and one for ssh_passwords)

```bash
ansible-playbook --vault-id vars@prompt --vault-id ssh@prompt vault_playbook.yaml
```

It is also possible to encrypt entire playbooks,

```bash
ansible-vault encrypt --vault-id playbook@prompt vault_playbook.yaml
```

Then to run this encrypted playbook,

```bash
ansible-playbook --vault-id vars@prompt --vault-id ssh@prompt --vault-id playbook@prompt vault_playbook.yaml
```
