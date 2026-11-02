# RH124 - Chapter 12: Install and Update Software Packages

---

## 1. Register a system

- Subscription gives access to Red Hat repositories (CDN): updates, security patches, support, Insights.
- SCA (Simple Content Access): the subscription is checked per organization, not per system.
- Manage subscriptions in the Hybrid Cloud Console (`console.redhat.com`).

```bash
rhc connect                                  # preferred (RHEL 8.8+)
rhc connect --activation-key=KEY --organization=ORG_ID   # no password
rhc disconnect

subscription-manager register                # older systems
subscription-manager register --activationkey KEY --org ORG_ID
subscription-manager status
subscription-manager unregister
insights-client --register                   # Insights (not done by subscription-manager)
```

- Activation key = pre-shared token. More secure and good for automation.
- `rhc` enables: content, analytics, remote-management.
- Kickstart: `rhsm` command (needs activation key + org ID).
- System purpose: role, service level, usage (`subscription-manager syspurpose ...`).

| Path                       | Meaning                       |
| -------------------------- | ----------------------------- |
| `/etc/pki/product-default` | installed Red Hat products    |
| `/etc/pki/consumer`        | identifies the system         |
| `/etc/pki/entitlement`     | authenticates to repositories |

- `rct cat-cert file.pem` reads a certificate.

## 2. RPM packages

- RPM is the package format. It tracks files, checks dependencies, and has a local database.
- File name: `name-version-release.arch.rpm` (e.g. `coreutils-8.32-31.el9.x86_64.rpm`).
- A package has: files, metadata (name, version, dependencies, changelog), scripts.
- Packages are signed with GPG. A wrong signature = install fails.
- An update replaces the old version with a full new package. Only `kernel` keeps several versions.

```bash
rpm -qa                          # all installed packages
rpm -qf /etc/ssh/ssh_config      # which package owns this file
rpm -q dnf                       # installed version
rpm -qi dnf                      # details
rpm -ql dnf                      # files of the package
rpm -qc openssh-clients          # config files only
rpm -qd openssh-clients          # docs only
rpm -q --scripts openssh-server  # install/remove scripts
rpm -q --changelog audit         # change log
rpm -qlp file.rpm                # files in a downloaded (not installed) package
rpm -ivh file.rpm                # install a local package
```

### Extract a package without installing

```bash
rpm2cpio file.rpm | cpio -idv                  # extract everything here
rpm2cpio file.rpm | cpio -id "*/etc/x/y.conf"  # one file
rpm2cpio file.rpm | cpio -tv                   # only list files
```

- `-p` = work with a package file, not the installed one.
- Careful with third-party RPMs: install scripts run as root.

---

rpm -qf /usr/bin/python3 # какому пакету принадлежит этот файл
rpm -ql nginx # что установил пакет
rpm -qc nginx # где его конфиги

## 3. Install and update with DNF

- DNF replaced YUM in RHEL 9. `yum` is a link to `dnf`.
- `rpm` does not use repositories or resolve dependencies. `dnf` does.

```bash
dnf list 'http*'                # installed and available packages
dnf search KEYWORD              # name + summary
dnf search all 'web server'     # also the full description
dnf info httpd                  # package details
dnf provides /var/www/html      # which package gives this path

sudo dnf install httpd          # install (with dependencies)
sudo dnf update                 # update everything
sudo dnf update httpd           # update one package
sudo dnf remove httpd           # remove (and packages that need it!)

dnf group list                  # groups
dnf group info "RPM Development Tools"
sudo dnf group install "RPM Development Tools"

dnf history                     # past transactions
sudo dnf history undo 6         # undo transaction 6
```

- `dnf remove` also removes packages that depend on it. Read the list.
- Group packages: mandatory and default are installed, optional are not.
- Log: `/var/log/dnf.rpm.log`.
- Kernel: several versions can be installed. `uname -r` = running kernel.

### Modules (AppStream)

- BaseOS = core OS. AppStream = applications (modules and normal RPMs).
- A module has streams (versions). Only one stream can be enabled. A profile is a set of packages for a use case.

```bash
dnf module list                 # available modules
dnf module list NAME            # streams of a module
dnf module info NAME            # details (add :stream or --profile)
dnf module provides PACKAGE     # which module gives this package
```

---

sudo dnf search nginx # найти пакет
sudo dnf install nginx # установить
sudo dnf provides '\*/bin/curl' # какой пакет даёт команду (шаблон в кавычках)
sudo dnf history # что и когда ставили

## 4. DNF repositories

```bash
dnf repolist all                                  # all repositories and status
sudo dnf config-manager --enable REPO_ID          # enable (permanent)
sudo dnf config-manager --disable REPO_ID         # disable (permanent)
sudo dnf config-manager --add-repo="https://example.com/repo/"   # add a repo

sudo rpm --import https://example.com/RPM-GPG-KEY   # import GPG key FIRST
sudo dnf install https://example.com/epel-release-latest-9.noarch.rpm

sudo dnf install PACKAGE --enablerepo=PATTERN     # enable for ONE command only
sudo dnf install PACKAGE --disablerepo=PATTERN
```

- Repository files: `/etc/yum.repos.d/*.repo` (recommended) or `/etc/dnf/dnf.conf`. `.repo` files win.
- A `.repo` file:

```
  [myrepo]
  name=My repository
  baseurl=https://example.com/repo/
  enabled=1
  gpgcheck=1
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-example
```

- `enabled=0` = defined but not used by default.
- Import the GPG key before installing signed packages. `--nogpgcheck` is dangerous (fake or bad packages).
- With SCA, a system can use any repository from the organization's subscriptions.

---

dnf repolist # какие репозитории подключены
sudo dnf install epel-release # EPEL на RHEL-подобных системах (не из курса, так делают на Rocky или Alma)
sudo dnf install nginx --enablerepo=epel # включить репозиторий на один раз
