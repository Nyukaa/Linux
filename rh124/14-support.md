# RH124 - Chapter 14: Analyze Servers and Get Support

---

## 1. Web console (Cockpit)

- Web interface to manage and monitor RHEL. Logs, graphs, services, network, users, and a terminal in the browser.

```bash
sudo dnf install cockpit
sudo systemctl enable --now cockpit.socket
sudo firewall-cmd --add-service=cockpit --permanent    # only with a custom firewall
sudo firewall-cmd --reload
```

- Open `https://servername:9090`. The self-signed certificate gives a browser warning.
- Login: any local user. Default is **limited access**. Privileged users switch to **administrative access** (asks for the password).

| Page               | What it does                                         |
| ------------------ | ---------------------------------------------------- |
| Overview           | system info, performance, power (reboot / shut down) |
| Logs               | filter by date and priority                          |
| Terminal           | full terminal in the browser                         |
| Services           | start, stop, restart, mask                           |
| Networking         | interfaces, IP addresses, firewall                   |
| Accounts           | create users, change passwords                       |
| Diagnostic Reports | Create report, then Download report                  |

- A non-privileged user can only change their own password and manage their SSH keys.
- A diagnostic report collects logs, configuration and system info. It needs a privileged user.

## 2. Diagnostics report and support

- Red Hat Customer Portal: `access.redhat.com` (docs, Knowledgebase, support cases, downloads, security advisories, CVE).
- Hybrid Cloud Console: `console.redhat.com`.

### Prepare a support case

1. Describe the problem: symptoms and steps to reproduce.
2. Background: product and version, `sos report`, for kernel problems a `kdump` crash dump.
3. Severity.

| Severity   | Meaning                                                 |
| ---------- | ------------------------------------------------------- |
| Urgent (1) | production down or data lost, no workaround             |
| High (2)   | production works but is severely limited, no workaround |
| Medium (3) | partial loss of use, workaround exists                  |
| Low (4)    | question, docs error, feature request                   |

- After an Urgent or High case, also call the support center.

### sos report

- Collects: kernel version, loaded modules, config files, command output, installed packages.

```bash
sudo dnf install sos
sudo sos report                 # creates /var/tmp/sosreport-....tar.xz
ls -l /var/tmp/
sudo sos clean /var/tmp/sosreport-....tar.xz    # hide user names, host names, IPs, MACs
sudo sos report --upload        # send to Red Hat support
```

- `sos clean` makes a new `...-obfuscated.tar.xz`. Keep the mapping file private.
- `sos collect` gathers reports from many nodes.
- Web console: Diagnostic Reports, Run report, Download (needs a privileged user).

## 3. Red Hat Insights

- Insights = predictive analytics as SaaS. Finds security, performance, availability and stability issues. Console: `console.redhat.com/insights`.
- An agent sends configuration metadata over TLS (a subset of what `sos report` collects).
- Fixes come with a risk estimate and an Ansible Playbook or step-by-step instructions.

```bash
sudo rhc connect -a ACTIVATION_KEY -o ORG_ID   # register + enable Insights
sudo insights-client --register                # register and upload first data
sudo insights-client                           # refresh metadata
```

| Service       | What it does                                      |
| ------------- | ------------------------------------------------- |
| Advisor       | configuration issues (Total risk, Risk of change) |
| Vulnerability | CVEs and exposed systems                          |
| Compliance    | OpenSCAP policies (e.g. PCI DSS)                  |
| Patch         | advisories + Ansible Playbook to update RPMs      |
| Drift         | compare systems or one system over time           |
| Policies      | custom rules and alerts                           |

- Inventory = list of registered systems (column "Last seen").
- Remediations = list of generated Ansible Playbooks.
- Package: `insights-client`.
