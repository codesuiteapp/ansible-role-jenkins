# Ansible Role: Jenkins CI

Installs Jenkins CI on RHEL/CentOS and Debian/Ubuntu servers.

## Requirements

Requires `docker` to be installed on the server.

## Role Variables

    dkr_username: "_your_username_"
    dkr_passwd: "_p@ssw0rd1"
    passwd_adm_user: "asdfg1"

## Example Playbook

```yml
- name: Install Jenkins Server
  hosts: localhost
  connection: local

vars:
    today: "{{ lookup('pipe', 'date +\"%Y%m%d\"') }}"
    dkr_username: "_your_username_"
    dkr_passwd: "_p@ssw0rd1"
    passwd_adm_user: "P@ssw0rd1"

  # vars_files:
  #   - "aval_vars.yml"

  roles:
    - { role: codesuiteapp.jenkins }
```

## License

GPL-2.0-or-later

## Author Information

