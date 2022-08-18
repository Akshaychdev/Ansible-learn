# [Automation With Ansible](https://www.udemy.com/share/103P2u3@GQB3U3-FB_bWJSd-9ZTiTmsyg3VeA2wb5of_ya7D2AApviIZ-uWLLVnVVtK5vDLp/) <!-- omit in toc -->

This Doc Content and Material adapted from the [Dive into Ansible Course by James Spurin](https://www.udemy.com/share/103P2u3@GQB3U3-FB_bWJSd-9ZTiTmsyg3VeA2wb5of_ya7D2AApviIZ-uWLLVnVVtK5vDLp/)

![Ansible Cover](Automation%20With%20Ansible%206980e66f838c4aca875bb436e0a6abf6/ansible.png)

## 🥡 Table of Contents <!-- omit in toc -->

- [🧩 Introduction and Setup](#-introduction-and-setup)
- [🚀 Ansible Modules and Playbooks](#-ansible-modules-and-playbooks)
- [🈹 Jinja2 and Template and Practice Project 1](#-jinja2-and-template-and-practice-project-1)
- [🚂 Ansible Playbook: Deep Dive](#-ansible-playbook-deep-dive)
- [🏗️ Structuring Ansible Playbooks](#️-structuring-ansible-playbooks)
- [🔂 Using Ansible With Cloud Services and Containers](#-using-ansible-with-cloud-services-and-containers)
- [🔌 Creating Ansible Modules and Plugins](#-creating-ansible-modules-and-plugins)
- [🦥 Other Ansible Resources and Areas](#-other-ansible-resources-and-areas)
- [👣 Google console Setup Commands](#-google-console-setup-commands)

## 🧩 Introduction and Setup

- General Introduction and config setup

[Introduction and Setup](Automation%20With%20Ansible%206980e66f838c4aca875bb436e0a6abf6/Introduction%20and%20Setup%207a3606800cd04ede90df3527a97a639c.md)

## 🚀 Ansible Modules and Playbooks

- Common modules and playbook setup

[Ansible Modules and Playbooks](Automation%20With%20Ansible%206980e66f838c4aca875bb436e0a6abf6/Ansible%20Modules%20and%20Playbooks%20f36d917b734a468e9cac76c0b496e8b2.md)

## 🈹 Jinja2 and Template and Practice Project 1

- About the template -  logic language used in Ansible

[Jinja 2, Templates & Practice Task 1](Automation%20With%20Ansible%206980e66f838c4aca875bb436e0a6abf6/Jinja%202,%20Templates%20&%20Practice%20Task%201%201fcc4ad752a040cbade1770613f624cc.md)

## 🚂 Ansible Playbook: Deep Dive

[👩🏾‍🎓 Ansible Playbooks - Deep Dive](Automation%20With%20Ansible%206980e66f838c4aca875bb436e0a6abf6/%F0%9F%91%A9%F0%9F%8F%BE%E2%80%8D%F0%9F%8E%93%20Ansible%20Playbooks%20-%20Deep%20Dive%202f3e15f8b9ff461386de1359c45f1fb2.md)

## 🏗️ Structuring Ansible Playbooks

[Structuring Ansible Playbooks](Automation%20With%20Ansible%206980e66f838c4aca875bb436e0a6abf6/Structuring%20Ansible%20Playbooks%20798161b1cc0047809ada760a74ac005e.md)

## 🔂 Using Ansible With Cloud Services and Containers

[Using Ansible With Cloud Services and Containers](Automation%20With%20Ansible%206980e66f838c4aca875bb436e0a6abf6/Using%20Ansible%20With%20Cloud%20Services%20and%20Containers%20c569963e6bdb4d47973b533bbbcdcfa4.md)

## 🔌 Creating Ansible Modules and Plugins

[Creating Ansible Modules and Plugins](Automation%20With%20Ansible%206980e66f838c4aca875bb436e0a6abf6/Creating%20Ansible%20Modules%20and%20Plugins%2024fd1c1e347041f3aea07fc107ad1b14.md)

## 🦥 Other Ansible Resources and Areas

[Other Ansible Resources and Areas](Automation%20With%20Ansible%206980e66f838c4aca875bb436e0a6abf6/Other%20Ansible%20Resources%20and%20Areas%202e9cdcfd7daf4c49a8da1f31174495bd.md)

## 👣 Google console Setup Commands

```bash
# clone the lab to home
git clone -b cloudshell-gcp \
    https://github.com/spurin/diveintoansible-lab.git \
    ${HOME}/diveintoansible-lab

# Generate SSH Keys
ssh-keygen -f \
    ${HOME}/diveintoansible-lab/config/guest_ssh \
    -P "" <<< y; cp -rf \
    ${HOME}/diveintoansible-lab/config/guest_ssh \
    ${HOME}/diveintoansible-lab/config/root_ssh; \
    cp -rf \
    ${HOME}/diveintoansible-lab/config/guest_ssh.pub \
    ${HOME}/diveintoansible-lab/config/root_ssh.pub

# Docker compose UP
cd ${HOME}/diveintoansible-lab; \
    bin/docker-compose up \
    --quiet-pull

# Docker pull to ubuntu-c
git clone https://github.com/spurin/diveintoansible.git
```
