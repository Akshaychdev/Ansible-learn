# Structuring Ansible Playbooks <!-- omit in toc -->

## 🥡 Table of Contents <!-- omit in toc -->

- [⏬ Using Includes and Imports](#-using-includes-and-imports)
  - [🤏 **Include Tasks**](#-include-tasks)
  - [🍤 **Import Tasks**](#-import-tasks)
- [🏷️ Using Tags](#️-using-tags)
  - [🧑🏽‍🦲 **Special Tags**](#-special-tags)
- [🦹🏽‍♂️ Ansible Roles](#️-ansible-roles)
  - [⛽ Creating and managing ansible roles](#-creating-and-managing-ansible-roles)

**MAIN TOPICS**

- Using Includes and Imports
- Using Tags
- Using Roles

## ⏬ Using Includes and Imports

- `include_tasks`
- `import_tasks`
- Static Vs Dynamic
- `import_playbook`

If it need to install a prerequisite of applications before running a playbook,
that can be written on a separate module and to be included in main playbook

### 🤏 **Include Tasks**

```yaml
-
  hosts: all
  tasks:
     - name: Play 1 - Task 1
       debug:
         msg: Play 1 - Task 1

     - include_tasks: play1_task2.yaml
```

The `play1_task2.yaml`,

```yaml
- name: Play 1 - Task 2
  debug:
    msg: Play 1 - Task 2
```

### 🍤 **Import Tasks**

```yaml
-
  hosts: all

  tasks:
     - name: Play 1 - Task 1
       debug:
         msg: Play 1 - Task 1

     - import_tasks: play1_task2.yaml
```

The `play1_task2.yaml` is the same file,

The output looks the same but there is a major difference in how both are done,

- All `import` statements are pre-processed before execution, which are **static** behaviour.
- `when` statement applies to  **all** individual tasks at task point of execution.

    ![Untitled](Structuring%20Ansible%20Playbooks%20798161b1cc0047809ada760a74ac005e/Untitled.png)

- All `include` statements are processed as they are encountered, **dynamic** behaviour.
- `when` statement applies to **all** tasks at initial point of execution

    ![Untitled](Structuring%20Ansible%20Playbooks%20798161b1cc0047809ada760a74ac005e/Untitled%201.png)

To show the difference, can execute an example playbbok

```yaml
-
  hosts: centos1
  tasks:
     - debug:
         msg: ===================== Testing include_tasks =====================

     # include_tasks is dynamic
     #
     # The when statement is executed once, if the condition is met, all
     # tasks are executed
     - include_tasks: include_tasks.yaml
       when: include_tasks_var is not defined

     - debug:
         msg: ===================== Testing import_tasks ======================

     # import_tasks is static
     #
     # Each task that in the include will be independently executed against
     # the when condition
     - import_tasks: import_tasks.yaml
       when: import_tasks_var is not defined
```

`import_tasks.yaml`

```yaml
- set_fact:
    import_tasks_var: foo

- name: 2nd Task
  debug:
    msg: 2nd Task

- name: 3rd Task
  debug:
    msg: 3rd Task
```

`include_tasks.yaml`

```yaml
- set_fact:
    include_tasks_var: foo

- name: 2nd Task
  debug:
    msg: 2nd Task

- name: 3rd Task
  debug:
    msg: 3rd Task
```

when statement is set to `is not defined` so it will be executed only if the variable is not defined, also note that the first task is to define the variable,

**Result**:- In the `include_tasks` - As the condition is only checked once, At initial point the variable is not set, so the entire playbook will execute altogether,

But in `import_tasks`, condition get evaluated for individual tasks, to only the first task is executed and rest are skipped,

![Untitled](Structuring%20Ansible%20Playbooks%20798161b1cc0047809ada760a74ac005e/Untitled%202.png)

`**import_playbook**`

The `import_playbook`, There is no hosts or tasks defined in the main executed playbook,

It is taken from the imported playbook, it is also static as the tasks are individually executed against the `when` condition,

```yaml
# No hosts
- import_playbook: imported_playbook.yaml
  when: import_playbook_var is not defined
```

`imported_playbook.yaml`,

```yaml
-
  hosts: centos1
  tasks:
     - set_fact:
         import_playbook_var: true

     - debug:
         msg: Playbook executed
```

![Untitled](Structuring%20Ansible%20Playbooks%20798161b1cc0047809ada760a74ac005e/Untitled%203.png)

## 🏷️ Using Tags

- Segmentation and Execution With Tags
- Skipping With Tags
- Playbook Tags
- Special Tags and Tag Inheritance

Tags can be used in tasks so that, for a set of tasks for installing nginx that has got tags “installing-nginx” can be selectively executed from a playbook with,

For eg,

```yaml
-
  hosts: linux
  vars_files:
    - vars/logos.yaml

  tasks:
    - name: Install EPEL
      yum:
        name: epel-release
        update_cache: yes
        state: latest
      when: ansible_distribution == 'CentOS'
      tags:
        - install-epel

    - name: Install Nginx
      package:
        name: nginx
        state: latest
      tags:
        - install-nginx

    - name: Restart nginx
      service:
        name: nginx
        state: restarted
      notify: Check HTTP Service
      tags:
        - restart-nginx

# ......... more tags and tasks
```

```yaml
# if we need to install and restart nginx
# can use 1 or more tags seperated with a comma
ansible-playbook nginx_playbook.yaml --tags "install-nginx,restart-nginx"

# can do inverse operattion by selectivly omitting certain tags
# it will execute all other tags without restart-nginx tagged tasks
ansible-playbook nginx_playbook.yaml --skip-tags "restart-nginx"
```

By default all tags are assigned to `all` tag,

```yaml
ansible-playbook nginx_playbook.yaml --tags "all"
```

It is also possible to apply a tag to an entire Play,

Ie if there is a play for installing a web application with its components, a tag can be assigned to it by the name `webapp`,

<aside>
👉🏻 Note :- If we add a tag to an entire play, the `gather_facts` task will be changed from “always” to this “tag”

To avoid this effect one can add an empty play just with `hosts` to the playbook to ensure `gather_facts` be fetched always,

</aside>

```yaml
-
  hosts: linux
-
  hosts: linux
  tags:
    - webapp

  vars_files:
    - vars/logos.yaml
```

### 🧑🏽‍🦲 **Special Tags**

There is a special tag - `always`, that will be executed that ensures that task will be executed always, implicitly

```yaml
  - name: Restart nginx
    service:
        name: nginx
        state: restarted
      notify: Check HTTP Service
      tags:
        - always
```

Always tags also can be skipped with skip tags,

```yaml
ansible-playbook nginx_playbook.yaml "install-nginx" --skip-tags "always"
```

Other Special Tags,

- `tagged`: Only tasks with a tag will be run
- `untagged`: Only un-tagged Tasks Will be run, (always will run)
- `all`: By default ansible is running tasks with tag all

Tags can also be incorporated with `include` or `import` tasks, where every imported/included tasks and plays will be incorporated with that tags,

```bash
for tag in include_tasks import_tasks import_playbook; do echo =============== Testing ${tag} =============; ansible-playbook include_import_playbook.yaml --tag "${tag}"; done
```

## 🦹🏽‍♂️ Ansible Roles

Ansible Roles are simply a directory structure that makes using variables, templates, files, tasks.. etc from different sources easily manageable, accessible and ease of use.

Roles allows one to structure and group playbook tasks and associated components in to a role for ease of consumption,

Now we can look into,

- Using Roles and the Role structure
- Create roles with ansible Galaxy
- To move an existing project, to a Role
- Role execution, Role Parameters and Dependencies

Benefits of Using Roles,

- Larger Projects are made easier to manage
- Roles are grouped, with logical structure, making them easier to share
- Roles can be written, to specific requirements, for example a web server role, a DNS role or a patching role,
- Roles can be developed independently, in parallel by different entities
- Templates, Vars and Files, have designated directories and inclusion is simplified,
- Roles, can have dependencies on other roles, therefore providing automated inclusion.

A role is simply a directory structure with top level directory being the name of the role,

![Untitled](Structuring%20Ansible%20Playbooks%20798161b1cc0047809ada760a74ac005e/Untitled%204.png)

`defaults`: Where the default variables are included,

`files`: when using copy or script modules, this is where the module looks for associated files

`handlers`: Handler context included here

`meta`:  Where includes settings for roles and dependencies

`README.md`: Documentation regarding the role, must if the role is to be published

`tasks`: Include the tasks associated with the role

`templates`: put the files and `jinja2` templates here

`tests`: where you writes unit test for the role, tests can also be run as part of installation process with ansible-galaxy

`vars`: Where the role-vars are included.

### ⛽ Creating and managing ansible roles

Ansible role skeleton can be created using ansible galaxy.

1. Initialise the `nginix` role,

    ```bash
    ansible-galaxy init nginx

    find nginx
    # Directory Outline
    nginx
    nginx/vars
    nginx/vars/main.yml
    nginx/tests
    nginx/tests/inventory
    nginx/tests/test.yml
    nginx/handlers
    nginx/handlers/main.yml
    nginx/tasks
    nginx/tasks/main.yml
    nginx/meta
    nginx/meta/main.yml
    nginx/README.md
    nginx/templates
    nginx/files
    nginx/.travis.yml
    nginx/defaults
    nginx/defaults/main.yml
    ```

2. Next step is to move the earlier unorganised contents into the desired sections in the role.
    - handlers to `nginx/handlers/main.yml`.

    ```bash
    vim nginx_playbook.yaml nginx/handlers/main.yml

    # use :next to move to 2nd file for pasting in VIM
    # move between files without saving
    # use next! and prev! OR e! < FileName >

    ---
    # handlers file for nginx
    - name: Check HTTP Service
      uri:
        url: http://{{ ansible_default_ipv4.address }}
        status_code: 200
    ```

    - Templates to `nginx/templates`,

    ```bash
    mv templates/* nginx/templates/
    rm -rf templates
    ```

    - Files to `nginx/files/`

    ```bash
    mv files/* nginx/files/
    rm -rf files
    ```

    - Move the variables,

    ```bash
    cat vars/logos.yaml >> nginx/vars/main.yml

    # remove the excess yaml --- and ...'s
    vi nginx/vars/main.yml
    rm -rf vars
    ```

    - Move tasks to `nginx/tasks/main.yml`

    ```bash
    vi nginx_playbook.yaml nginx/tasks/main.yml

    # move the tasks to main.yml
    # All lines indentation decrese in vim, :1,$s/^    //g, (the space between - how many spaces to decrease), its a find and replace for 4 starting whitespaces with blank string

    # Also remove things other than hosts from nginx_playbook.yaml
    # no need specify source "templates" dir. explicitly in tasks/main.yml (as our files present in roles default location)
    ```

    - Finally update our main playbook `nginx_playbook.yaml` to include the created role,

    ```bash
    roles:
      - nginx
    ```

3. Set the webpage creation included in it as a separate role for more reusability,
    - `ansible-galaxy init` for the webapp,

    ```bash
    ansible-galaxy init webapp
    ```

    - Move `templates`, `files` and `vars`,

    ```bash
    mv nginx/templates/* webapp/templates/
    mv nginx/files/* webapp/files/

    # Swap the nginx - vars with webapp -vars
    mv webapp/vars/main.yml /tmp/
    mv nginx/vars/main.yml webapp/vars/
    mv /tmp/main.yml nginx/vars/
    ```

    - Edit the tasks and separate tasks for `webapp` and `nginx` roles ,

    ```bash
    vim nginx/tasks/main.yml webapp/tasks/main.yml
    # move the last 3 tasks webapp specific
    # rename main playbook file
    mv nginx_playbook.yaml nginx_webapp_playbook.yaml
    # Add the new role to webapp

    roles:
      - nginx
     - webapp
    ```

    - The template file destination is supplied by a variable `target_dir` under group vars,

    ```bash
    # in revision 6
    cat webapp/defaults/main.yml

    ---
    # defaults file for webapp
    target_dir: /var/www/html

    # based on std nginx ubuntu, that doesnt works for centos
    ```

    - Ansible got an alternate way of specifying roles that can pass parameters with it, here uses `jinja2` templates logic, along with role specification.

    ```bash
    roles:
     - nginx
      - { role: webapp, target_dir: "{%- if ansible_distribution == 'CentOS' -%}/usr/share/nginx/html{%- elif ansible_distribution == 'Ubuntu' -%}/var/www/html{%- endif %}" }
    ```

If one role has dependency over other role, can set this using `meta` context,

```bash
vi webapp/meta/main.yml

# Add to end
dependencies:
  - nginx
```

Now the `nginx` role can be removed from the playbook as it is a dependency to `webapp`
