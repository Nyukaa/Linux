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

## 2. Installing Ansible

| Source                                    | Support                                                                                |
| ----------------------------------------- | -------------------------------------------------------------------------------------- |
| Community (upstream, `pip`)               | none from Red Hat                                                                      |
| `ansible-core` RPM in RHEL 9 AppStream    | limited (only content from Red Hat: `rhel-system-roles`, Insights playbooks, OpenSCAP) |
| Red Hat Ansible Automation Platform (AAP) | full (this course: AAP 2.2, Ansible Core 2.13)                                         |

| AAP component                                      | Purpose                                                               |
| -------------------------------------------------- | --------------------------------------------------------------------- |
| Ansible Core                                       | language and basic tools, modules only from `ansible.builtin`         |
| Content Collections                                | modules, roles, plug-ins by topic (120+ certified)                    |
| Automation content navigator (`ansible-navigator`) | replaces `ansible-playbook` and others, runs playbooks in a container |
| Execution environment                              | container image with Ansible Core, collections and dependencies       |
| Automation controller                              | central control: web UI and REST API                                  |
| Automation hub                                     | `console.redhat.com`, certified collections                           |

Control node (needs Python 3.8+ and an AAP subscription):

```bash
sudo dnf install ansible-navigator
ansible-navigator --version
podman login registry.redhat.io
podman pull registry.redhat.io/ansible-automation-platform-22/ee-supported-rhel8:latest
ansible-navigator images
```

- `sudo dnf install ansible-core` only if you need `ansible-playbook` (runs on the control node, no container).
- The classroom is preconfigured with `ee-supported-rhel8`.

Managed hosts:

- No agent. Need SSH, `sudo` (if not root), Python on Linux/UNIX.
- SELinux enabled: install `python3-libselinux`.
- Windows: PowerShell 3.0+, .NET 4.0+, WinRM (`ansible.windows` collection).
- Network devices: modules run on the control node (no Python on the device).

### Dev use

```bash
ansible-navigator images   # which execution environments do I have locally?
```

- The same execution environment runs the playbook on a laptop, in CI and in controller. Not from the course.

References: Ansible Automation Platform Installation Guide, Simple Content Access.

## Summary

- Automation reduces human error and keeps infrastructure consistent.
- Ansible is open source, agentless (only the control node and execution environments have the software), connects over SSH.
- Playbooks are readable text files describing the desired state.
- AAP is the fully supported version. `ansible-navigator` is the key tool to develop and run automation.
