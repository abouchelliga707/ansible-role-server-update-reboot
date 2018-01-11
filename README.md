# ansible-role-server-update-reboot

Ansible role to update server to latest packages, reboot server, and wait for the server to start up. Add more roles after this to continue installing/configuring server.  
Can also exclude packages from being updated, only update specified packages, or install specified packages.  
Works with Redhat/CentOS and Ubuntu.  

Can be used to update packages for [Meltdown/Spectre Mitigation](#spectremeltdown-mitigation) for Redhat/CentOS 7 and Ubuntu 16.04  

Note:  
This role can reboot the server if there is a kernel update and if the reboot variable is true (reboot is default setting).


Distros tested
------------

* Ubuntu 16.04 - It should work on older versions of Ubuntu/Debian based systems.
* CentOS 7.4


Group Variables
------------

./group_vars/centos-dev/proxy.yml  
With a proxy:
```
proxy_env:
  http_proxy: http://my.internal.proxy:80
  https_proxy: https://my.internal.proxy:80
```

With no proxy:
```
proxy_env: []
```


Default Settings
------------

- **debug_enabled_default**: true|false (default false)
- **update_default**: true|false (default true)
- **reboot_default**: true|false (default true)
- **server_update_reboot_wait**: true|false (default true)
- **server_update_ssh_port**: SSH port of server (default 22)

Variables for RHEL/CentOS:
- **server_update_yum_exclude_pkgs**: comma separated string of packages to exclude from update. Can use wildcards. (default [])
- **server_update_yum_install_pkgs**: comma separated string of packages to ONLY update. Can use wildcards. (default '*' meaning all packages)

Variables for Ubuntu:
- **server_update_apt_exclude_default**: true|false. set true if using exclude list below (default false)
- **server_update_apt_exclude_pkgs**: List of packages to not update (each on separate line). Can include wildcard (but use ^ to begin match or a lot will match) to match multiple packages. (default undefined)
- **server_update_apt_default**: full|update_specific|install (default full)
  - full: update all packages using "apt-get dist-upgrade"
  - update_specific: only update from list in variable server_update_apt_install_pkgs
  - install: only install from list in variable server_update_apt_install_pkgs
- **server_update_apt_install_pkgs**: List of packages to ONLY update or install (each on separate line). Can include wildcard to match multiple packages. (default undefined)


Example Playbook server-update-reboot.yml
------------

```
---
- hosts: '{{inventory}}'
  become: yes
  roles:
  - server-update-reboot
  - server-config-xyz
```


Prep
------------

- install ansible
- create keys
- ssh to client to add entry to known_hosts file
- configure client server authorized_keys
- run ansible commands

Usage
------------

Use all defaults to: update, reboot server, and wait for server to start up.
```
ansible-playbook server-update-reboot.yml --extra-vars "inventory=all-dev" -i hosts-dev
```

Same as above, but do not reboot server:
```
ansible-playbook server-update-reboot.yml --extra-vars "inventory=all-dev reboot_default=false" -i hosts-dev
```

Update all packages except package(s) specified (for RHEL):
```
ansible-playbook server-update-reboot.yml --extra-vars 'inventory=centos-dev server_update_yum_exclude_pkgs="mysql*, bash, openssh*"' -i hosts-dev
```

Only update (or install) specific packages (for RHEL):
```
ansible-playbook server-update-reboot.yml --extra-vars "inventory=centos-dev server_update_yum_install_pkgs='kernel-*, iwl*firmware, microcode_ctl, dracut'" -i hosts-dev
```

Update all packages except package(s) specified (for Ubuntu):
```
ansible-playbook server-update-reboot.yml --extra-vars 'inventory=ubuntu-dev server_update_apt_exclude_default=true' --extra-vars '{"server_update_apt_exclude_pkgs": [bash, openssl, ^mysql*, ^openssh*]}' -i hosts-dev
```

Only update specific packages (for Ubuntu):
```
ansible-playbook server-update-reboot.yml --extra-vars "inventory=ubuntu-dev server_update_apt_default=update_specific" --extra-vars "{'server_update_apt_install_pkgs': [linux-firmware, linux-generic, linux-headers-generic, linux-image-generic, intel-microcode, openssh*]}" -i hosts-dev
```

Only install specific packages (for Ubuntu). Be careful with wildcards:
```
ansible-playbook server-update-reboot.yml --extra-vars "inventory=ubuntu-dev server_update_apt_default=install" --extra-vars "{'server_update_apt_install_pkgs': [bash, openssh-server]}" -i hosts-dev
```


## Spectre/Meltdown Mitigation

To patch Redhat/CentOS 7 and Ubuntu 16.04, for Spectre and Meltdown (CVE-2017-5754, CVE-2017-5753, CVE-2017-5715)  


### For Redhat/CentOS
```
ansible-playbook server-update-reboot.yml --extra-vars "inventory=centos-dev server_update_yum_install_pkgs='kernel-*, iwl*firmware, microcode_ctl, dracut'" -i hosts-dev
```

### For Ubuntu 16.04
```
ansible-playbook server-update-reboot.yml --extra-vars "inventory=ubuntu-dev server_update_apt_default=update_specific" --extra-vars "{'server_update_apt_install_pkgs': [linux-firmware, linux-generic, linux-headers-generic, linux-image-generic, intel-microcode]}" -i hosts-dev
```


## Notes
### RHEL5
RHEL/CentOS 5 has a dependency that needs to be installed: python-simplejson  
This command will use the raw module to install it:
```
ansible centos5 -m raw -a "yum install -y python-simplejson" --become --ask-pass --become-method=su --ask-become-pass --extra-vars="ansible_ssh_user=username123" -i hosts-dev
```

### SELinux
If SELinux is enabled/permissive a dependency is needed: libselinux-python  
This command will use the raw module to install it:
```
ansible centos5 -m raw -a "yum install -y libselinux-python" --become --ask-pass --become-method=su --ask-become-pass --extra-vars="ansible_ssh_user=username123" -i hosts-dev
```