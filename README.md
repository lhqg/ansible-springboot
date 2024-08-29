![GitHub Release (latest SemVer)](https://img.shields.io/github/v/release/lhqg/ansible-springboot)
[![License](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0.html)
[![GitHub Issues](https://img.shields.io/github/issues/lhqg/ansible-springboot)](https://github.com/lhqg/ansible-springboot/issues)
[![GitHub PR](https://img.shields.io/github/issues-pr/lhqg/ansible-springboot)](https://github.com/lhqg/ansible-springboot/pulls)
[![GitHub commit activity](https://img.shields.io/github/commit-activity/y/lhqg/ansible-springboot)](https://github.com/lhqg/ansible-springboot/commits/main)
[![GitHub Last commit](https://img.shields.io/github/last-commit/lhqg/ansible-springboot)](https://github.com/lhqg/ansible-springboot/commits/main)
![GitHub Downloads](https://img.shields.io/github/downloads/lhqg/ansible-springboot/total)

# Ansible roles to deploy Springboot applications on Linux

## Table of Contents

1. [Overview](#overview)
2. [Springboot foundation role](#springboot_foundation-role)
3. [Springboot instance role](#springboot_instance-role)
4. [Limitations - OS compatibility, etc.](#limitations)
5. [Development - Guide for contributing to the module](#development)
6. [Disclaimer](#disclaimer)

## Overview

These OS structures are:

* OS user and group,
* LVM LV and filesystems (optional),
* Directory trees
* systemd units
* packages

## springboot_foundation role

The `springboot_foundation` role prepares the OS to host one or multiple Springboot applications.

It creates structures such as user/group, LVM FS/LV, directories.
It ensures the SELinux and systemd packages for Springboot are installed.

### Directories

### Role variables

`sbf_user_id`: UID (default: 5000) for the `springboot` user which will run the Springboot process(es).
`sbf_group_id`: GID (default: 5000) for the `springboot` group.

`sbf_dedicated_lv_opt`: (_yes_|no) Whether the /opt/springboot directory tree should be created as a dedicated filesystem and logical volume,
`sbf_dedicated_vg_opt`: Name of the volume group to host the /opt/springboot filesystem.

`sbf_dedicated_lv_srv`: (_yes_|no) Whether the /srv/springboot directory tree should be created as a dedicated filesystem and logical volume,
`sbf_dedicated_vg_srv`: Name of the volume group to host the /srv/springboot filesystem.

`sbf_dedicated_lv_log`: (_yes_|no) Whether the /var/log/springboot directory tree should be created as a dedicated filesystem and logical volume,
`sbf_dedicated_vg_log`: Name of the volume group to host the /var/log/springboot filesystem.

`sbf_systemd_target_enabled`: (_yes_|no) Whether the `springboot.target` systemd unit should be enabled so that all Springboot applications are started at boot ime.

## springboot_instance role

The `springboot_instance` role deploys a specific Springboot application on the host.

See [springboot_instance role README](ansible-roles/ansible-role-springboot_instance/README.md)

## Limitations

Works only on Red Hat Linux and Red Hat like distributions.

## Development

See https://github.com/lhqg/ansible-springboot/CONTRIBUTING.md

## Disclaimer

This Ansible code is provided AS-IS. People and organisation
willing to use it must be fully aware that they are doing so at their own risks and
expenses.

The Author(s) of this Ansible roles SHALL NOT be held liable nor accountable, in
 any way, of any malfunction or limitation of said module, nor of the resulting damage, of
 any kind, resulting, directly or indirectly, of the usage of this Ansible roles.

It is strongly advised to always use the last version of the code, to check for the
compatibility of the platform where it is about to be deployed, to compile the module on
the target specific Linux distribution and version where it is intended to be used.

Finally, users should check regularly for updates.
