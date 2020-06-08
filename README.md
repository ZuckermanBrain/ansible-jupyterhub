app-jupyterhub
=========

This role sets up a JupyterHub instance.  Unlike other JupyterHub roles, it allows you to optionally define a service account that JupyterHub runs as.  This requires that you configure either the `sudospawner` or `batchspawner` if you use a non-root account (see [here](https://jupyterhub.readthedocs.io/en/stable/reference/config-sudo.html)).  This is desirable from a security perspective because it limits the amount of damage in the event that JupyterHub is compromised and also provides an audit trail.  If you configure the `sudospawner`, the `sudospawner` can run as anyone in the group designated by the `jupyterhub_group` variable.  If you configure the `batchspawner`, you must also define a set of commands that the `batchspawner` can run as any member of the JupyterHub group.

JupyterHub is installed into its own Python virtualenv.  A `systemd` unit file is also created for a `jupyterhub` service.

This role should ideally be combined with a reverse proxy configuration.

Requirements
------------

SSL certificates and a reverse proxy are not installed by this role and must be taken care of by separate playbooks/tasks/roles.  It is recommended that the private key at least be encapsulated as an encrypted varible (see [here](https://blog.confirm.ch/deploying-ssl-private-keys-with-ansible/) for a description of how to accomplish this.

Role Variables
--------------
`jupyterhub_venv_path` (string) : A path to where the Python virtualenv that JupyterHub is installed into should live.
`jupyterhub_config` (dict) : A dictionary that maps isomorphically to the configuration dictated in the JupyterHub `.py` configuration file.
`jupyterhub_config_yaml` (string) : A path where a YAML file generated from `jupyterhub_config_py` should live.
`jupyterhub_config_py` (string) : The path to where JupyterHub's `.py` config file should live.  Note that in this configuration, all that the `.py` does is load the YAML file and then populate configuration variables with entries in the dictionary defined in the file at `jupyterhub_config_yaml`.
`jupyterhub_config_path` (string) : The path to where JupyterHub's `.py` and `.yaml` config files should live.
`jupyterhub_service_user` (string) : A service user that should run JupyterHub.
`jupyterhub_group` (string) : The group that JupyterHub users should belong to.
`jupyterhub_spawners` (list) : The JupyterHub spawners to install.
`jupyterhub_batchspawner_commands` (string; not applicable if `batchspawner` not installed) : A comma-separated list with the full paths to batch scheduler commands that the SLURM `batchspawner` should be able to run.

Dependencies
------------

 * [ansible-slurm](https://github.com/galaxyproject/ansible-slurm) (if you are using `batchspawner`).  This is referred to as `app-slurm` using our group's naming conventions.
 * [ansible-role-apache](https://github.com/geerlingguy/ansible-role-apache) (if you are using a reverse proxy).  This is referred to as `app-apache` using our group's naming conventions.

Example Playbook
----------------

```
- name: Install Slurm
  hosts: jupyterhub
  become: true
  roles:
   - role: app-slurm

- name: Install certificates.
  hosts: jupyterhub
  become: true
  vars_files:
    - /etc/ansible/vars/jupyterhub_ssl_certificates.yml
  tasks:
    - name: make sure SSL certificate directory exists
      file:
        path: '{{ jupyterhub_ssl_cert_dir }}'
        state: directory
        owner: root
        group: root
        mode: 0755

    - name: make sure SSL certificate is installed
      copy:
        content: '{{ jupyterhub_ssl_certificate }}'
        dest: '{{ jupyterhub_ssl_certificate_path }}'
        owner: root
        group: root
        mode: 0644

    - name: make sure SSL private key is installed
      copy:
        content: '{{ jupyterhub_ssl_private_key }}'
        dest: '{{ jupyterhub_ssl_private_key_path }}'
        owner: root
        group: root
        mode: 0600
      no_log: true

- name: Install Jupyterhub
  hosts: jupyterhub
  become: true
  roles:
   - role: app-jupyterhub
   - role: app-apache
```

License
-------

Revised 3-Clause BSD License
