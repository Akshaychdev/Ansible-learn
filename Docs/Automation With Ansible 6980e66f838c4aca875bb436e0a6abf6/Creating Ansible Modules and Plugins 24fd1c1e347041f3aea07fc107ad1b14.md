# Creating Ansible Modules and Plugins <!-- omit in toc -->

## 🥡 Table of Contents <!-- omit in toc -->

- [🚀 Creating Ansible Modules](#-creating-ansible-modules)
  - [👏🏽 Creating a PING module](#-creating-a-ping-module)
  - [🐚 Custom Module - Shell Script to MIMIC the JSON output from module](#-custom-module---shell-script-to-mimic-the-json-output-from-module)
  - [🏃 Testing the Final `ping` module with shell script](#-testing-the-final-ping-module-with-shell-script)
  - [🪝Create ansible modules in Python](#create-ansible-modules-in-python)
- [👩🏿‍🔧 Creating Ansible Plugins](#-creating-ansible-plugins)
  - [👨🏾‍💻 Developing Custom Plugins](#-developing-custom-plugins)
  - [🎞️ Developing Filters](#️-developing-filters)

## 🚀 Creating Ansible Modules

Topics,

- Download the ansible source code
- Use the developer “Hacking” tools to Interrogate modules
- Understand the structure (Return Code and JSON), to report module success or failure.
- Build a simple ping module, using shell script.
- Move the simple ping module to python code leveraging Ansible Module Framework,
- Show debug Information for Failures.

### 👏🏽 Creating a PING module

The ping module exists in ansible is a connectivity check (SSH), what we are implementing is a Network ICMP(Internet Control Message Protocol) ping, can use linux `ping` command to bootstrap this.

Get the ansible source-code, to look at the inbuilt modules and module testing utility,

```bash
cd ~
git clone https://github.com/ansible/ansible.git
```

There is a Test module built in to the ansible at location

```bash
vim ~/ansible/hacking/test-module

# Update environment to python3
#!/usr/bin/env python => #!/usr/bin/env python3
```

We can check the functionality of ansible test module by checking an already existing module like the `command` module (located at `~/ansible/lib/ansible/modules/command.py`), (`test-module` is an executable, and can be used just like the `ansible-playbook`) - with a simple argument `hostname`,

```bash
~/ansible/hacking/test-module -m ~/ansible/lib/ansible/modules/command.py -a hostname
```

![Untitled](Creating%20Ansible%20Modules%20and%20Plugins%2024fd1c1e347041f3aea07fc107ad1b14/Untitled.png)

The `changed` parameter will also be set “true” in the JSON output from the `test-module` execution.

Simulating a failed test output by supplying a non-existing argument,

```bash
~/ansible/hacking/test-module -m ~/ansible/lib/ansible/modules/command.py -a ijjindakiko
```

![Untitled](Creating%20Ansible%20Modules%20and%20Plugins%2024fd1c1e347041f3aea07fc107ad1b14/Untitled%201.png)

### 🐚 Custom Module - Shell Script to MIMIC the JSON output from module

Can start creating the custom `ping` module by writing a shell script that will produce JSON outputs as per the standards like a normal ansible module - `icmp.sh`

```bash
#!/bin/bash

# count reqeusts - 1, stdout and stderr bypass to null
ping -c 1 127.0.0.1 >/dev/null 2>/dev/null

if [ $? == 0 ];
  then
  echo "{\"changed\": true, \"rc\": 0}"
else
  echo "{\"failed\": true, \"msg\": \"failed to ping\", \"rc\": 1}"
fi
```

```bash
# output
{"changed": true, "rc": 0}
```

Make the shell script to fail by changing the [localhost](http://localhost) ip to `128.0.0.1`,

```bash
# failed output
{"failed": true, "msg": "failed to ping", "rc": 1}
```

Now we can use the `hacking-test` module in ansible to test our `[icmp.sh](http://icmp.sh)` module,

```bash
~/ansible/hacking/test-module -m icmp.sh
```

![Untitled](Creating%20Ansible%20Modules%20and%20Plugins%2024fd1c1e347041f3aea07fc107ad1b14/Untitled%202.png)

Now we can change the shell script to accept parameters, ie the hosts and IP address to ping to.

```bash
#!/bin/bash

# Capture inputs, these are passed as a file to the module
source $1 >/dev/null 2>&1

# Set our variables, set default if not assigned
TARGET=${target:-127.0.0.1}

ping -c 1 ${TARGET} >/dev/null 2>/dev/null

if [ $? == 0 ];
  then
  echo "{\"changed\": true, \"rc\": 0}"
else
  echo "{\"failed\": true, \"msg\": \"failed to ping\", \"rc\": 1}"
fi
```

→ `source $1 >/dev/null 2>&1` : Ansible modules expect all of the variables to be passed as a file, which is the first argument passed to the script, (ansible test does the passing of variables)

So by sourcing the file, if the `target` is set we can use that else use default `127.0.0.1`,

```bash
# using with target
~/ansible/hacking/test-module -m icmp.sh -a "target=centos1"
```

Note that the generated output and inp. are saving to location `/home/ansible/*`,

- `/home/ansible/.ansible_module_generated` : Will be the script we used ie `icmp.sh`.
- `/home/ansible/.ansible_test_module_arguments`: The arguments passed like “target=centos4"

So we can also re-run the last tested module with

```bash
/home/ansible/.ansible_module_generated /home/ansible/.ansible_test_module_arguments
```

### 🏃 Testing the Final `ping` module with shell script

Ansible will look for custom modules in a directory called `library` relative, playbook to use the custom module build

The `[icmp.sh](http://icmp.sh)` is renamed to `icmp` and put in location `icmp/library`,

```bash
ansible-playbook icmp_playbook.yaml
```

```yaml
-
  hosts: linux

  tasks:
    - name: Test icmp module
      icmp:
        target: 127.0.0.1
```

That task actually lets ping every hosts to themselves via icmp network ping -

### 🪝Create ansible modules in Python

With python we get a lot of templated code already built - with command line arguments, templates for easy documentation (`ansible-doc`), that makes it more ready to publish.

[Developing modules in ansible with Python](https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_general.html)

The `[icmp.sh](http://icmp.sh)` converted to python equivalent with documentation,

[icmp_ping.py](Creating%20Ansible%20Modules%20and%20Plugins%2024fd1c1e347041f3aea07fc107ad1b14/icmp_ping.py)

```python
# the main run script used
def run_module():
    module_args = dict(
        target=dict(type='str', required=True)
    )

    result = dict(
        changed=False
    )

    module = AnsibleModule(
        argument_spec=module_args,
        supports_check_mode=True
    )

    if module.check_mode:
        return result

    ping_result = module.run_command('ping -c 1 {}'.format(module.params['target']))

    # use whatever logic you need to determine whether or not this module
    # made any modifications to your target
    if module.params['target']:
        result['debug'] = ping_result
        result['rc'] = ping_result[0]
        if result['rc']:
          result['failed'] = True
          module.fail_json(msg='failed to ping', **result)
        else:
          result['changed'] = True
          module.exit_json(**result)
```

To run the module

```bash
~/ansible/hacking/test-module -m icmp.py -a "target=centos1"
```

To view the co-related document, (after moving the [`icmp.py`](http://icmp.py) to `library`)

```bash
ansible-doc -M library icmp.py
```

## 👩🏿‍🔧 Creating Ansible Plugins

In ansible there are lot of plugins hidden or we are using unknowingly that this is a plugin,

For example the `with_items`, keyword - we are using a builtin lookup plugin called `items.py`,

which is located at ansible code base - [https://github.com/ansible/ansible/blob/devel/lib/ansible/plugins/lookup/items.py](https://github.com/ansible/ansible/blob/devel/lib/ansible/plugins/lookup/items.py)

Also host vars and group vars - [https://github.com/ansible/ansible/blob/devel/lib/ansible/plugins/vars/host_group_vars.py](https://github.com/ansible/ansible/blob/devel/lib/ansible/plugins/vars/host_group_vars.py)

Official Ansible Doc on developing Plugin:

[https://docs.ansible.com/ansible/latest/dev_guide/developing_plugins.html](https://docs.ansible.com/ansible/latest/dev_guide/developing_plugins.html)

### 👨🏾‍💻 Developing Custom Plugins

In ansible documentation it describes the custom directories that we should use for plugins,

[https://docs.ansible.com/ansible/latest/dev_guide/developing_plugins.html#developing-particular-plugin-types](https://docs.ansible.com/ansible/latest/dev_guide/developing_plugins.html#developing-particular-plugin-types)

![Untitled](Creating%20Ansible%20Modules%20and%20Plugins%2024fd1c1e347041f3aea07fc107ad1b14/Untitled%203.png)

We are creating a custom plugin by modifying already existing `with_items` plugin, named `with_items_sorted`,- That sorts a given lists before performing a lookup

As we are creating a `lookup_plugin`, we needs to look in `lookup_plugins` directory,

```bash
# create directory
mkdir lookup_plugins
cd lookup_plugins
```

When creating plugins and filters always start with a known working example, to get the syntax - methods - documentation style etc.

Download the `[items.py](http://items.py)` from source code,

```bash
wget https://raw.githubusercontent.com/ansible/ansible/devel/lib/ansible/plugins/lookup/items.py

# rename
mv items.py sorted_items.py
```

There is a main import of code from lookup base, which is at

[https://github.com/ansible/ansible/blob/stable-2.13/lib/ansible/plugins/lookup/__init__.py](https://github.com/ansible/ansible/blob/stable-2.13/lib/ansible/plugins/lookup/__init__.py)

There is an abstract - method called `run` in it, that means When the playbook specifies a lookup, this method is run.  The arguments to the lookup become the arguments to this method.

```python
from ansible.plugins.lookup import LookupBase

# LookupBase gets inherited
class LookupModule(LookupBase):

    def run(self, terms, **kwargs):

        return self._flatten(terms)
```

It just return a flatten list (checks for incoming terms and group to single list)

The modification is done to the terms and - ie gets sorted

```python
    def run(self, terms, **kwargs):
        return self._flatten(sorted(terms, key=str))

# better approach
    def run(self, terms, **kwargs):
        return sorted(self._flatten(terms), key=str)
```

Also replace `with_items` ⇒ `with_sorted_items`

```bash
# using vim find and replace
:1,$s/with_items/with_sorted_items/g
```

Example playbook to test it,

```yaml
-
  hosts: centos1

 tasks:
     - name: loop through list
       debug:
         msg: "An item: {{item}}"
       with_sorted_items:
         - 3
         - 2
         - 1
         - Z
         - A
         - M
```

### 🎞️ Developing Filters

Make a directory called `filter_plugins`, clone the filter `[core.py](http://core.py)` source code,

```bash
mkdir filter_plugins
cd filter_plugins

wget https://raw.githubusercontent.com/ansible/ansible/devel/lib/ansible/plugins/filter/core.py

mv core.py reverse_upper.py

# Set numbering
: set number
```

Making a filter `reverse_upper` taking base `regex_escape` in `core.py`,

```bash
def reverse_upper(string):
    """
    Reverse and Upper String
    """
    return string[::-1].upper()


class FilterModule(object):
    ''' Ansible core jinja2 filters '''

    def filters(self):
        return {
            'reverse_upper': reverse_upper
            }
```

To test in `python3` command line,

```bash
from reverse_upper import reverse_upper

reverse_upper("SAT")
```

Play book to test the filter,

```bash
-
  hosts: all

  tasks:
     - name: Reverse and upper ansible_distribution
       debug:
         msg: "Reverse and upper of ansible_distribution: {{ ansible_distribution | reverse_upper }}"
```
