# Jinja 2, Templates & Practice Task 1 <!-- omit in toc -->

## 🥡 Table of Contents <!-- omit in toc -->

- [🧞 Templating With Jinja 2](#-templating-with-jinja-2)
  - [😨 If & if-else statements](#-if--if-else-statements)
  - [👩🏻 For Loops with jinja2](#-for-loops-with-jinja2)
  - [🎞️ Ansible Jinja Filters](#️-ansible-jinja-filters)
  - [🕍 Ansible Template Module](#-ansible-template-module)
- [🤾🏻 Ansible Playbook Creation and Execution: Sample Project](#-ansible-playbook-creation-and-execution-sample-project)

## 🧞 Templating With Jinja 2

### 😨 If & if-else statements

- The debug message only shows when the host name is `ubuntu-c`, a `-` symbol used at the end of the if statements to remove “empty character” return.

    ```yaml
    tasks:
        - name: Ansible Jinja2 if
          debug:
            msg: >
                 --== Ansible Jinja2 if statement ==--

                 {# If the hostname is ubuntu-c, include a message -#}
                 {% if ansible_hostname == "ubuntu-c" -%}
                       This is ubuntu-c
                 {% endif %}
    ```

- With Configurations
  - with `ansible.cfg`,

        ```bash
        [defaults]
        inventory = hosts
        host_key_checking = False
        ```

  - And `hosts  --->`

    ```bash
    # hosts
    [control]
    ubuntu-c

    [centos]
    centos[1:3]

    [ubuntu]
    ubuntu[1:3]

    [linux:children]
    centos
    ubuntu
    ```

  - `group_vars`, `centos` and `ubuntu`

        ```bash
        ---
        # centos
        ansible_user: root
        ...
        ```

    ```bash
    ---
    # ubuntu
    ansible_become: true
    ansible_become_pass: password
    ...
    ```

  - `host_vars`, `centos1` and `ubuntu-c`

        ```bash
        ---
        # centos1
        ansible_port: 2222
        ...
        ```

    ```bash
    ---
    # ubuntu-c
    ansible_connection: local
    ...
    ```

- `if-elif-else` statement with jinja2

    ```yaml
    {% if ansible_hostname == "ubuntu-c" -%}
      This is ubuntu-c
    {% elif ansible_hostname == "centos1" -%}
      This is centos1 with it's modified SSH Port
    {% else -%}
      This is good old {{ ansible_hostname }}
    {% endif %}
    ```

- Another example to check if a variable is defined or not after setting a variable with `jinja2`.

    ```yaml
    {% set example_variable = 'defined' -%}
    {% if example_variable is defined -%}
     example_variable is defined
    {% else -%}
     example_variable is not defined
    {% endif %}
    ```

### 👩🏻 For Loops with jinja2

- Basic jinja for loop with index key,

    ```yaml
    {% for entry in ansible_interfaces -%}
     Interface entry {{ loop.index }} = {{ entry }}
    {% endfor %}
    ```

- Using range with `for` loop,

    ```yaml
    {% for entry in range(1, 11) -%}
     {{ entry }}
    {% endfor %}

    # prints out 1 to 10
    ```

- Using `break` and `continue` with for loop,

    ```yaml
    # start 10, end 0, with step of -1 => 10, 9, 8 .. 1
    {% for entry in range(10, 0, -1) -%}
     {% if entry == 5 -%}
      {% break %}
     {% endif -%}
     {{ entry }}
    {% endfor %}

    # prints only upto 6 from 10
    ```

    To use this it needs enabling extensions in `ansible.cfg`,

    ```yaml
    jinja2_extensions = jinja2.ext.loopcontrols
    ```

    We can also use extra syntax in jinja like,

    ```yaml
     {% if entry is odd -%}
      {% continue %}
     {% endif -%}
    ```

### 🎞️ Ansible Jinja Filters

- [Ansible builtin and Jinja filters](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html).
- Some Examples of Jinja Filters.

    ```yaml
    ---=== Ansible Jinja2 filters ===---

    --== min [1, 2, 3, 4, 5] ==--

    {{ [1, 2, 3, 4, 5] | min }}

    --== max [1, 2, 3, 4, 5] ==--

    {{ [1, 2, 3, 4, 5] | max }}

    --== unique [1, 1, 2, 2, 3, 3, 4, 4, 5, 5] ==--

    {{ [1, 1, 2, 2, 3, 3, 4, 4, 5, 5] | unique }}

    --== difference [1, 2, 3, 4, 5] vs [2, 3, 4] ==--

    {{ [1, 2, 3, 4, 5] | difference([2, 3, 4]) }}

    --== random ['rod', 'jane', 'freddy'] ==--

    {{ ['rod', 'jane', 'freddy'] | random }}

    --== urlsplit hostname ==--

    {{ "http://docs.ansible.com/ansible/latest/playbooks_filters.html" | urlsplit('hostname') }}
    ```

### 🕍 Ansible Template Module

Jinja2 is not directly wriiten to the playbook most of the times but with a `template`,

To use jinja2 file with `*.j2` extension, one can use templating engine with ansible `template` module,

```yaml
tasks:
 - name: Jinja2 template
  template:
   src: template.j2
   dest: "/tmp/{{ ansible_hostname }}_template.out"
   trim_blocks: true
   mode: 0644
```

This reads the template file as a source, then output the destination with the `hostname` in filename,

run it only in `ubuntu-c`, (control server)

sample `template.j2`

```bash
--== Ansible Jinja2 if statement ==--

{# If the hostname is ubuntu-c, include a message -#}
{% if ansible_hostname == "ubuntu-c" -%}
      This is ubuntu-c
{% endif %}
```

```bash
ansible-playbook jinja2_playbook.yaml -l ubuntu-c
```

Then `cat /tmp/ubuntu-c_template.out` gives,

```bash
--== Ansible Jinja2 if statement ==--

This is ubuntu-c
```

## 🤾🏻 Ansible Playbook Creation and Execution: Sample Project

***Target:- Playbook To install Nginx to multiple Servers running Ubuntu and Centos***

Repo link:- [https://github.com/Akshaychdev/Ansible-learn](https://github.com/Akshaychdev/Ansible-learn)

Outcomes:

- Create playbook that works on different OS distributions consistantly.
- Explore and use the ansible gem `Ansible Managed`, which is needed when ansible is used for configuration management.

    It adds a indication that a file or resource is managed by ansible and should not edit it.

***CHALLENGE***

1. Configure the `hosts` file to target the linux group.
2. Install the package named `epel-release` , which is needed for installing nginix in centos based systems, using either `yum` or `dnf` module.

    possible options to use,

    ```yaml
    update_cache:yes
    state:latest
    ```

    Target only `CentOS` hosts for this using the fact `ansible_distribution.`

    - Playbook for installing `epel-release` and nginix on linux hosts,

        ```yaml
        ---
        # Installing epel for centos hosts

        - name: Install nginix on hosts
          hosts: linux
          gather_facts: yes

          tasks:
            - name: Installing epel for centos hosts
              yum:
                name: epel-release
                state: latest
                update_cache: yes
              when: ansible_distribution == "CentOS"

            - name: Installing nginix for centos hosts
              yum:
                name: nginix
                state: latest
                update_cache: yes
              when: ansible_distribution == "CentOS"

            - name: Installing nginix on ubuntu hosts
              apt:
                name: nginix
                state: latest
                update_cache: yes
              when: ansible_sistribution == "Ubuntu"
        ```

3. Use the common package module in place of `yum` and `apt` to simplify the process.

- Playbook

    ```yaml
    ---
    # Installing epel for centos hosts

    - name: Install nginix on hosts
      hosts: linux
      gather_facts: yes

      tasks:
        - name: Installing epel for centos hosts
          yum:
            name: epel-release
            state: latest
            update_cache: yes
          when: ansible_distribution == "CentOS"

        - name: Installing nginx
          package:
            name: nginx
            state: latest

      - name: Restart Nginx
          service:
            name: nginx
            state: restarted
    ```

The reverse proxy section in the web UI [http://localhost:1000/](http://localhost:1000/) can be used to check the running nginix on these linux flavours,

1. **Create a handler** : Check HTTP service, with `uri` module, create a url with `ip_address` of each host machines and check whether the port `80` is accessible on all machine (which are now loaded with nginix default pages) - > ie whether the return code is `200` or not.

    For a handler to work it needs to be notified - notify it from restart service.

    <aside>
    💯 *Handlers are Tasks that only runs when a change is made on host on notify, eg: restart aapache if a task updates the configuration, but not if the configuration is unchanged, Handlers are tasks that only run when notified*

    </aside>

    - Check Availability after restart nginix - handler

        ```yaml
        - name: Checking Server Installation
          hosts: linux
          gather_facts: yes

          tasks:
            - name: Restart Nginx
              service:
                name: nginx
                state: restarted
              notify:
                - Check Web-Server Availability

          handlers:
            - name: Check Web-Server Availability
              uri:
            # From ansible fact - ansible_default_ipv4.address
                url: http://{{ ansible_default_ipv4.address }}
                status_code: 200
        ```

2. Create `groupvar` entries for both centos and ubuntu hosts with the values as Web root paths,

    ```yaml
    # in centos group vars: <playbook_dir>/group_vars/centos
    nginx_root_location: /usr/share/nginx/html
    # ubuntu : <playbook_dir>/group_vars/ubuntu
    nginx_root_location: /var/www/html
    ```

    Create a task with name `Template index.html-base.j2 to index.html on target`,

    Where a source template `html-css` file is already loaded in `<playbook_dir>/templates/index.html-base.j2`

    Use the `template` module to copy to destination paths (from `groupvars`) and set permission `0644`

    - `Template index.html-base.j2 to index.html on target`

        ```yaml
        tasks:
            # To copy a base template from source to different hosts to group-var set destination location
            # (Templates are processed by the Jinja2 templating language.)
            # No need to specify templates/index.html-base.j2 as template module looks in templates
            - name: Template index.html-base.j2 to index.html on target
              ansible.builtin.template:
                src: index.html-base.j2
                # Paste as `index.html`
                dest: '{{ nginx_root_location }}/index.html'
                mode: '0644'
        ```

3. Ansible Managed,
    - Update the `ansible.cfg` and include,

        ```bash
        ansible_managed = Managed by Ansible - file:{file} - host:{host} - uid:{uid}

        # With Date
        ansible_managed = Managed by Ansible - file:{file} modified on %Y-%m-%d %H:%M:%S by uid:{uid} on host:{host}
        ```

    - Update the Templating task to `Template index.html-ansible_managed.j2 to index.html on target`, update the template file src with `index.html-ansible_managed.j2`,

        <aside>
        🤷🏿‍♀️ `ansible_managed`, *is a global ansible variable*
        ***T**o ensure people looking at the remote file would keep their paws off it, warning against manual modifications to the file which would be overwritten at the next playbook run*

        </aside>

        - Result Code

            ```bash
            # To copy a modilfied template file - That containes the `ansible_managed` variable
                # Indicating the file is managed by ansible - index.html-ansible_managed.j2
                - name: Template index.html-ansible_managed.j2 to index.html on target
                  ansible.builtin.template:
                    src: index.html-ansible_managed.j2
                    dest: '{{ nginx_root_location }}/index.html'
                    mode: '0644'
            ```

        ![Untitled](Jinja%202,%20Templates%20&%20Practice%20Task%201%201fcc4ad752a040cbade1770613f624cc/Untitled.png)

        Now the template shows this comment.

4. Update the `template` to include the `vars` file (`vars/logos.yaml`), (contains centos and ubuntu logo data)
    - Adding external variables to playbook,

    ```yaml
    - hosts: all
      remote_user: root
      vars:
        favcolor: blue
      vars_files:
        - /vars/external_vars.yml
    ```

    - Solution

        ```yaml
        - name: Install nginix on hosts
          hosts: linux
          gather_facts: yes
          # Load external variables
          vars_files:
            - vars/logos.yaml

          tasks:

            # Adding logo data for centos and ubuntu hosts in template by using
            # external variables added to the playbook,
            - name: Template index.html-logos.j2 to index.html on target
              ansible.builtin.template:
                src: index.html-logos.j2
                dest: '{{ nginx_root_location }}/index.html'
                mode: '0644'
        ```

    Now there is a logo on side,

    ![Untitled](Jinja%202,%20Templates%20&%20Practice%20Task%201%201fcc4ad752a040cbade1770613f624cc/Untitled%201.png)

5. Easter Egg Task, Install the `unzip` package with ansible on hosts and use the inbuilt ansible archive module to unzip files to nginix root (that contains code for a easter egg game)
    - result code

        ```yaml
        # Installing Unzip Package
            - name: Installing Unzip
              package:
                name: unzip
                state: latest

            # Unzip playbook stacker game to nginx_root_location
            - name: Unarchive Playbook Stacker game
              ansible.builtin.unarchive:
                src: playbook_stacker.zip
                dest: '{{ nginx_root_location }}'
                mode: 0755

            # Add the template index.html-easter_egg.j2
            - name: Template index.html-easter_egg.j2 to index.html on target
              ansible.builtin.template:
                src: index.html-easter_egg.j2
                dest: '{{ nginx_root_location }}/index.html'
                mode: '0644'
        ```

    ![Untitled](Jinja%202,%20Templates%20&%20Practice%20Task%201%201fcc4ad752a040cbade1770613f624cc/Untitled%202.png)
