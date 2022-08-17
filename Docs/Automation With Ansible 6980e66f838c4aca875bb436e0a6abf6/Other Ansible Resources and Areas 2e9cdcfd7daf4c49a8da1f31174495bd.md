# Other Ansible Resources and Areas <!-- omit in toc -->

## 🥡 Table of Contents <!-- omit in toc -->

- [😵 Troubleshooting Ansible](#-troubleshooting-ansible)
  - [📶 Checking SSH connectivity](#-checking-ssh-connectivity)
  - [🎢 Playbook Checks and Troubleshooting](#-playbook-checks-and-troubleshooting)
  - [📠 Log Levels when running ansible playbook](#-log-levels-when-running-ansible-playbook)
- [🛐 Best Practices with Ansible](#-best-practices-with-ansible)

## 😵 Troubleshooting Ansible

Need to check the following,

- SSH Connectivity
- Syntax Check
- Step - Step Over Individual Tasks
- Start At
- Log Path
- Verbosity

One of the beast means of troubleshooting ansible is to use `debug` module, but to trouble shoot connection problem to host.

### 📶 Checking SSH connectivity

To connect to host `ubuntu-1` from `ubuntu-c`, but if there is some issues with SSH connection, the common one is wrong permissions to `.ssh` directory and `authorized_keys`,

```bash
# Explicitly Change Permission in authorized_keys file
chmod 777 ~/.ssh/authorized_keys

# For troubleshooting the connection try the verbose option for ssh
ssh -v ubuntu1

# Also debug from the remote side
```

To run the `ssh` demon with a specific port after accessing the remote side,

```bash
/usr/sbin/sshd -d -p 1234

# Now connet to this port from the problematic side
ssh ubuntu1 -p 1234
```

Then goes back to host side (where ssh demon initiated), now the log shows some error about the connections requests hit.

![Untitled](Other%20Ansible%20Resources%20and%20Areas%202e9cdcfd7daf4c49a8da1f31174495bd/Untitled.png)

### 🎢 Playbook Checks and Troubleshooting

- Syntax Check

    ```bash
    # run with syntacx check
    ansible-playbook blocks_playbook.yaml --syntax-check
    # No errors when output is correct
    ```

- Step through the playbook,

    ```bash
    ansible-playbook blocks_playbook.yaml --step
    ```

    ![Untitled](Other%20Ansible%20Resources%20and%20Areas%202e9cdcfd7daf4c49a8da1f31174495bd/Untitled%201.png)

    This is helpful when picking tasks from set of tasks (other than using tags)

- Go to a certain task

    ```bash
    ansible-playbook blocks_playbook.yaml --start-at-task="Task---Name"
    ```

- Logging Ansible Exception,

    By default Ansible doesn’t log exceptions, but it can make ansible log those by specifying the log path in `ansible.cfg`,

    ```bash
    [defaults]
    inventory=hosts
    forks=6
    log_path=log.txt
    ```

    Now the corresponding output to the tasks run can be seen in `log.txt`,

### 📠 Log Levels when running ansible playbook

Ansible got logging with verbosity levels ranging from 1-4

```bash
-v - output data displayed
-vv - o/p and i/p displayed
-vvv - Additional info about connections to host
-vvvv - extra verbosity includes connection verbosity and scripts
```

Highest verbosity Example,

## 🛐 Best Practices with Ansible

The [Documentation to follow best practices](https://docs.ansible.com/ansible/2.8/user_guide/playbooks_best_practices.html)- [https://docs.ansible.com/ansible/2.8/user_guide/playbooks_best_practices.html](https://docs.ansible.com/ansible/2.8/user_guide/playbooks_best_practices.html)

We’ve been following a lot of best practices as gone along

Main Points,

- Using WhiteSpaces - better use of blank lines and white spaces
- Naming all Tasks -
- Usage of pin-point comments
- Use Dynamic inventories with clouds - AWS etc, create dynamic inventories use AWS ones as examples
- Update in batches of machines -
- Handle OS and Distro differences - usage of ansible - distribution
- Adding comments to the variables - where they come from
