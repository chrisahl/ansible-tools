# ansible-tools

A set of tools for use with Ansible.

## Install required Python - awxkit
```bash
pip3 install -r requirements.txt
```

## Install required Ansible collection - awx.awx
Based on export and import tools in https://docs.ansible.com/ansible/latest/collections/awx/awx/index.html
```bash
ansible-galaxy collection install -r requirements.yml
```

## Playbooks

### Export data - export_controller.yml
Playbook `export_controller.yml` can be used to migrate Ansible controller data from one deployment to another deployment by saving the exported data to a file.  The file can later be imported using the `import_controller.yml` playbook.

#### Customize what gets exported
By manually editing `export_controller.yml` and the ` awx.awx.export` task, it is possible to control the data exported. For more information see [awx.awx.export documentation](https://docs.ansible.com/ansible/latest/collections/awx/awx/export_module.html)  By default, all data is exported.

#### Variables
##### Required
The following variables need to be defined when running the playbook:
- controller_host_src - URL of controller to migrate data from
- controller_user_src - Ansible user of controller to migrate data from
- controller_pwd_src - Ansible password of user of controller to migrate data from

##### Optional
The following variables are optional:
- output_filename: The filename to write the exported data to. Defaults to `output_controller_assets.yml`

### Import data - import_controller.yml
Playbook `import_controller.yml` can be used to migrate Ansible controller data from one deployment to another deployment by importing data from a file previously generated by `export_controller.yml`.

**NOTE: Be cautious when trying to migrate Users admin since it might change your current admin password on the deployment you are importing the data to.  If you exported the Users, it is advisable to edit the `output_controller_assets.yml` data file and delete the Users `admin` entry prior to running the import.**

#### Variables
##### Required
The following variables need to be defined when running the playbook:
- controller_host_dest - URL of controller to migrate data to
- controller_user_dest - Ansible user oauth token of controller to migrate data to
- controller_pwd_dest - Ansible password of user of controller to migrate data to

##### Optional
The following variables are optional:
- output_filename: The filename to read the imported data from. Defaults to output_controller_assets.yml

### Migration tool - migrate_controller.yml
Playbook `migrate_controller.yml` can be used to selectively migrate Ansible controller data from one deployment to another deployment without saving the data to a file. This requires both the source and target deployments to be available simultaneously. Ansible tags can be used to select which groups of data to migrate.

**NOTE: Be cautious when trying to migrate Users admin since it might change your current admin password on the deployment you are migrating the data to.**

#### Variables
##### Required
The following variables need to be defined when running the playbook:
- controller_host_src - URL of controller to migrate data from
- controller_user_src - Ansible user of controller to migrate data from
- controller_pwd_src - Ansible password of user of controller to migrate data from

- controller_host_dest - URL of controller to migrate data to
- controller_user_dest - Ansible user oauth token of controller to migrate data to
- controller_pwd_dest - Ansible password of user of controller to migrate data to

#### Using tags to limit playbook tasks
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
