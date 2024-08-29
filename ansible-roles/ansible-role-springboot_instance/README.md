ansible-role-springboot_instance
================================

An Ansible role that deploys Springboot an application instance.

Requirements
-------------

- The `ansible-role-springboot_foundation` ran successfully,
- The springboot application is being installed as a service,
- The application JAR file must be downloaded previously to `sbi_workdir` by the playbook,
- The configuration files should be stored/downloaded to {{ playbook_dir }}/files/conf. Also, they should be templated.
- This Role requires "root" access to manage services and to perform SELinux labelling of resources.

Role Variables
---------------

- Variables which **MUST** be defined at the playbook level:
  - `sbi_appname`: The name of the application (Only A-Z/a-z letters, 0-9 digits and _, **NO hyphen, slash, plus, at [- / + @]**)
  - `sbi_app_version`: Version of the application
  - `sbi_appjarname`: The JAR application name

- Variables which **SHOULD NOT** be altered (unless you both have a fairly good reason and know exactly how to deal with the SELinux customisations that shall be needed):
  - `sbi_service_name`: The name of the systemd service unit for the Springboot application (Default: "springboot@`sbi_appname`")

IMPORTANT: for all `*_mode` variables, the value MUST be surrounded with quotes (") and MUST be specified as 4 digit octal value (eg: "0700").

- Default variables (from defaults/main.yml):
  - `sbi_workdir`: The path where the artifacts are being downloaded on the target machines. (Default: "/var/tmp/springboot")
  - `sbi_log_dir`: The path where the Springboot instance will generates its log files. (Default: "/var/log/springboot/`sbi_appname`")
  - `sbi_appbase`: The path where the app will reside. (Default: "/opt/springboot/`sbi_appname`")
  - `sbi_appbase_mode`: The dir-mode permission for `sbi_appbase`. (Default: "0750")
  - `sbi_service_env_filename`: The name of the environment file for the systemd service unit. (Default: "/opt/springboot/`sbi_appname`/env")
  - `sbi_jar_location`: The directory where the *JAR application* will be deployed. (Default: "`sbi_appbase`/lib")
  - `sbi_conf_src`: Application configuration files are expected to be Jinja2 templates stored in the playbook in "{{ playbook_dir }}/files/conf", they should set all needed application properties. (Default: "`playbook_dir`/files/conf" ie "{{ playbook_dir }}/files/conf")
  - `sbi_appbase_conf_location`: The destination directory for the configuration files. (Default: "`sbi_appbase`/conf")
  - `sbi_java_security_file`: Location of the Java VM security properties files specific to the Springboot application. (Default: "`sbi_appbase_conf_location`/java.security")

  - `sbi_run_handlers`: Should the `Start Springboot instance` and `Cleanup workdir` handlers run automatically, or not. (Default: `yes`)
  - `sbi_cleanup_entire_workdir`: Should remove the entire `sbi_workdir` (not only the file `sbi_workdir`/`sbi_appjarname`), or not. Default value: "no". (**This works only if `sbi_run_handlers` is set on "yes" ("true") !**)

  - `sbi_listen_port`: TCP port the application binds to and listens on, for SELinux port to be labelled. (Default: 8443)
  - `sbi_memory_max_heapsize_mb`: Maximum heap size (in MiB) for the Java VM.((Default: 768)
  - `sbi_memory_min_heapsize_mb`: Minimum heap size (in MiB) for the Java VM.((Default: 384)
  - `sbi_memory_stacksize_kb`: Stack size (in KiB) for the Java VM. (Default: 220)
  - `sbi_memory_hugepages`: Whether the Java VM will use huge memory pages (2MiB pages instead of standard 4KiB). (Default: no)

  - `sbi_i18n_lang`: Value for the LANG environment variable for the Java VM. (Default: "en_US.UTF-8")

  - `sbi_enable_service`: Whether the systemd service unit for the Springboot application should be enabled (started at boot time). (Default: yes)
  - `sbi_startup_timeout_sec`: Delay (in seconds) for the Springboot application to startup correctly. (Default: 10)

  - `sbi_java_version`: Version of the Java runtime. (Default: 11)
  - `sbi_java_flavour`: Flavour of the Java runtime. (Default: "openjdk")
  - `sbi_java_extra_args`: Extra arguments for the Java VM, value is passed to the Java process by the systemd service unit environment file (`service-env.j2`jinja2 template). (Default: empty.)
  - `sbi_java_tmpdir`:  Location for Java temporary files. (Default: "`/srv/springboot`/`sbi_appname`")
  - `sbi_log_symlink`: Whether a `logs`symbolic link should be created in `sbi_appbase` to point to `sbi_log_dir`. (Default: `no`)

  - `sbi_enable_dynlibs`: Whether the Springboot appliction is allowed to create dynamic libraries or equivalent. (Default: false)

  - `sbi_app_mode`: The dir & files mode permission for the JAR application destination location ("`sbi_jar_location`/" & "`sbi_jar_location`/`sbi_appjarname`"). (Default: "0750")
  - `sbi_conf_mode`: The dir & files mode permission for the `sbi_appbase_conf_location` and each copied conf dir & file from `sbi_conf_src` to `sbi_appbase_conf_location`/. (Default: "0750")

  - `sbi_log_dir_mode`: The dir mode permission for the log folder. (Default: "0750")
  - `sbi_log_dir_retention_days`: Number of days to keep log files in `sbi_log_dir`.

  - `sbi_audit_keyword_read`: The keyword that will be tagged on audit read events on Springboot application sensitive files. (Default: empty, no event audited)
  - `sbi_audit_keyword_write`: The keyword that will be tagged on audit write events on Springboot application sensitive files. (Default: empty, no event audited)

  - `sbi_become`: Whether the role becomes `root` to create and manage Springboot application systemd units and to perform SELinux labelling of resources.
  - `sbi_run_dir`: The destination directory where the service of the application may store transient files while the application runs. (Default: "`sbi_appbase`/run")
  - `sbi_run_dir_mode`: The dir mode permission for `sbi_run_dir` where transient files will reside. (Default: "0750")

  - `sbi_extra_files_dest`: The destination directory for additional optional files,such as libraries or binaries, that may be needed by the JAR application. (Default: `sbi_jar_location`). (See the optional `sbi_extra_files_src` variable described below in the section ["Other variables that may optionally be added/changed:"](#optional)))
  - `sbi_extra_files_mode`: The dir & files modes of the destination folder for extra binary files that may be needed by the aplpication. (Default: "0750")
  - `sbi_keystores_mode`: The mode of the directory & files to be populated from `sbi_keystores_src` (local playbook path) to `sbi_keystores_dest` (on the target host). (Default: "0750")

  - `sbi_undeploy`: Triggers the un-deployment of the Springboot application. (Default value: `false`)
  - `sbi_remove_extra_files`: Triggers the removal of files/directories under `sbi_extra_files_dest` when `sbi_undeploy` is set to `true`. (Default value: `false`)
  - `sbi_remove_keystores`: Triggers the removal of files/directories under `sbi_keystores_dest` when `sbi_undeploy` is set to `true`. (Default value: `false`)

If you don't want the handlers to be run at the end of the role execution, you should set the variable for the `sbi_run_handlers` to `false`. However, if you want to run the handlers afterwards, the first task that is executed after the role should set the value for this variable to true.

```yaml
  post_tasks:
    - set_fact:
        sbi_run_handlers: yes
```

```yaml
---
# Definition of defaults file for ansible-role-springboot_instance

sbi_become: yes

sbi_workdir: /var/tmp/springboot
sbi_appbase: "/opt/springboot/{{ sbi_appname }}"

sbi_service_env_filename: "/opt/springboot/{{ sbi_appname }}/env"

sbi_jar_location: "{{ sbi_appbase }}/lib"
sbi_conf_src: "{{ playbook_dir }}/files/conf"
sbi_appbase_conf_location: "{{ sbi_appbase }}/conf"
sbi_java_security_file: "{{ sbi_appbase_conf_location }}/java.security"
sbi_keystores_dest: "{{ sbi_appbase }}/keys"
sbi_log_dir: "/var/log/springboot/{{ sbi_appname }}"
sbi_srv_dir: "/srv/springboot/{{ sbi_appname }}"
sbi_run_handlers: yes
sbi_cleanup_entire_workdir: no
sbi_extra_files_dest: "{{ sbi_jar_location }}"
sbi_appbase_mode: "0750"

sbi_listen_port: 8443
#sbi_monitoring_port: 

sbi_memory_max_heapsize_mb: 768
sbi_memory_min_heapsize_mb: 384
sbi_memory_stacksize_kb: 220
sbi_memory_hugepages: no

sbi_i18n_lang: "en_US.UTF-8"

sbi_enable_service: yes
sbi_startup_timeout_sec: 10
sbi_app_log_file: "{{ sbi_log_dir }}/{{ sbi_appname }}-{{ sbi_app_version }}.log"

sbi_java_version: 11
sbi_java_flavour: "openjdk"
sbi_java_extra_args: ""
sbi_java_tmpdir: "{{ sbi_srv_dir }}"
sbi_log_symlink: no

sbi_enable_dynlibs: false

# dir permissions for "{{ sbi_appbase }}/run/" used to store the running transient files
sbi_run_dir: "{{ sbi_appbase }}/run"
sbi_run_dir_mode: "0750"

# dir/files permissions for "{{ sbi_jar_location }}/" & "{{ sbi_jar_location }}/{{ sbi_appjarname }}"
# ( of course, the user running the application must be able to read)
sbi_app_mode: "0750"

# dir/files permissions for "{{ sbi_appbase_conf_location }}/" , "{{ sbi_appbase_conf_location }}//each_copied_conf-dir/" & "{{ sbi_appbase_conf_location }}//each_copied_conf-dir/each_copied_conf-file":
sbi_conf_mode: "0750"

# dir permissions and retention for "{{ sbi_log_dir }}/" used to store the running jar logs
sbi_log_dir_mode: "0750"
sbi_log_dir_retention_days: 30

# dir/files permissions for "{{ sbi_extra_files_dest }}/" && "{{ sbi_extra_files_dest }}/each_copied_extra_file"
sbi_extra_files_mode: "0750"

# dir/files permissions for "{{ sbi_keystores_dest }}/" && "{{ sbi_keystores_dest }}/each_copied_keystore_file"
sbi_keystores_mode: "0750"

sbi_undeploy: false
sbi_remove_extra_files: false
sbi_remove_keystores: false

```

Optional variables
------------------

- <a name="optional">Other variables that may optionally be added/changed:</a>
  - `sbi_extra_files_src`: The source location of some extra binary/library files that may be needed by the application. Recommended value: "`playbook_dir`/files/extra".
  - `sbi_keystores_src`: The source location of the keystore & cacerts files (should be relative to local playbook project path) . If you need an environment based separation, you'll need to customize this location to include you environment name. Recommended value: "`playbook_dir`/files/keys".
  - `sbi_keystores_dest`: The destination location (directory) of the keystore & cacerts files (on the remote target servers). Recommended value: "`sbi_appbase`/keys"

- <a name="keystore">**Using keystores (after setting `sbi_keystores_src` & `sbi_keystores_dest`):**</a><br/>
  - Since the variables `sbi_keystores_src` & **`sbi_keystores_dest`** are referring to folders which may contain any number of keystores-files, but the springboot application will require only one keystore by setting the property `server.ssl.key-store`, one (the user, role consumer) should specify that keystore (otherwise no keystores are used) like this:<br/>
  - `sbi_java_extra_args: "-Dserver.ssl.key-store=file:{{ sbi_keystores_dest }}/{{ sbi_appname }}_keystore1.jks -Dserver.ssl.key-store-password={{ my_app_keystore_vault_password_var }}"`<br/>
  - Don't forget to set other keystore properties like `server.ssl.key-store-password` or `server.ssl.key-password`.<br/>
  - See also: [Configure SSL in Springboot Application](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#howto-configure-ssl)

- Variables that are used by the template env file `service-env.j2` which should be defined:
  - `sbi_log_dir`
  - `sbi_appbase_conf_location`
  - `sbi_jar_location`
  - `sbi_appjarname`

- <a name="example">Example of variable definition at playbook level:</a>

```yaml
---
sbi_appname: MyApp
sbi_app_version: 1.0.0
sbi_appjarname: "MyApp-{{ sbi_app_version }}.jar"
sbi_keystores_src: "{{ playbook_dir }}/files/keys"
sbi_keystores_dest: "{{ sbi_appbase }}/keys"
sbi_java_extra_args: "-Dserver.ssl.key-store=file:{{ sbi_keystores_dest }}/{{ sbi_appname }}_keystore1.jks -Dserver.ssl.key-store-password={{ my_app_keystore_vault_password_var }}"
sbi_cleanup_entire_workdir: yes
```

Role execution flow
--------------------

At runtime, the role will perform the following actions:

- Attempt to stop the server via an existing systemd service instance. If the service exists, it is stopped and the application files are removed (see: `sbi_user`, `sbi_log_dir`, `sbi_become`, `sbi_service_name`)
- If the service does not exist, it is registered as an instance of a systemd service template (for RHEL7 or RHEL8) (see: `sbi_start_at_boot`, `sbi_become` & `sbi_service_name`)
- The application is copied to the target location (see: `sbi_appname`, `sbi_app_version`, `sbi_appjarname`, `sbi_appbase`, `sbi_jar_location`, `sbi_workdir`)
- Copy all the configuration files via template (see: `sbi_conf_src` & `sbi_appbase_conf_location`)
- Copy extra binary files if they exist (and if `sbi_extra_files_src` & `sbi_extra_files_dest` are defined and non-empty)
- Copy keystores & certificates if they exist. (*User/Consumer have to add them to the springboot application config !*, see: [Using Keystores](#keystore))
- Start the service via handler (if `sbi_run_handlers` is `yes`/`true`)
- Cleanup the working directory via handler (if `sbi_run_handlers` is `yes`/`true`)

Dependencies
-------------

- RHEL style distribution, version 7.x to 9.x
- Ansible role ansible-role-springboot_foundation
- `ansible` version >= 2.5.0
- ability to become root
