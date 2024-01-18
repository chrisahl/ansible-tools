# ansible-tools

A set of tools for use with Ansible.

## Migration tool - migrate_controller.yml
Playbook `migrate_controller.yml` can be used to migrate Ansible controller data from one deployment to another deployment.

### Install required Python - awxkit
```bash
pip3 install -r requirements.txt
```

### Install required Ansible collection - awx.awx
Based on export and import tools in https://docs.ansible.com/ansible/latest/collections/awx/awx/index.html
```bash
ansible-galaxy collection install -r requirements.yml
```

### Variables
The following variables need to be defined when running the migration playbook:
- controller_host_src - URL of controller to migrate data from
- controller_user_src - Ansible user of controller to migrate data from
- controller_pwd_src - Ansible password of user of controller to migrate data from

- controller_host_dest - URL of controller to migrate data to
- controller_user_dest - Ansible user oauth token of controller to migrate data to
- controller_pwd_dest - Ansible password of user of controller to migrate data to

### Using tags to limit playbook tasks
The playbook has tags defined for the various tasks so you can use
`--tags` or `--skip-tags` and control which Ansible pieces get migrated.  
By default, all tasks get run.

To see the list of available tags:
```bash
ansible-playbook --list-tags migrate_controller.yml
```

For example, to skip the migration of Users, add this to the `ansible-playbook` command:
```bash
--skip-tags "Users"
```
