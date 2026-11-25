# RH294 Red Hat Enterprise Linux Automation with Ansible

Goal: automate provisioning, configuration, application deployment and orchestration with Ansible.

Prerequisite: RHCSA (EX200) or equivalent RHEL knowledge.

Course objectives:

- Automate common RHEL administration tasks with Ansible.
- Install and configure automation content navigator to run playbooks in a container-based execution environment.
- Create and manage inventories, prepare managed hosts for Ansible.
- Write effective playbooks.
- Reuse code with roles and Ansible Content Collections.

## Lab environment

| Machine                                    | Role                                  |
| ------------------------------------------ | ------------------------------------- |
| `workstation`                              | graphical workstation, where you work |
| `servera`, `serverb`, `serverc`, `serverd` | managed hosts (managed with Ansible)  |
| `utility`                                  | utility services for the classroom    |
| `bastion`                                  | gateway, must always run              |
| `classroom`                                | hosts the software and lab materials  |

- All machines are in `lab.example.com`. Users: `student` / `student`, `root` / `redhat`.
- Log in to `workstation`, then use `ssh` to reach the other machines.
- `DELETE` and `Reset` erase the disks. Keep auto-stop (max 11 hours) and auto-destroy (max 14 days) timers set while you work.

### Dev use

```
playbook in git -> same setup on every server   # like a Dockerfile, but for servers
```

- Same idea as infrastructure as code. Not from the course.
