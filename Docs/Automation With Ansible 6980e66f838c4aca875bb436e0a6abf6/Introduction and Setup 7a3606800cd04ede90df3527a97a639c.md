# Introduction and Setup <!-- omit in toc -->

## 🥡 Table of Contents <!-- omit in toc -->

- [🍳 Course Contents](#-course-contents)
- [⛳ Start With Automation](#-start-with-automation)
  - [🐙 Pull Based and Push based systems](#-pull-based-and-push-based-systems)
  - [🇻🇨 Usage of VCS in Automation](#-usage-of-vcs-in-automation)
- [🖊️ Introduction](#️-introduction)
- [🚇 Setup Ansible and Docker](#-setup-ansible-and-docker)
  - [🍀 **SSH connectivity overview between Ansible - server and host**](#-ssh-connectivity-overview-between-ansible---server-and-host)
- [🇧🇿 Ansible Configuration](#-ansible-configuration)
  - [🚅 Ansible Configuration Locations](#-ansible-configuration-locations)
- [👨🏾‍💻 Ansible Inventories](#-ansible-inventories)
  - [💽 INI Host file inventory](#-ini-host-file-inventory)
  - [🏮 Using `yaml` format for inventory](#-using-yaml-format-for-inventory)
  - [🦩 `JSON` Equivalent Inventory file](#-json-equivalent-inventory-file)
  - [👽 Extra Info with inventories](#-extra-info-with-inventories)

## 🍳 Course Contents

- Introduction to ansible
- Lab Setup
- Ansible architecture and design
- Ansible playbooks, Introduction
- Ansible playbooks, deeper
- Structuring Ansible playbooks
- Using ansible with cloud services and containers
- Creating Modules and Plugins
- Other resources and Supplementary.

## ⛳ Start With Automation

- Identify a single manual process, (like restarting servers), automate that single process with any of the tools available.. like ansible
- Then repeat the process by picking another process,...

### 🐙 Pull Based and Push based systems

![Introduction%20and%20Setup%207a3606800cd04ede90df3527a97a639c/Screenshot_from_2021-04-05_15-54-22.png](Introduction%20and%20Setup%207a3606800cd04ede90df3527a97a639c/Screenshot_from_2021-04-05_15-54-22.png)

1. **Pull Based**:- There is a master server that stores all the configuration information,

    Install agents on each of the child nodes (in which the conf. need to be managed)

    The Agents pulls the configuration from the master server.

    → Here Agents will poll the master server to check out if there is any changes in the master server, or a change triggers the master-slave connection, and the configuration management achieved.

    eg: Puppet and Chef

    ***Adv***

    - Clients Contact the server independently of each other, so the system is more scalable.

    ***DisAdv***

    - Need to install and manage an agent on each of the target systems.
2. **Push Based:**- There is a central server configured that pushes the changes to its child Nodes,

    Eg: Ansible

    ***Adv***

    - Easy to start as there is no need to install and manage an agent.

    ***DisAdv***

    - When scaled more, as the number of nodes increases, Some performance issue arises, need to depend on hyper-threading and other stuffs to achieve a stable performance

### 🇻🇨 Usage of VCS in Automation

- As all the systems users, infrastructure and configuration as code architecture, this is stored in vcs systems like git to ensure easy rollbacks,
- Master servers in both cases pulls there code from the VCS.

## 🖊️ Introduction

Ansible is an open source configuration management, software provisioning and application deployment tool set created by *Michael DeHaan* in 2012,

Acquired by Redhat inc. in 2015, since then the RedHat and opensource community further developed and improved the platform,

Ansible today is a multitude of tools, modules and software defined infrastructure collectively called as the ***ansible tool-set***. One of the major reason for the success of ansible is the [extensive module libraries](https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html) openly available for consumption.[*ranges from cloud computing, Networking, Server Configuration and Management, Virtualisation, containers .. etc*].

> ***"There may be an ansible module already out there to support the automation requirements you have"***
>

Also can create one's own module using the Ansible Framework,

1. **Ansible Executable**

Extend the power of Ansible and its modules to command Line.

![https://cdn.educba.com/academy/wp-content/uploads/2019/10/ansible-architecture.png](https://cdn.educba.com/academy/wp-content/uploads/2019/10/ansible-architecture.png)

- Type `ansible` at a prompt where ansible is installed
- Great for initial set-up and testing and everyday usage.
- Interact with modules and target infrastructure (server setup).

1. **Ansible Playbook**

Use ansible playbook executable

![https://4.bp.blogspot.com/-MjY5oVa9yog/WVhAHtzRgUI/AAAAAAAAAX8/XXN5eFX-qGYI9wYlQzC9oJPRCWblu8X-QCLcBGAs/w1200-h630-p-k-no-nu/ansible.PNG](https://4.bp.blogspot.com/-MjY5oVa9yog/WVhAHtzRgUI/AAAAAAAAAX8/XXN5eFX-qGYI9wYlQzC9oJPRCWblu8X-QCLcBGAs/w1200-h630-p-k-no-nu/ansible.PNG)

[Quick Ansible Lookup](http://unixcap.blogspot.com/p/ansible.html) :- link

Through a playbook use Ansible's Human readable configuration deployment and orchestration language.

A playbook is simply a book of `play`s, hmm 🤥,
A play simply the configurations and changes one would like to achieve against a set of targets,

1. **Ansible Inventories**

Inventories are a collection of targets, 🎯,
Its mainly the hosts that needs to be connected,
But can also consist of Hosts, Network Switches, Containers, Storage Arrays .. etc., it can also contain useful information about targets,

*Dynamic Inventories:-* where the inventory is an executable with data been sourced dynamically, so the data can be stored elsewhere and make use of it at the runtime,

## 🚇 Setup Ansible and Docker

1. **Install Docker on Ubuntu**

- Install docker

    [Digital ocean guide](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04)

- Install docker-compose

    [Docker guide](https://docs.docker.com/compose/install/)

    ```bash
    sudo apt-get install docker-compose-plugin

    # Check version
    docker compose version
    ```

1. **Install the Ansible Lab**

[Course Installation Repository](https://github.com/spurin/diveintoansible-lab) :- [https://github.com/spurin/diveintoansible-lab](https://github.com/spurin/diveintoansible-lab)

```bash
# permission issues with new docker compose plugin
sudo usermod -a -G docker [user]
# logout and login

# with detached mode
docker compose up -d
```

Wow!!, Now you got a fully working lab setup in local (💰 worth)

[http://localhost:1000/](http://localhost:1000/) → With the docker compose up to access the web interface (also nicely designed)

(i) Log in to `ubuntu-c`, with user `ansible` and password: `password` (Which is the ansible controller and others are hosts)

(ii) To refresh the containers, (uses docker volumes - so no data will be lost), use

```bash
# Only after stoping with `ctrl + c` or
docker-compose stop
docker-compose rm
```

(iii) connect with ssh naively, instead of using the web interface, (from the env file)

```bash
# Main ubuntu-c controller
ssh -p 2221 ansible@localhost # ubuntu-c

# child Hosts
ssh -p 2222 ansible@localhost # ubuntu-1
ssh -p 2223 ansible@localhost # ubuntu-2
ssh -p 2224 ansible@localhost # ubuntu-3
ssh -p 2225 ansible@localhost # centos-1
ssh -p 2226 ansible@localhost # centos-2
ssh -p 2227 ansible@localhost # centos-3

# to .bashrc/.zshrc
alias ubuntu-c="ssh -p 2221 ansible@localhost"
alias ubuntu-1="ssh -p 2222 ansible@localhost"
alias ubuntu-2="ssh -p 2223 ansible@localhost"
alias ubuntu-3="ssh -p 2224 ansible@localhost"
alias centos-1="ssh -p 2225 ansible@localhost"
alias centos-2="ssh -p 2226 ansible@localhost"
alias centos-3="ssh -p 2227 ansible@localhost"
```

[ttyd](https://github.com/tsl0922/ttyd) is the web terminal using

Docker extras, [visit this to get the essential docker commands](https://typeofnan.dev/how-to-stop-all-docker-containers/)

```bash
# Stop all running docker containers
docker kill $(docker ps -q)

# Remove all containers at once (probably dont need that)
docker rm $(docker ps -a -q)

# Docker containers with size
docker ps -s
```

[Extremely Useful Docker Commands List](https://www.codenotary.com/blog/extremely-useful-docker-commands/)

1. **Set-Up SSH connectivity between Hosts**

As Ansible is agent-less a trusted verified password less connection is required for  - automation and other tasks, ***SSH(**Secure Shell**)*** is used for this purpose

![https://linuxapt.com/assets/uploads/media-uploader/generate-ssh-keys-in-linux-01607975652.png](https://linuxapt.com/assets/uploads/media-uploader/generate-ssh-keys-in-linux-01607975652.png)

### 🍀 **SSH connectivity overview between Ansible - server and host**

![ssh-conn.png](Introduction%20and%20Setup%207a3606800cd04ede90df3527a97a639c/ssh-conn.png)

Out of any hosts can be connected via ssh, for ubuntu-1, enter

```bash
ssh ubuntu1
# enter password when prompted

# inspect cd .ssh/known_hosts
cat known_hosts
```

In `known_hosts`, there are two entries, when the fingerprint is accepted at the first time ssh connection, the fingerprint of both host-name and ip-address stored,

(can verify this using generating the fingerprint using the `ssh-keygen` command)

For the host:-  `ssh-keygen -H -F ubuntu1` (F - fingerprint)

![ssh-finger-match.png](Introduction%20and%20Setup%207a3606800cd04ede90df3527a97a639c/ssh-finger-match.png)

Now run the same command against the ip address (get the ip-address with by `ping` ing `ping ubuntu1`)

For client:- `ssh-keygen -H -F 172.18.0.3` (This matches with the second entry)

If the `known_hosts` file is deleted the fingerprint verification needed once again,

**Ansible - with SSH**

IN above there established a secure channel using password, the password gets validated and establishes the connectivity,

![ssh-connection-ansible-host.png](Introduction%20and%20Setup%207a3606800cd04ede90df3527a97a639c/ssh-connection-ansible-host.png)

But every time using a password is inconvenient and less secure, therefore a SSH public - private key pair is generated and access given by validating that,

The authorise key files located in `.ssh/` directory,

The file `.ssh/authorized_keys`  contains public keys of known hosts,

Secure SSH channel established from `ubuntu-c` using its private key, and the verifying with public keys from  "ubuntu-1, 2 and 3", as they tally up the connection is now established.

Now generate a public-private key pair for client `ubuntu-c`,

```bash
ssh-keygen
# keep all defaults

# pub key ends with host name
# ..... +9k= ansible@ubuntu-c
```

We can copy the contents of public key to the the hosts - `.ssh/authorized_keys` file, that need to be connected for verification,

But when doing this manually the permissions of the `.ssh` directory and the `authorized_keys` file need to set correct → otherwise SSH will reject it.

There is an inbuilt tool to do this ie, `ssh-copy-id`,

```bash
# u_name@remote-sys
ssh-copy-id ansible@ubuntu1

# Enter the pssword and it will copy the id, also that will be last time you need a password
```

Now comes the boring part all the remaining 5 hosts need to entered with the public key, from the ansible client, as there is no boring stuff and all the boring stuffs need to be automated,
Gonna auto**mate that task of coping ssh public key and entering password** first

with the use of `sshpass` package

```bash
sudo apt update
sudo apt install sshpass

# creaete a password.txt file with our password in it
echo password > password.txt

# Now the variations we need
# 2 users, ansible and root
# 2 os types, ubuntu and centos
# 3 + 3 instances, ubuntu1, 2 and 3 also CentOs 1, 2 and
```

```bash
#!/bin/bash

for user in ansible root
do
 for os in ubuntu centos
 do
  for instance in 1 2 3
  do
   sshpass -f password.txt ssh-copy-id -o StrictHostKeyChecking=no ${user}@${os}${instance}
  done
 done
done
```

To automatically accept fingerprints, use `StrictHostKeyChecking=no`,

→ `-f` passing a file with `sshpass`

→ `-o` for ssh, that tells an option is gonna use

→ `-p` can be used to enter password in the command itself, (adding hard-coded password never a best option) `-p 'password'`.

→  This is a good [stkoverflow ques](https://stackoverflow.com/questions/19302572/how-to-put-sshpass-command-inside-a-bash-script).
After running the script file clear the `password.txt`, created

Now the connections for the Lab look like this

![passwordless-hosts.png](Introduction%20and%20Setup%207a3606800cd04ede90df3527a97a639c/passwordless-hosts.png)

The fantastic `ansible` executable can now be used to check whether ansible can access all the hosts now linked with ssh,

```bash
ansible -i,ubuntu1,ubuntu2,ubuntu3,centos1,centos2,centos3 all -m ping
```

→ `-i` - Inventory (use "," after to specify inventory is part of command line) - usually inventory specified as a inventory-file
→ `all` - Group of hosts that wishes to target (here all hosts specified in the cmd line targeted)
→ `-m` - the ansible module to use, here using the `ping` module, that pings to the given host returns `pong` if successful.

1. **Setting The Repository Up**

in `ubuntu-c`, do a

```bash
git clone https://github.com/spurin/diveintoansible.git
```

## 🇧🇿 Ansible Configuration

- Ansible Configuration
- Ansible Inventories

    → Applying Host. Group variables

    → Simplify Groups with ranges

    → Add root connectivity to inventory

    → Different target configurations in ansible

- Ansible Module

    ![https://miro.medium.com/max/1400/1*Zva2UdEsNTv_gflQO3IWyA.png](https://miro.medium.com/max/1400/1*Zva2UdEsNTv_gflQO3IWyA.png)

### 🚅 Ansible Configuration Locations

In ubuntu-c,

```bash
> ansible --version

ansible [core 2.11.2]
  config file = None
  configured module search path = ['/home/ansible/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.8/dist-packages/ansible
  ansible collection location = /home/ansible/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible
  python version = 3.8.5 (default, May 27 2021, 13:30:53) [GCC 9.3.0]
  jinja version = 3.0.1
  libyaml = True

# This is the ansible core version

# Ansible package version by looking at python packages installed
pip3 freeze | grep ansible

ansible==4.1.0
ansible-core==2.11.2
```

Here `config file = None`, there are no configuration files set up,

**Possible ansible file locations**

- In `etc/ansible/`, (Lowest priority location)

```bash
# default location when ansible installed with package managers
/etc/ansible/ansible.cfg
```

After creating the file and path ansible will refer to that file when doing a `ansible --version`,

- In `~/.ansible.cfg` (ansible conf file in home directory)

```bash
# create one using
touch ~/.ansible.cfg
```

This will be the present configuration file taken by ansible as default,

- in the current directory `./ansible.cfg`

```bash
# If a cfg file in current path it willbe used default
touch tmp/ansible.cfg

# Note that this is not hidden
```

This is preferred, so that each of the dir got its own ansible config file,

- Ultimate priority one `ANSIBLE_CONFIG` (Env. variable with a filename target)

```bash
# create an ansible config file with any name and set path as env variable value
touch example_ansible.cfg
export ANSIBLE_CONFIG=/home/ansible/example_ansible.cfg

# unset the variable to change that
unset ANSIBLE_CONFIG
```

revert all the dummy configs created

## 👨🏾‍💻 Ansible Inventories

- Ansible connectivity - [ to centos (via `root`) and Ubuntu hosts (via `sudo`)].
- Inventory Host variables (***hostvars***)
- Simplification of inventory with ranges (***host rages***)
- Inventory Group variables (***groupvars***), Inventory children groups

Going with the fully hands on approach!!

```bash
ansible@ubuntu-c:~/diveintoansible/Ansible Architecture and Design/Inventories$ ls

01  02  03  04  05  06  07  08  09  10  11  12  13  14  15  16  challenge  template
```

Go with the `template`, for the hands on experience,

### 💽 INI Host file inventory

There is `ansible.cfg` and a `hosts` file are in the [INI format](https://en.wikipedia.org/wiki/INI_file#:~:text=An%20INI%20file%20is%20a,sections%20that%20organize%20the%20properties.) (that is also the php.ini, etc using).

There are different ways to write conf logic in `ansible.cfg` files, with `ini`, `yaml` or `json`.

```bash
cat hosts

[all]
centos1
```

as we specify `[all]`, every host is automatically assigned to `all` group in ansible,

- Ignore the host-key-check (fingerprint check in ansible connection),

```bash
ANSIBLE_HOST_KEY_CHECKING=False

ansible all -m ping

# recreated the ~/.ssh/known_hosts file
# Add the confg to ansible.cfg
```

- Adding all hosts (both `ubuntu` and `centos` to hosts file), by creating groups

```bash
# ping with groups
ansible centos -m ping
ansible ubuntu -m ping

# To get a condensed view
ansible all -m ping -o

# list all hosts in a centos group
ansible centos --list-hosts

# match with regular expressions
ansible ~.*3 --list-hosts

"~" -> Matching a pattern
".*3" -> any number of any chrtrs ending in 3
```

- Ansible needed `root` privileges in the hosts, ssh connect to remote system as the `root` user.

This can be done using a `hostvar` (host variable)

```bash
[centos]
centos1 ansible_user=root
```

- use `id` command to verify that it is connected to `root`, ***to run a command***

```bash
ansible all -m command -a 'id' -o

> centos1 | CHANGED | rc=0 | (stdout) uid=0(root) gid=0(root) groups=0(root)
```

- Having direct `root` access is convenient, but restricting direct root access is more common, more secure. So connect to the host as a normal user, then escalate to **super user**.
- This approach is common, so there is an inbuilt variable (hostvar) for this

    ```bash
    [ubuntu]
    ubuntu1 ansible_become=true ansible_become_pass=password

    # give the user su access with sudo password

    ansible all -a 'id' -o
    ```

- Dont need "-m command" module strict as ansible uses command module as default.

    ```bash
    ubuntu1 | CHANGED | rc=0 | (stdout) uid=0(root) gid=0(root) groups=0(root)

    # Changed From
    ubuntu1 | CHANGED | rc=0 | (stdout) uid=1000(ansible) gid=1000(ansible) groups=1000(ansible),27(sudo)
    ```

![ssh-list-hosts.png](Introduction%20and%20Setup%207a3606800cd04ede90df3527a97a639c/ssh-list-hosts.png)

- Changing the `sshd` standard port from port 22, (the hosts and ports and hostnames are configured in docker-compose `yaml` file)

    ```bash
    [centos]
    centos1 ansible_user=root ansible_port=2222

    # After changing the port in docker-compose file

    # Also this is possible
    [centos]
    centos1:2222 ansible_user=root
    ```

- Adding a control group and also adding the local control server together in the inventory, specifying an ansible connection type as local,

    ```bash
    [control]
    ubuntu-c ansible_connection=local
    ```

    Ansible connection is set to local rather than SSH for the `ubuntu-c`, which is the same system running ansible.

- Simplify the host-file using ranges,

    ```bash
    centos[2:3] ansible_user=root
    # centos2 and centos3
    ```

- Use of group-vars, to specify common properties of a group

    ```bash
    [centos:vars]
    ansible_user=root
    ```

- Also the mixture of `ubuntu` and `centos` groups can be grouped to a common `linux` group, (if there other OS groups to control, or just for Grouping correctly)

    ```bash
    [linux:children]
    centos
    ubuntu
    ```

***note: Specific host entries take precedence overall entries like,***

Final

```bash
[control]
ubuntu-c ansible_connection=local

[centos]
centos1 ansible_port=2222
centos[2:3]

[centos:vars]
ansible_user=root

[ubuntu]
ubuntu[1:3]

[ubuntu:vars]
ansible_become=true
ansible_become_pass=password
```

### 🏮 Using `yaml` format for inventory

- Change the `inventory` to `hosts.yaml`,

    ```bash
    [defaults]
    inventory = hosts.yml
    ```

- `hosts.yml`

    ```yaml
    ---
    control:
      hosts:
        ubuntu-c:
          ansible_connection: local
    centos:
      hosts:
        centos1:
          ansible_port: 2222
        centos2:
        centos3:
      vars:
        ansible_user: root
    ubuntu:
      hosts:
        ubuntu1:
        ubuntu2:
        ubuntu3:
      vars:
        ansible_become: true
        ansible_become_pass: password
    linux:
      children:
        centos:
        ubuntu:
    ...
    ```

    Use the `---` and `...` as start and end of `yaml` file, (not a strict one, nice anyway), yaml is covered detail later,

### 🦩 `JSON` Equivalent Inventory file

- Getting a `json` equivalent from a `yaml` file, can be done with a python oneliner,

    ```bash
    python3 -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin, Loader=yaml.FullLoader), sys.stdout, indent=4)' < hosts.yml > hosts.json
    ```

- That gives,

    ```json
    {
        "control": {
            "hosts": {
                "ubuntu-c": {
                    "ansible_connection": "local"
                }
            }
        },
        "centos": {
            "hosts": {
                "centos1": {
                    "ansible_port": 2222
                },
                "centos2": null,
                "centos3": null
            },
            "vars": {
                "ansible_user": "root"
            }
        },
        "ubuntu": {
            "hosts": {
                "ubuntu1": null,
                "ubuntu2": null,
                "ubuntu3": null
            },
            "vars": {
                "ansible_become": true,
                "ansible_become_pass": "password"
            }
        },
        "linux": {
            "children": {
                "centos": null,
                "ubuntu": null
            }
        }
    }
    ```

### 👽 Extra Info with inventories

The Inventory can also be specified with the Inventory, with `-i`

```json
ansible all -i hosts.yml --list-hosts
```

Use `EXTRA_VARS`, ie `-e` to override, inventory set variable temporary,

```json
ansible linux -m ping -o -e 'ansible_port=22'
```

That should fail on centos1, cz that listens on port `2222`,
