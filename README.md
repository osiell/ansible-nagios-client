# Nagios client (including NSCA client)

Ansible role to configure a Debian or Ubuntu host monitored by Nagios (see the
*nagios-server* role).

Nagios plugins and the NSCA client are installed from the standard APT
repositories.
For passive monitoring purpose, you can deploy a Git or Mercurial repository
containing custom NSCA client scripts which will be executed by a crontab
(itself in the repository) with the `nsca_scripts_*` options. Such a repository
must contain a *crontab* file (which will be installed in the `/etc/cron.d`
directory) beside your custom scripts, e.g.:

    .
    ├── crontab
    ├── my_nsca_check_apt.sh
    └── my_nsca_check_users.sh

The *crontab* file could contain the following:

```
NSCA_HOST=192.168.1.10
NSCA_PORT=5667
*/30 *    * * *     root    /PATH/TO/REPO/my_nsca_check_apt.sh
*/5  *    * * *     root    /PATH/TO/REPO/my_nsca_check_users.sh
```

In this example, the NSCA_HOST and NSCA_PORT environment variables at the top
is used by each script to send data to the NSCA server with the
`/usr/sbin/send_nsca` binary.

## Supported Platforms

- Debian Wheezy (7)
- Ubuntu Trusty (14.04)

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
    - nsca_scripts_repo_type: git
    - nsca_scripts_repo_url: https://SERVER/MY_NSCA_SCRIPTS.git
    - nsca_scripts_repo_dest: /opt/nsca_scripts
```

The `/opt/nsca_scripts/crontab` file will be linked to
`/etc/cron.d/nsca_client`.

## Variables

```yaml
nsca_config_password: False     # Must be identical to the NSCA server
nsca_config_encryption_method: 1
nsca_scripts_repo_type: git     # git or hg
nsca_scripts_repo_url: False
nsca_scripts_repo_dest: /opt/nsca_scripts
nsca_scripts_repo_version: False
```
