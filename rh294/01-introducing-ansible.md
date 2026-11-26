# Chapter 1. Introducing Ansible

Goal: understand the basic concepts of Ansible and install the development tools from Red Hat Ansible Automation Platform.

Sections:

1. Automating Linux Administration with Ansible
2. Installing Ansible

## 1. Automating Linux Administration with Ansible

- Manual administration causes mistakes and servers that drift apart. Automation makes deployment fast and consistent.
- Infrastructure as Code: the required state is described in readable text files in version control. Change the code, not the servers by hand.
- Ansible = open source automation language (Playbooks) and engine. Agentless: connects over SSH (default) or WinRM, runs modules, removes them after.

| Term            | Meaning                                               |
| --------------- | ----------------------------------------------------- |
| Control node    | machine where Ansible is installed                    |
| Managed hosts   | machines that Ansible manages                         |
| Inventory       | list of managed hosts and groups                      |
| Play / Playbook | tasks for hosts in order / YAML file with plays       |
| Module          | small program that puts the system in a desired state |

- Idempotent: running a playbook again on correct hosts changes nothing.
- A failed task stops the playbook for that host, other hosts continue.
- Declarative: describe the desired state, not the steps.
- Automation controller (formerly Ansible Tower): web UI and REST API, access control, shared SSH credentials, job log.

### Dev use

```yaml
- name: nginx is installed
  ansible.builtin.dnf:
    name: nginx
    state: present
```

- Same result on every run: it installs only if missing. Playbook syntax comes later, not from this section.

References: Ansible, How Ansible Works.
