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

`sb_user_id`: UID (default: 5000) for the `springboot` user which will run the Springboot process(es).
`sb_group_id`: GID (default: 5000) for the `springboot` group.

`sb_dedicated_lv_opt`: (_yes_|no) Whether the /opt/springboot directory tree should be created as a dedicated filesystem and logical volume,
`sb_dedicated_vg_opt`: Name of the volume group to host the /opt/springboot filesystem.

`sb_dedicated_lv_srv`: (_yes_|no) Whether the /srv/springboot directory tree should be created as a dedicated filesystem and logical volume,
`sb_dedicated_vg_srv`: Name of the volume group to host the /srv/springboot filesystem.

`sb_dedicated_lv_log`: (_yes_|no) Whether the /var/log/springboot directory tree should be created as a dedicated filesystem and logical volume,
`sb_dedicated_vg_log`: Name of the volume group to host the /var/log/springboot filesystem.

`sb_systemd_target_enabled`: (_yes_|no) Whether the `springboot.target` systemd unit should be enabled so that all Springboot applications are started at boot ime.

## springboot_instance role

The `springboot_instance` role deploys a specific Springboot application on the host.

See [springboot_instance role README](ansible-roles/ansible-role-springboot_instance/README.md)

### Role variables

`sb_appname`: Name of the Springboot application (only A-Z letters, 0-9 figures, - and _).
`sb_app_version`: Version of the Springboot application.
`sb_appjarname`: Name of the Springboot application JAR file.

`sb_listen_port`: (default: 8443) TCP port the application binds to and listens on, for SELinux port to be labelled.

`sb_memory_max_heapsize_mb`: (default: 768) Maximum heap size for the Java VM.
`sb_memory_min_heapsize_mb`: (default: 384) Minimum heap size for the Java VM.
`sb_memory_stacksize_kb`: (default: 220) Stack size for the Java VM.
`sb_memory_hugepages`: (yes|_no_) Whether the Java VM will use huge memory pages (2MiB pages instead of standard 4KiB).

`sb_i18n_lang`: (default: en_US.UTF-8) Value for the LANG environment variable for the Java VM.

`sb_enable_service`: (_yes_|no) Whether the systemd service unit for the Springboot application should be enabled (started at boot time).
`sb_startup_timeout_sec`: 10

`sb_java_version`: (default: 11) Version of the Java runtime.
`sb_java_flavour`: (default: openjdk) Flavour of the Java runtime.
`sb_java_extra_args`: (default: *empty*) Extra arguments for the Java VM.
`sb_java_tmpdir`: (default: /srv/springboot/*sb_appname*) Location for Java temporary files.

`sb_enable_dynlibs`: (true|_false_) Whether the Springboot appliction is allowed to create dynamic libraries or equivalent.

## Limitations

Works only on Red Hat Linux and Red Hat like distributions.

## Development

See https://github.com/hubertqc/ansible-springboot/CONTRIBUTING.md

## Disclaimer

This Ansible code is provided AS-IS. People and organisation
willing to use it must be fully aware that they are doing so at their own risks and
expenses.

The Author(s) of this Ansible roles SHALL NOT be held liable nor accountable, in
 any way, of any malfunction or limitation of said module, nor of the resulting damage, of
 any kind, resulting, directly or indirectly, of the usage of this Puppet module.

It is strongly advised to always use the last version of the code, to check for the
compatibility of the platform where it is about to be deployed, to compile the module on
the target specific Linux distribution and version where it is intended to be used.

Finally, users should check regularly for updates.
