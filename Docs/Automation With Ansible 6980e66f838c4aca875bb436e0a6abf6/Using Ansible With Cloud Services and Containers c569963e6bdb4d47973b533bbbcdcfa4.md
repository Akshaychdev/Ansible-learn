# Using Ansible With Cloud Services and Containers <!-- omit in toc -->

## 🥡 Table of Contents <!-- omit in toc -->

- [♟️ Using Ansible With AWS](#️-using-ansible-with-aws)
  - [🛫 Setting Up AWS for Accessing](#-setting-up-aws-for-accessing)
  - [👩🏼‍🏭 Setup Ansible for AWS Access](#-setup-ansible-for-aws-access)
  - [🪅 Using AWS Dynamic Inventories](#-using-aws-dynamic-inventories)
  - [👩🏼‍🎤 Checking and terminating the Instances](#-checking-and-terminating-the-instances)
- [🦈 Using Ansible With Docker](#-using-ansible-with-docker)
  - [🥣 Configuring the Docker](#-configuring-the-docker)
  - [⛴️ Docker Playbook to pull Images](#️-docker-playbook-to-pull-images)
  - [🌀 Spin UP Docker Containers](#-spin-up-docker-containers)
  - [🚣🏼‍♀️ Use `Dockerfile` to spin containers with playbook](#️-use-dockerfile-to-spin-containers-with-playbook)
  - [🏗️ Connect to Running Containers Via Docker With Ansible](#️-connect-to-running-containers-via-docker-with-ansible)
  - [❌ Remove Spawned containers and Images with Ansible](#-remove-spawned-containers-and-images-with-ansible)

**INTRODUCTION**

Very Important topics Discussing How to Spawn and manage AWS EC2 instances and Docker containers using Ansible.

## ♟️ Using Ansible With AWS

Topics Covered

- Configure Ansible For AWS Support
- Creating instances through Ansible with AWS and allocating IP addresses.
- Using AWS Dynamic Inventory
- Spinning Up the webapp using AWS
- Terminating and removing AWS instances.

### 🛫 Setting Up AWS for Accessing

The modules within ansible for AWS are built using popular `boto` and `boto3` libraries, therefore lot of functionalities for AWS present in it.

Use an AWS Free Tier Instance for checking it,

Steps to get Ansible ready to Connect with AWS

- After creating/logging in the account, create a separate user with EC2 permissions (best practise).
- From `EC2 Dashboard`, create `key pairs`, that are used to connect to EC2 instances with SSH.

    ![Untitled](Using%20Ansible%20With%20Cloud%20Services%20and%20Containers%20c569963e6bdb4d47973b533bbbcdcfa4/Untitled.png)

    ![Untitled](Using%20Ansible%20With%20Cloud%20Services%20and%20Containers%20c569963e6bdb4d47973b533bbbcdcfa4/Untitled%201.png)

- Create an API Access key pair for the New created User or for the default User,

![For the default user](Using%20Ansible%20With%20Cloud%20Services%20and%20Containers%20c569963e6bdb4d47973b533bbbcdcfa4/Untitled%202.png)

For the default user

![With Dedicated IAM user](Using%20Ansible%20With%20Cloud%20Services%20and%20Containers%20c569963e6bdb4d47973b533bbbcdcfa4/Untitled%203.png)

With Dedicated IAM user

And create a new API Access Key pairs and safely save them,

![Untitled](Using%20Ansible%20With%20Cloud%20Services%20and%20Containers%20c569963e6bdb4d47973b533bbbcdcfa4/Untitled%204.png)

- Need a VPC to use, if not present search for `VPC`, if default not present create a new one,(A [VPC](https://docs.aws.amazon.com/vpc/latest/userguide/configure-your-vpc.html) is a virtual network that closely resembles a traditional network that you'd operate in your own data center.)

    ![Untitled](Using%20Ansible%20With%20Cloud%20Services%20and%20Containers%20c569963e6bdb4d47973b533bbbcdcfa4/Untitled%205.png)

### 👩🏼‍🏭 Setup Ansible for AWS Access

Ansible AWS environments use `boto` and `boto3`, so we need to set environment variables that corresponds to API access and others AWS information for access,

```bash
# Set AWS env Variables
export AWS_ACCESS_KEY_ID='AKPXXXXXXXXXXXX'
export AWS_SECRET_ACCESS_KEY='77H888XXXXXXXXXXX_XXX'
```

- Install `boto` and `boto3`,

    ```bash
    sudo pip3 install boto boto3
    ```

- The playbook to create security group for this project,

    *(An AWS security group **acts as a virtual firewall for your EC2 instances to control incoming and outgoing traffic**. Both inbound and outbound rules control the flow of traffic to and traffic from your instance, respectively)*

    ```yaml
    ---
    -
      hosts: localhost
      connection: local
      gather_facts: false

      tasks:
        - name: Create a security group in AWS for SSH access and HTTP
          ec2_group:
             name: ansible
             description: Ansible Security Group
             region: us-east-1
             rules:
          # Allows traffic to port 80 for Web Server
                - proto: tcp
                  from_port: 80
                  to_port: 80
                  cidr_ip: 0.0.0.0/0
          # Allows SSH port 22
                - proto: tcp
                  from_port: 22
                  to_port: 22
                  cidr_ip: 0.0.0.0/0
    ...
    ```

- Now It is time to create instances, to get the instance type available in Free tier to use with ansible play, go to `EC2 Instances`> `Launch Instance`,

    For the AMI (Amazon Machine Image) - Look for free tier enabled

    ![Untitled](Using%20Ansible%20With%20Cloud%20Services%20and%20Containers%20c569963e6bdb4d47973b533bbbcdcfa4/Untitled%206.png)

    and copy the `ami` - no : `ami-068257025f72f470d`, Put the `instance_type`, `region`, `count`, `tag` - ()Important to identify the instance later), add capture those created instance IP’s and add to a host-group created,

    ```yaml
    - name: Provision a set of instances
          ec2:
             key_name: ansible
             group: ansible
             instance_type: t2.micro
             image: ami-068257025f72f470d
             region: us-east-1
             wait: true
             exact_count: 20
             count_tag:
                Name: AnsibleNginxWebservers
             instance_tags:
                Name: Ansible
          register: ec2

        - name: Add all instance public IPs to host group
          add_host:
            hostname: "{{ item.public_ip }}"
            groups: ansiblehosts
          with_items: "{{ ec2.instances }}"

        - name: Show group
          debug:
            var: groups.ansiblehosts
    ```

    → `ec2.instances`: Will be list with all of our instances,

    → `ansiblehosts`: Created a group in ansible with all of our VM’s dynamically

    → `instance_tags: Ansible`: That the applied AWS tag for each instance created.

    The EC2 instance after running the playbook will create 20 - Ec2 instances

    ![Untitled](Using%20Ansible%20With%20Cloud%20Services%20and%20Containers%20c569963e6bdb4d47973b533bbbcdcfa4/Untitled%207.png)

    If you rerun the playbook again, it will hit the `instance Limit Exceeded Error` (There will be an Instance Limit for Free Tier),

    A get-around for this error is to ignore the errors for the task,

    ```yaml
      - name: Provision a set of instances
          ec2:
            .......
          ignore_errors: true
    ```

    Now Nothing is going to register on `ec2` variable as instances already cleared and the next task will fail, As this fails the idempotent nature of play book - we need a better workaround for this,

    That will be with the use of AWS Dynamic Inventories,

### 🪅 Using AWS Dynamic Inventories

AWS Dynamic inventories are well written and maintained by the community,

To recall what is a dynamic inventory(with inventory scripts) in Ansible:- It is a python,PHP or any language script that lists changing hosts and supports cli `--list`.. etc methods, comes handy in cloud environments such as AWS where IP addresses change dynamically.

There is already a dynamic inventory with an associated `.ini` file available for our use-case(as it is common), the associated `ini` file can be copied to `/etc/ansible` or can set a `etc_ini_path` variable to the location.

There is also need to change a caching option that caches the API details for a certain amount of time, which is not needed in our case.

To GET and SET the dynamic inventory,

- Make a directory for the inventory,

    ```yaml
    mkdir inventory
    cd inventory
    ```

- Google for “ansible ec2 dynamic inventory github”, and download the `[ec2.py](http://ec2.py)` file to the directory, or [find it here](https://docs.ansible.com/ansible/latest/collections/amazon/aws/aws_ec2_inventory.html)

    ```bash
    wget https://raw.githubusercontent.com/ansible/ansible/stable-2.9/contrib/inventory/ec2.py
    wget https://raw.githubusercontent.com/ansible/ansible/stable-2.9/contrib/inventory/ec2.ini

    chmod +x ec2.py
    ```

- Make some changes to the `inventory` file,

    ```python
    # Change tp python3 env
    #!/usr/bin/env python3

    # Fix an issue with the script if exists
    # in line 172, comment the import line
    from ansible.module_utils import ec2 as ec2_utils
    ```

- Edit the `ec2.ini` file, pass `AWS_ACCESS_KEY` and `AWS_SECRET_KEY` in the `ec2.ini` file if needed at end,

    ```bash
    # Change the caching parameters
    cache_max_age = 0

    # export file path
    export EC2_INI_PATH=inventory/ec2.ini
    ```

- Edit the inventory file in the `ANSIBLE.CFG` configuration files too.
- to see the output, run `./ec2.py - - list`,

    ```bash
    inventory/ec2.py --list | more
    ```

    ![Untitled](Using%20Ansible%20With%20Cloud%20Services%20and%20Containers%20c569963e6bdb4d47973b533bbbcdcfa4/Untitled%208.png)

    Using the `_meta` method, to get and cycle efficiently,

    ```yaml
    # Add the refresh inventory line with playbook after provisioning
        - name: Provision a set of instances
          ec2:
            .................

        - name: Refresh inventory to ensure new instances exist in inventory
          meta: refresh_inventory
    ```

    Look for a property called `tag_Name_Ansible`, There is a group exists to target alla of our machines,

- Also, run `ansible all — — list-hosts` to see the available hosts.
- Edit the `ansible.cfg` file, enter the key file,

    ```bash
    [defaults]
    # Updated inventory path
    inventory = inventory/ec2.py
    host_key_checking = False
    # To handle 20 instances
    forks=20
    ansible_managed = Managed by Ansible - file:{file} - host:{host} - uid:{uid}
    ```

- Check the `group_vars` already created,

    ```bash
    cat group_vars/tag_Name_Ansible

    ---
    ansible_ssh_private_key_file: ~/.ssh/ansible.pem
    ansible_user: ec2-user
    ansible_become: true
    ...

    # Need to copy the created private key and set permissions
    chmod 600 ~/.ssh/ansible.pem

    # Check the key working by pinging the hosts
    ansible tag_Name_Ansible -m ping -o
    ```

- Now, run playbook that downloads the required packages into the instance and copy the code into the document root of the webserver.

    Added task to run the nginx and webapp role(from prev.)

    ```yaml
    -
      hosts: tag_Name_Ansible

      roles:
        - { role: webapp, target_dir: /usr/share/nginx/html }
    ```

### 👩🏼‍🎤 Checking and terminating the Instances

To add a `pause` in the play book that enables checking before terminating,

```yaml
-
  hosts: tag_Name_Ansible

  tasks:
    - debug:
        msg: "Check http://{{ ansible_host }}"

    - pause:
        prompt: "Verify service availability and continue to terminate"
```

- To remove and terminate the instances,

```yaml
    - name: Remove tagged EC2 instances from security group by setting an empty group
      ec2:
        state: running
        region: "{{ ec2_region }}"
        instance_ids: "{{ ec2_id }}"
        group_id: ""
      delegate_to: localhost

    - name: Terminate EC2 instances
      ec2:
        state: absent
        region: "{{ ec2_region }}"
        instance_ids: "{{ ec2_id }}"
        wait: true
      delegate_to: localhost
```

- To remove the security group created,

```yaml
-
  hosts: localhost
  connection: local
  gather_facts: false

  tasks:
  - name: Remove ansible security group
    ec2_group:
      name: ansible
      region: us-east-1
      state: absent
```

Double check the AWS console that all the running instances are now terminated and they wont develop any charges,

## 🦈 Using Ansible With Docker

- Configure the Docker Setup
- Pull Docker Images and Experiment With Docker
- Build Customised Containers and Images
- Use Ansible to connect to running Containers
- Terminate and Remove Docker Resources

### 🥣 Configuring the Docker

In the Lab setup comes with the course, docker is running in a separate container

```bash
# ping the running docker container
ping docker
```

1. Install docker and required python Docker Libraries, and we are using this local docker installation to connect to the running docker conatiner.

    ```bash
    # install_docker.sh
    sudo apt update
    sudo apt install -y docker.io
    pip3 install docker

    # Run the script
    bash -x install_docker.sh
    ```

2. Set up the docker host explicitly, it is required in our case as we are running docker separately in a container, this setup actually where to look for docker, which is not needed when docker running from localhost

    ```bash
    # cat envdocker
    export DOCKER_HOST=tcp://docker:2375

    # Source the file
    source envdocker
    ```

    Now the docker commands can be executed from the `ubuntu-c` console, `docker ps -a`.. etc.

### ⛴️ Docker Playbook to pull Images

We are using `docker_image` module to pull multiple images from a `with_items` list, as the docker is remote need an entry like `docker_host`

```yaml
-
  hosts: ubuntu-c

  tasks:
    - name: Pull images
      docker_image:
        docker_host: tcp://docker:2375
        name: "{{ item }}"
        source: pull
      with_items:
        - centos
        - ubuntu
        - redis
        - nginx
        # n.b. large image, >1GB
        - wernight/funbox
```

Now the `docker images`, will list the pulled images

```bash
REPOSITORY        TAG       IMAGE ID       CREATED         SIZE
redis             latest    3e42dd4e79c7   13 days ago     117MB
nginx             latest    b692a91e4e15   13 days ago     142MB
ubuntu            latest    df5de72bdb3b   13 days ago     77.8MB
centos            latest    5d0da3dc9764   11 months ago   231MB
wernight/funbox   latest    538c146646c3   4 years ago     1.12GB
```

Can Try some fun things with the downloaded `wernight/funbox` image,

```bash
docker run --rm -it wernight/funbox cmatrix
docker run --rm -it wernight/funbox nyancat
docker run --rm -it wernight/funbox asciiquarium
```

### 🌀 Spin UP Docker Containers

Using `docker_container` module to spin up containers from image,

```yaml
---
-
  hosts: ubuntu-c

  tasks:
    - name: Create an nginx container
      docker_container:
        docker_host: tcp://docker:2375
        name: containerwebserver
        image: nginx
        ports:
          - 80:80
        container_default_behavior: no_defaults
```

`containerwebserver` is the container name and it is from pulled `nginx` base image.

Bind local port `80` to docker container port `80`,

```bash
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                NAMES
d405afcbe6e0   nginx     "/docker-entrypoint.…"   16 seconds ago   Up 15 seconds   0.0.0.0:80->80/tcp   containerwebserver
```

To access this - Allow Port 1000 if running lab on VM(like Azure/GCP/AWS)

Then access [http://<VM PUBLIC IP>:1000/ubuntu1/](http://20.2.65.63:1000/ubuntu1/), to get the page loaded from the docker container.

### 🚣🏼‍♀️ Use `Dockerfile` to spin containers with playbook

To create a `Dockerfile` that can be used to build custom images,

```yaml
    - name: Create a customised Dockerfile
      copy:
        dest: /shared/Dockerfile
        mode: 0644
        content: |
          FROM nginx
```

It just got the single line that describes the base image, Task to build a customised image using this `Dockerfile`,

```yaml
  - name: Build a customised image
      docker_image:
        docker_host: tcp://docker:2375
        name: nginxcustomised:latest
        source: build
        build:
          path: /shared
          pull: yes
        state: present
        force_source: yes

  # Container from the customized Image
   - name: Create an nginxcustomised container
      docker_container:
        docker_host: tcp://docker:2375
        name: containerwebserver
        image: nginxcustomised:latest
        ports:
          - 80:80
        container_default_behavior: no_defaults
        recreate: yes
```

As there is Zero modifications to the base image, the container ids gonna look same as first built one,

```yaml
nginxcustomised   latest    b692a91e4e15   13 days ago     142MB
nginx             latest    b692a91e4e15   13 days ago     142MB
```

To customise this lets change the contents and build now the index page changed,

```yaml
  - name: Create a customised index.html
      copy:
        dest: /shared/index.html
        mode: 0644
        content: |
          Customised page for nginxcustomised

    - name: Create a customised Dockerfile
      copy:
        dest: /shared/Dockerfile
        mode: 0644
        content: |
          FROM nginx
          COPY index.html /usr/share/nginx/html/index.html
```

### 🏗️ Connect to Running Containers Via Docker With Ansible

Ansible treats running docker containers like targets,

```yaml
-
  hosts: ubuntu-c

  tasks:
    - name: Pull python image
      docker_image:
        docker_host: tcp://docker:2375
        name: python:3.8.5
        source: pull

    - name: Create 3 python containers
      docker_container:
        docker_host: tcp://docker:2375
        name: "python{{ item }}"
        image: python:3.8.5
        container_default_behavior: no_defaults
        command: sleep infinity
      with_sequence: 1-3
-
  hosts: containers
  gather_facts: False
  tasks:
    - name: Ping containers
      ping:
...
```

There is 2 major requirement for that,

1. The container needs python installed.
2. The container needs to be in running state.

We are confirming that with the first 2 tasks, pulls a python image and builds the container and runs `sleep infinity` on them,

Need to modify the `hosts` file to add these containers,

```bash
[containers]
python[1:3] ansible_connection=docker ansible_python_interpreter=/usr/bin/python3
```

→ `ansible_connection=docker`: Specify Ansible connection is made via docker

→ `ansible_python_interpreter=/usr/bin/python3`: use python3 to avoid errors

This is a useful feature to checkout

### ❌ Remove Spawned containers and Images with Ansible

Remove all containers and images to make it clean,

```yaml
-
  hosts: ubuntu-c

  tasks:
    - name: Remove old containers
      docker_container:
        docker_host: tcp://docker:2375
        name: "{{ item }}"
        state: absent
        container_default_behavior: no_defaults
      with_items:
        - containerwebserver
        - python1
        - python2
        - python3

    - name: Remove images
      docker_image:
        docker_host: tcp://docker:2375
        name: "{{ item }}"
        state: absent
      with_items:
        - centos
        - ubuntu
        - redis
        - nginx
        - wernight/funbox
        - nginxcustomised
        - python:3.8.5

    - name: Remove files
      file:
        path: "{{ item }}"
        state: absent
      with_items:
        - /shared/Dockerfile
        - /shared/index.html
```
