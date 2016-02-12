# Nagios client (including NSCA client)

Ansible role to configure a Debian or Ubuntu host monitored by Nagios (see the
*nagios-server* role).

Nagios plugins and the NSCA client are installed from the standard APT
repositories.
For passive monitoring purpose, you can deploy a Git and Mercurial repositories
containing custom scripts which can be executed by a crontab (itself in the
repository) with the `nsca_scripts_repo_git` and `nsca_scripts_repo_hg`
options. Such a repository can contain a *crontab* file (which will be
installed in the `/etc/cron.d` directory) beside your custom scripts, e.g.:

    .
    ├── crontab
    ├── check_apt.sh
    └── check_users.sh

The *crontab* file could contain the following:

```
NSCA_HOST=192.168.1.10
NSCA_PORT=5667
*/30 *    * * *     root    /PATH/TO/REPO/check_apt.sh
*/5  *    * * *     root    /PATH/TO/REPO/check_users.sh
```

In this example, the NSCA_HOST and NSCA_PORT environment variables at the top
is used by each script to send data to the NSCA server with the
`/usr/sbin/send_nsca` binary.

## Supported Platforms

* Debian
    - Wheezy    (7)
    - Jessie    (8)
* Ubuntu
    - Trusty    (14.04)

## Example (Playbook)

Basic installation, the Nagios plugins can be used through SSH or NRPE (active
monitoring):

```yaml
- name: Any host
  hosts: monitored_host
  sudo: yes
  roles:
    - nagios-client
```

Installation with a custom collection of scripts to use with NSCA (passive
monitoring):

```yaml
- name: Any host
  hosts: monitored_host
  sudo: yes
  roles:
    - nagios-client
  vars:
    - nsca_config_password: PASSWORD
    - nsca_scripts_repo_git:
        - repo: https://example.com/MY_NSCA_SCRIPTS.git
          dest: /opt/nsca_scripts
          crontab_src: crontab
          crontab_dest: nsca_checks
```

The `/opt/nsca_scripts/crontab` file will be linked to
`/etc/cron.d/nsca_checks`.

Installation with a custom collection of scripts (without a crontab file) from
one repository, and the crontab file in another one:

```yaml
- name: Any host
  hosts: monitored_host
  sudo: yes
  roles:
    - nagios-client
  vars:
    - nsca_config_password: PASSWORD
    - nsca_scripts_repo_git:
        - repo: https://example.com/NSCA_SCRIPTS_COMMON.git
          dest: /opt/nsca_scripts_common
        - repo: https://example.com/MY_CRONTAB.git
          dest: /opt/nsca_scripts
          crontab_src: crontab
          crontab_dest: nsca_checks
```

## Variables

```yaml
nsca_config_password: False     # Must be identical to the NSCA server
nsca_config_encryption_method: 1
nsca_scripts_repo_type: git     # git or hg
nsca_scripts_repo_url: False
nsca_scripts_repo_dest: /opt/nsca_scripts
nsca_scripts_repo_version: False
```
