ansible-role-springboot_instance
================================

An Ansible role that deploys Springboot an application instance.

Requirements
-------------

- The `ansible-role-springboot_foundation` ran successfully,
- The springboot application is being installed as a service,
- The application JAR file must be downloaded previously to `springboot_instance__workdir` by the playbook,
- The configuration files should be stored/downloaded to {{ playbook_dir }}/files/conf. Also, they should be templated.
- This Role requires "root" access to manage services and to perform SELinux labelling of resources.

Role Variables
---------------

- Variables which **MUST** be defined at the playbook level:
  - `springboot_instance__appname`: The name of the application (Only A-Z/a-z letters, 0-9 digits and _, **NO hyphen, slash, plus, at [- / + @]**)
  - `springboot_instance__app_version`: Version of the application
  - `springboot_instance__appjarname`: The JAR application name

- Variables which **SHOULD NOT** be altered (unless you both have a fairly good reason and know exactly how to deal with the SELinux customisations that shall be needed):
  - `springboot_instance__service_name`: The name of the systemd service unit for the Springboot application (Default: "springboot@`springboot_instance__appname`")

IMPORTANT: for all `*_mode` variables, the value MUST be surrounded with quotes (") and MUST be specified as 4 digit octal value (eg: "0700").

- Default variables (from defaults/main.yml):
  - `springboot_instance__workdir`: The path where the artifacts are being downloaded on the target machines. (Default: "/var/tmp/springboot")
  - `springboot_instance__log_dir`: The path where the Springboot instance will generates its log files. (Default: "/var/log/springboot/`springboot_instance__appname`")
  - `springboot_instance__appbase`: The path where the app will reside. (Default: "/opt/springboot/`springboot_instance__appname`")
  - `springboot_instance__appbase_mode`: The dir-mode permission for `springboot_instance__appbase`. (Default: "0750")
  - `springboot_instance__service_env_filename`: The name of the environment file for the systemd service unit. (Default: "/opt/springboot/`springboot_instance__appname`/env")
  - `springboot_instance__jar_location`: The directory where the *JAR application* will be deployed. (Default: "`springboot_instance__appbase`/lib")
  - `springboot_instance__conf_src`: Application configuration files are expected to be Jinja2 templates stored in the playbook in "{{ playbook_dir }}/files/conf", they should set all needed application properties. (Default: "`playbook_dir`/files/conf" ie "{{ playbook_dir }}/files/conf")
  - `springboot_instance__appbase_conf_location`: The destination directory for the configuration files. (Default: "`springboot_instance__appbase`/conf")
  - `springboot_instance__java_security_file`: Location of the Java VM security properties files specific to the Springboot application. (Default: "`springboot_instance__appbase_conf_location`/java.security")

  - `springboot_instance__run_handlers`: Should the `Start Springboot instance` and `Cleanup workdir` handlers run automatically, or not. (Default: `true`)
  - `springboot_instance__cleanup_entire_workdir`: Should remove the entire `springboot_instance__workdir` (not only the file `springboot_instance__workdir`/`springboot_instance__appjarname`), or not. Default value: "false". (**This works only if `springboot_instance__run_handlers` is set on "true" ("true") !**)

  - `springboot_instance__listen_port`: TCP port the application binds to and listens on, for SELinux port to be labelled. (Default: 8443)
  - `springboot_instance__memory_max_heapsize_mb`: Maximum heap size (in MiB) for the Java VM.((Default: 768)
  - `springboot_instance__memory_min_heapsize_mb`: Minimum heap size (in MiB) for the Java VM.((Default: 384)
  - `springboot_instance__memory_stacksize_kb`: Stack size (in KiB) for the Java VM. (Default: 220)
  - `springboot_instance__memory_hugepages`: Whether the Java VM will use huge memory pages (2MiB pages instead of standard 4KiB). (Default: false)

  - `springboot_instance__i18n_lang`: Value for the LANG environment variable for the Java VM. (Default: "en_US.UTF-8")

  - `springboot_instance__enable_service`: Whether the systemd service unit for the Springboot application should be enabled (started at boot time). (Default: true)
  - `springboot_instance__startup_timeout_sec`: Delay (in seconds) for the Springboot application to startup correctly. (Default: 10)

  - `springboot_instance__java_version`: Version of the Java runtime. (Default: 11)
  - `springboot_instance__java_flavour`: Flavour of the Java runtime. (Default: "openjdk")
  - `springboot_instance__java_extra_args`: Extra arguments for the Java VM, value is passed to the Java process by the systemd service unit environment file (`service-env.j2`jinja2 template). (Default: empty.)
  - `springboot_instance__java_tmpdir`:  Location for Java temporary files. (Default: "`/srv/springboot`/`springboot_instance__appname`")
  - `springboot_instance__log_symlink`: Whether a `logs`symbolic link should be created in `springboot_instance__appbase` to point to `springboot_instance__log_dir`. (Default: `false`)

  - `springboot_instance__enable_dynlibs`: Whether the Springboot appliction is allowed to create dynamic libraries or equivalent. (Default: false)

  - `springboot_instance__app_mode`: The dir & files mode permission for the JAR application destination location ("`springboot_instance__jar_location`/" & "`springboot_instance__jar_location`/`springboot_instance__appjarname`"). (Default: "0750")
  - `springboot_instance__conf_mode`: The dir & files mode permission for the `springboot_instance__appbase_conf_location` and each copied conf dir & file from `springboot_instance__conf_src` to `springboot_instance__appbase_conf_location`/. (Default: "0750")

  - `springboot_instance__log_dir_mode`: The dir mode permission for the log folder. (Default: "0750")
  - `springboot_instance__log_dir_retention_days`: Number of days to keep log files in `springboot_instance__log_dir`.

  - `springboot_instance__audit_keyword_read`: The keyword that will be tagged on audit read events on Springboot application sensitive files. (Default: empty, no event audited)
  - `springboot_instance__audit_keyword_write`: The keyword that will be tagged on audit write events on Springboot application sensitive files. (Default: empty, no event audited)

  - `springboot_instance__become`: Whether the role becomes `root` to create and manage Springboot application systemd units and to perform SELinux labelling of resources.
  - `springboot_instance__run_dir`: The destination directory where the service of the application may store transient files while the application runs. (Default: "`springboot_instance__appbase`/run")
  - `springboot_instance__run_dir_mode`: The dir mode permission for `springboot_instance__run_dir` where transient files will reside. (Default: "0750")

  - `springboot_instance__extra_files_dest`: The destination directory for additional optional files,such as libraries or binaries, that may be needed by the JAR application. (Default: `springboot_instance__jar_location`). (See the optional `springboot_instance__extra_files_src` variable described below in the section ["Other variables that may optionally be added/changed:"](#optional)))
  - `springboot_instance__extra_files_mode`: The dir & files modes of the destination folder for extra binary files that may be needed by the aplpication. (Default: "0750")
  - `springboot_instance__keystores_mode`: The mode of the directory & files to be populated from `springboot_instance__keystores_src` (local playbook path) to `springboot_instance__keystores_dest` (on the target host). (Default: "0750")

  - `springboot_instance__undeploy`: Triggers the un-deployment of the Springboot application. (Default value: `false`)
  - `springboot_instance__remove_extra_files`: Triggers the removal of files/directories under `springboot_instance__extra_files_dest` when `springboot_instance__undeploy` is set to `true`. (Default value: `false`)
  - `springboot_instance__remove_keystores`: Triggers the removal of files/directories under `springboot_instance__keystores_dest` when `springboot_instance__undeploy` is set to `true`. (Default value: `false`)

If you don't want the handlers to be run at the end of the role execution, you should set the variable for the `springboot_instance__run_handlers` to `false`. However, if you want to run the handlers afterwards, the first task that is executed after the role should set the value for this variable to true.

```yaml
  post_tasks:
    - set_fact:
        springboot_instance__run_handlers: true
```

```yaml
---
# Definition of defaults file for ansible-role-springboot_instance

springboot_instance__become: true

springboot_instance__workdir: /var/tmp/springboot
springboot_instance__appbase: "/opt/springboot/{{ springboot_instance__appname }}"

springboot_instance__service_env_filename: "/opt/springboot/{{ springboot_instance__appname }}/env"

springboot_instance__jar_location: "{{ springboot_instance__appbase }}/lib"
springboot_instance__conf_src: "{{ playbook_dir }}/files/conf"
springboot_instance__appbase_conf_location: "{{ springboot_instance__appbase }}/conf"
springboot_instance__java_security_file: "{{ springboot_instance__appbase_conf_location }}/java.security"
springboot_instance__keystores_dest: "{{ springboot_instance__appbase }}/keys"
springboot_instance__log_dir: "/var/log/springboot/{{ springboot_instance__appname }}"
springboot_instance__srv_dir: "/srv/springboot/{{ springboot_instance__appname }}"
springboot_instance__run_handlers: true
springboot_instance__cleanup_entire_workdir: false
springboot_instance__extra_files_dest: "{{ springboot_instance__jar_location }}"
springboot_instance__appbase_mode: "0750"

springboot_instance__listen_port: 8443
#springboot_instance__monitoring_port: 

springboot_instance__memory_max_heapsize_mb: 768
springboot_instance__memory_min_heapsize_mb: 384
springboot_instance__memory_stacksize_kb: 220
springboot_instance__memory_hugepages: false

springboot_instance__i18n_lang: "en_US.UTF-8"

springboot_instance__enable_service: true
springboot_instance__startup_timeout_sec: 10
springboot_instance__app_log_file: "{{ springboot_instance__log_dir }}/{{ springboot_instance__appname }}-{{ springboot_instance__app_version }}.log"

springboot_instance__java_version: 11
springboot_instance__java_flavour: "openjdk"
springboot_instance__java_extra_args: ""
springboot_instance__java_tmpdir: "{{ springboot_instance__srv_dir }}"
springboot_instance__log_symlink: false

springboot_instance__enable_dynlibs: false

# dir permissions for "{{ springboot_instance__appbase }}/run/" used to store the running transient files
springboot_instance__run_dir: "{{ springboot_instance__appbase }}/run"
springboot_instance__run_dir_mode: "0750"

# dir/files permissions for "{{ springboot_instance__jar_location }}/" & "{{ springboot_instance__jar_location }}/{{ springboot_instance__appjarname }}"
# ( of course, the user running the application must be able to read)
springboot_instance__app_mode: "0750"

# dir/files permissions for "{{ springboot_instance__appbase_conf_location }}/" , "{{ springboot_instance__appbase_conf_location }}//each_copied_conf-dir/" & "{{ springboot_instance__appbase_conf_location }}//each_copied_conf-dir/each_copied_conf-file":
springboot_instance__conf_mode: "0750"

# dir permissions and retention for "{{ springboot_instance__log_dir }}/" used to store the running jar logs
springboot_instance__log_dir_mode: "0750"
springboot_instance__log_dir_retention_days: 30

# dir/files permissions for "{{ springboot_instance__extra_files_dest }}/" && "{{ springboot_instance__extra_files_dest }}/each_copied_extra_file"
springboot_instance__extra_files_mode: "0750"

# dir/files permissions for "{{ springboot_instance__keystores_dest }}/" && "{{ springboot_instance__keystores_dest }}/each_copied_keystore_file"
springboot_instance__keystores_mode: "0750"

springboot_instance__undeploy: false
springboot_instance__remove_extra_files: false
springboot_instance__remove_keystores: false

```

Optional variables
------------------

- <a name="optional">Other variables that may optionally be added/changed:</a>
  - `springboot_instance__extra_files_src`: The source location of some extra binary/library files that may be needed by the application. Recommended value: "`playbook_dir`/files/extra".
  - `springboot_instance__keystores_src`: The source location of the keystore & cacerts files (should be relative to local playbook project path) . If you need an environment based separation, you'll need to customize this location to include you environment name. Recommended value: "`playbook_dir`/files/keys".
  - `springboot_instance__keystores_dest`: The destination location (directory) of the keystore & cacerts files (on the remote target servers). Recommended value: "`springboot_instance__appbase`/keys"

- <a name="keystore">**Using keystores (after setting `springboot_instance__keystores_src` & `springboot_instance__keystores_dest`):**</a><br/>
  - Since the variables `springboot_instance__keystores_src` & **`springboot_instance__keystores_dest`** are referring to folders which may contain any number of keystores-files, but the springboot application will require only one keystore by setting the property `server.ssl.key-store`, one (the user, role consumer) should specify that keystore (otherwise no keystores are used) like this:<br/>
  - `springboot_instance__java_extra_args: "-Dserver.ssl.key-store=file:{{ springboot_instance__keystores_dest }}/{{ springboot_instance__appname }}_keystore1.jks -Dserver.ssl.key-store-password={{ my_app_keystore_vault_password_var }}"`<br/>
  - Don't forget to set other keystore properties like `server.ssl.key-store-password` or `server.ssl.key-password`.<br/>
  - See also: [Configure SSL in Springboot Application](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#howto-configure-ssl)

- Variables that are used by the template env file `service-env.j2` which should be defined:
  - `springboot_instance__log_dir`
  - `springboot_instance__appbase_conf_location`
  - `springboot_instance__jar_location`
  - `springboot_instance__appjarname`

- <a name="example">Example of variable definition at playbook level:</a>

```yaml
---
springboot_instance__appname: MyApp
springboot_instance__app_version: 1.0.0
springboot_instance__appjarname: "MyApp-{{ springboot_instance__app_version }}.jar"
springboot_instance__keystores_src: "{{ playbook_dir }}/files/keys"
springboot_instance__keystores_dest: "{{ springboot_instance__appbase }}/keys"
springboot_instance__java_extra_args: "-Dserver.ssl.key-store=file:{{ springboot_instance__keystores_dest }}/{{ springboot_instance__appname }}_keystore1.jks -Dserver.ssl.key-store-password={{ my_app_keystore_vault_password_var }}"
springboot_instance__cleanup_entire_workdir: true
```

Role execution flow
--------------------

At runtime, the role will perform the following actions:

- Attempt to stop the server via an existing systemd service instance. If the service exists, it is stopped and the application files are removed (see: `springboot_instance__user`, `springboot_instance__log_dir`, `springboot_instance__become`, `springboot_instance__service_name`)
- If the service does not exist, it is registered as an instance of a systemd service template (for RHEL7 or RHEL8) (see: `springboot_instance__start_at_boot`, `springboot_instance__become` & `springboot_instance__service_name`)
- The application is copied to the target location (see: `springboot_instance__appname`, `springboot_instance__app_version`, `springboot_instance__appjarname`, `springboot_instance__appbase`, `springboot_instance__jar_location`, `springboot_instance__workdir`)
- Copy all the configuration files via template (see: `springboot_instance__conf_src` & `springboot_instance__appbase_conf_location`)
- Copy extra binary files if they exist (and if `springboot_instance__extra_files_src` & `springboot_instance__extra_files_dest` are defined and non-empty)
- Copy keystores & certificates if they exist. (*User/Consumer have to add them to the springboot application config !*, see: [Using Keystores](#keystore))
- Start the service via handler (if `springboot_instance__run_handlers` is `true`)
- Cleanup the working directory via handler (if `springboot_instance__run_handlers` is `true`)

Dependencies
-------------

- RHEL style distribution, version 7.x to 9.x
- Ansible role ansible-role-springboot_foundation
- `ansible` version >= 2.5.0
- ability to become root
