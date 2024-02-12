ansible-role-springboot_instance
================================

An Ansible role that deploys Springboot an application instance.

Requirements
-------------

- The `ansible-role-springboot_foundation` ran successfully,
- The springboot application is being installed as a service,
- The application JAR file must be downloaded previously to `sb_workdir` by the playbook,
- The configuration files should be stored/downloaded to {{ playbook_dir }}/files/conf. Also, they should be templated.
- This Role requires "root" access to manage services and to perform SELinux labelling of resources.

Role Variables
---------------

- Variables which **MUST** be defined at the playbook level:
  - `sb_appname`: The name of the application (Only A-Z/a-z letters, 0-9 digits and _)
  - `sb_app_version`: Version of the application
  - `sb_appjarname`: The JAR application name

- Variables which **SHOULD NOT** be altered (unless you both have a fairly good reason and know exactly how to deal with the SELinux customisations that shall be needed):
  - `sb_root`: The top level directory for (all) Springboot applications static files on the host. (Default: "/opt/springboot")
  - `sb_srvroot`: The top level directory for (all) Springboot applications working files on the host. (Default: "/srv/springboot")
  - `sb_logroot`: The top level directory for (all) Springboot applications log files on the host. (Default: "/var/log/springboot")
  - `sb_service_name`: The name of the systemd service unit for the Springboot application (Default: "springboot@`sb_appname`")

IMPORTANT: for all `*_mode` variables, the value MUST be surrounded with quotes (") and MUST be specified with 4 octal digit.

- Default variables (from defaults/main.yml):
  - `sb_workdir`: The path where the artifacts are being downloaded on the target machines. (Default: "/var/tmp/springboot")
  - `sb_appbase`: The path where the app will reside. (Default: "`sb_root`/springboot/`sb_appname`")
  - `sb_appbase_mode`: The dir-mode permission for `sb_appbase`. (Default: "0750")
  - `sb_service_env_filename`: The name of the environment file for the systemd service unit. (Default: "`sb_root`/springboot/`sb_appname`/env")
  - `sb_jar_location`: The directory where the *JAR application* will be deployed. (Default: "`sb_appbase`/lib")
  - `sb_conf_src`: Application configuration files are expected to be Jinja2 templates stored in the playbook in "{{ playbook_dir }}/files/conf", they should set all needed application properties. (Default: "`playbook_dir`/files/conf" ie "{{ playbook_dir }}/files/conf")
  - `sb_appbase_conf_location`: The destination directory for the configuration files. (Default: "`sb_appbase`/conf")
  - `sb_java_security_file`: Location of the Java VM security properties files specific to the Springboot application. (Default: "`sb_appbase_conf_location`/java.security")

  - `sb_run_handlers`: Should the `Start Springboot instance` and `Cleanup workdir` handlers run automatically, or not. (Default: `yes`)
  - `sb_cleanup_entire_workdir`: Should remove the entire `sb_workdir` (not only the file `sb_workdir`/`sb_appjarname`), or not. Default value: "no". (**This works only if `sb_run_handlers` is set on "yes" ("true") !**)

  - `sb_listen_port`: TCP port the application binds to and listens on, for SELinux port to be labelled. (Default: 8443)
  - `sb_memory_max_heapsize_mb`: Maximum heap size (in MiB) for the Java VM.((Default: 768)
  - `sb_memory_min_heapsize_mb`: Minimum heap size (in MiB) for the Java VM.((Default: 384)
  - `sb_memory_stacksize_kb`: Stack size (in KiB) for the Java VM. (Default: 220)
  - `sb_memory_hugepages`: Whether the Java VM will use huge memory pages (2MiB pages instead of standard 4KiB). (Default: no)

  - `sb_i18n_lang`: Value for the LANG environment variable for the Java VM. (Default: "en_US.UTF-8")

  - `sb_enable_service`: Whether the systemd service unit for the Springboot application should be enabled (started at boot time). (Default: yes)
  - `sb_startup_timeout_sec`: Delay (in seconds) for the Springboot application to startup correctly. (Default: 10)

  - `sb_java_version`: Version of the Java runtime. (Default: 11)
  - `sb_java_flavour`: Flavour of the Java runtime. (Default: "openjdk")
  - `sb_java_extra_args`: Extra arguments for the Java VM, value is passed to the Java process by the systemd service unit environment file (`service-env.j2`jinja2 template). (Default: empty.)
  - `sb_java_tmpdir`:  Location for Java temporary files. (Default: "`sb_srvroot`/`sb_appname`")
  - `sb_log_symlink`: Whether a `logs`symbolic link should be created in `sb_appbase` to point to `sb_log_dir`. (Default: `no`)

  - `sb_enable_dynlibs`: Whether the Springboot appliction is allowed to create dynamic libraries or equivalent. (Default: false)

  - `sb_app_mode`: The dir & files mode permission for the JAR application destination location ("`sb_jar_location`/" & "`sb_jar_location`/`sb_appjarname`"). (Default: "0750")
  - `sb_conf_mode`: The dir & files mode permission for the `sb_appbase_conf_location` and each copied conf dir & file from `sb_conf_src` to `sb_appbase_conf_location`/. (Default: "0750")

  - `sb_log_dir_mode`: The dir mode permission for the log folder. (Default: "0750")

  - `sb_become`: Whether the role becomes `root` to create and manage Springboot application systemd units and to perform SELinux labelling of resources.
  - `sb_run_dir`: The destination directory where the service of the application may store transient files while the application runs. (Default: "`sb_appbase`/run")
  - `sb_run_dir_mode`: The dir mode permission for `sb_run_dir` where transient files will reside. (Default: "0750")

  - `sb_extra_files_dest`: The destination directory for additional optional files,such as libraries or binaries, that may be needed by the JAR application. (Default: `sb_jar_location`). (See the optional `sb_extra_files_src` variable described below in the section ["Other variables that may optionally be added/changed:"](#optional)))
  - `sb_extra_files_mode`: The dir & files modes of the destination folder for extra binary files that may be needed by the aplpication. (Default: "0750")
  - `sb_keystores_mode`: The mode of the directory & files to be populated from `sb_keystores_src` (local playbook path) to `sb_keystores_dest` (on the target host). (Default: "0750")

  - `sb_undeploy`: Triggers the un-deployment of the Springboot application. (Default value: `false`)
  - `sb_remove_extra_files`: Triggers the removal of files/directories under `sb_extra_files_dest` when `sb_undeploy` is set to `true`. (Default value: `false`)
  - `sb_remove_keystores`: Triggers the removal of files/directories under `sb_keystores_dest` when `sb_undeploy` is set to `true`. (Default value: `false`)

If you don't want the handlers to be run at the end of the role execution, you should set the variable for the `sb_run_handlers` to `false`. However, if you want to run the handlers afterwards, the first task that is executed after the role should set the value for this variable to true.

```yaml
  post_tasks:
    - set_fact:
        sb_run_handlers: yes
```

```yaml
---
---
# Definition of defaults file for ansible-role-springboot_instance

sb_become: yes

sb_root: /opt/springboot
sb_logroot: /var/log/springboot
sb_srvroot: "/srv/springboot"

sb_workdir: /var/tmp/springboot
<<<<<<< HEAD
sb_appbase: "{{ sb_root }}/springboot/{{ sb_appname }}"

sb_service_env_filename: "{{ sb_root }}/springboot/{{ sb_appname }}/env"
=======
sb_appbase: "/opt/springboot/{{ sb_appname }}"

sb_service_env_filename: "/opt/springboot/{{ sb_appname }}/env"
>>>>>>> 83accc7 (Improve documentation)

sb_jar_location: "{{ sb_appbase }}/lib"
sb_conf_src: "{{ playbook_dir }}/files/conf"
sb_appbase_conf_location: "{{ sb_appbase }}/conf"
sb_java_security_file: "{{ sb_appbase_conf_location }}/java.security"
sb_keystores_dest: "{{ sb_appbase }}/keys"
<<<<<<< HEAD
sb_log_dir: "{{ sb_logroot }}/{{ sb_appname }}"
sb_srv_dir: "{{ sb_srvroot }}/{{ sb_appname }}"
=======
sb_log_dir: "/var/log/springboot/{{ sb_appname }}"
sb_srv_dir: "/srv/springboot/{{ sb_appname }}"
>>>>>>> 83accc7 (Improve documentation)
sb_run_handlers: yes
sb_cleanup_entire_workdir: no
sb_extra_files_dest: "{{ sb_jar_location }}"
sb_appbase_mode: "0750"

sb_listen_port: 8443
#sb_monitoring_port: 

sb_memory_max_heapsize_mb: 768
sb_memory_min_heapsize_mb: 384
sb_memory_stacksize_kb: 220
sb_memory_hugepages: no

sb_i18n_lang: "en_US.UTF-8"

sb_enable_service: yes
sb_startup_timeout_sec: 10
<<<<<<< HEAD
=======
sb_app_log_file: "{{ sb_log_dir }}/{{ sb_appname }}-{{ sb_app_version }}.log"
>>>>>>> 83accc7 (Improve documentation)

sb_java_version: 11
sb_java_flavour: "openjdk"
sb_java_extra_args: ""
sb_java_tmpdir: "{{ sb_srv_dir }}"
sb_log_symlink: no

sb_enable_dynlibs: false

# dir permissions for "{{ sb_appbase }}/run/" used to store the running transient files
sb_run_dir: "{{ sb_appbase }}/run"
sb_run_dir_mode: "0750"

# dir/files permissions for "{{ sb_jar_location }}/" & "{{ sb_jar_location }}/{{ sb_appjarname }}"
# ( of course, the user running the application must be able to read)
sb_app_mode: "0750"

# dir/files permissions for "{{ sb_appbase_conf_location }}/" , "{{ sb_appbase_conf_location }}//each_copied_conf-dir/" & "{{ sb_appbase_conf_location }}//each_copied_conf-dir/each_copied_conf-file":
sb_conf_mode: "0750"

# dir permissions and retention for "{{ sb_log_dir }}/" used to store the running jar logs
sb_log_dir_mode: "0750"
sb_log_dir_retention_days: 30

# dir/files permissions for "{{ sb_extra_files_dest }}/" && "{{ sb_extra_files_dest }}/each_copied_extra_file"
sb_extra_files_mode: "0750"

# dir/files permissions for "{{ sb_keystores_dest }}/" && "{{ sb_keystores_dest }}/each_copied_keystore_file"
sb_keystores_mode: "0750"

sb_undeploy: false
sb_remove_extra_files: false
sb_remove_keystores: false

```

Optional variables
------------------

- <a name="optional">Other variables that may optionally be added/changed:</a>
  - `sb_extra_files_src`: The source location of some extra binary/library files that may be needed by the application. Recommended value: "`playbook_dir`/files/extra".
  - `sb_keystores_src`: The source location of the keystore & cacerts files (should be relative to local playbook project path) . If you need an environment based separation, you'll need to customize this location to include you environment name. Recommended value: "`playbook_dir`/files/keys".
  - `sb_keystores_dest`: The destination location (directory) of the keystore & cacerts files (on the remote target servers). Recommended value: "`sb_appbase`/keys"

- <a name="keystore">**Using keystores (after setting `sb_keystores_src` & `sb_keystores_dest`):**</a><br/>
  - Since the variables `sb_keystores_src` & **`sb_keystores_dest`** are referring to folders which may contain any number of keystores-files, but the springboot application will require only one keystore by setting the property `server.ssl.key-store`, one (the user, role consumer) should specify that keystore (otherwise no keystores are used) like this:<br/>
  - `sb_java_extra_args: "-Dserver.ssl.key-store=file:{{ sb_keystores_dest }}/{{ sb_appname }}_keystore1.jks -Dserver.ssl.key-store-password={{ my_app_keystore_vault_password_var }}"`<br/>
  - Don't forget to set other keystore properties like `server.ssl.key-store-password` or `server.ssl.key-password`.<br/>
  - See also: [Configure SSL in Springboot Application](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#howto-configure-ssl)

- Variables that are used by the template env file `service-env.j2` which should be defined:
  - `sb_log_dir`
  - `sb_appbase_conf_location`
  - `sb_jar_location`
  - `sb_appjarname`

- <a name="example">Example of variable definition at playbook level:</a>

```yaml
---
sb_appname: MyApp
sb_app_version: 1.0.0
sb_appjarname: "MyApp-{{ sb_app_version }}.jar"
sb_keystores_src: "{{ playbook_dir }}/files/keys"
sb_keystores_dest: "{{ sb_appbase }}/keys"
sb_java_extra_args: "-Dserver.ssl.key-store=file:{{ sb_keystores_dest }}/{{ sb_appname }}_keystore1.jks -Dserver.ssl.key-store-password={{ my_app_keystore_vault_password_var }}"
sb_cleanup_entire_workdir: yes
```

Role execution flow
--------------------

At runtime, the role will perform the following actions:

- Attempt to stop the server via an existing systemd service instance. If the service exists, it is stopped and the application files are removed (see: `sb_user`, `sb_log_dir`, `sb_become`, `sb_service_name`)
- If the service does not exist, it is registered as an instance of a systemd service template (for RHEL7 or RHEL8) (see: `sb_start_at_boot`, `sb_become` & `sb_service_name`)
- The application is copied to the target location (see: `sb_appname`, `sb_app_version`, `sb_appjarname`, `sb_appbase`, `sb_jar_location`, `sb_workdir`)
- Copy all the configuration files via template (see: `sb_conf_src` & `sb_appbase_conf_location`)
- Copy extra binary files if they exist (and if `sb_extra_files_src` & `sb_extra_files_dest` are defined and non-empty)
- Copy keystores & certificates if they exist. (*User/Consumer have to add them to the springboot application config !*, see: [Using Keystores](#keystore))
- Start the service via handler (if `sb_run_handlers` is `yes`/`true`)
- Cleanup the working directory via handler (if `sb_run_handlers` is `yes`/`true`)

Dependencies
-------------

- RHEL style distribution, version 7.x to 9.x
- Ansible role ansible-role-springboot_foundation
- `ansible` version >= 2.5.0
- ability to become root
