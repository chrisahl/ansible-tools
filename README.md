# ansible-tools

A set of tools for use with Ansible Automation Platform (AAP).

## Ensure python version is a minimum of 3.9
```bash
 python3 --version
```
If you have an older python version, install a newer version.

## Ensure Ansible tooling is installed
```bash
python3 -m pip install ansible
```

## Clone this repo
```bash
git clone https://github.com/chrisahl/ansible-tools.git
```

## Change to the cloned repo directory
```bash
cd ansible-tools
```

## Setup a Python virtual environment
```bash
python3 -m venv venv
source venv/bin/activate
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
By manually editing `export_controller.yml` and the ` awx.awx.export` task, it is possible to control the data exported. For example, to skip the exporting of Users, change `awx.awx.export` by commenting out and uncommenting as follows:
```
    - name: Export assets
      awx.awx.export:
        # all: true
        applications: ['all']
        credential_types: ['all']
        credentials: ['all']
        execution_environments: ['all']
        inventory: ['all']
        inventory_sources: ['all']
        job_templates: ['all']
        notification_templates: ['all']
        organizations: ['all']
        projects: ['all']
        schedules: ['all']
        teams: ['all']
        # users: ['all']
        workflow_job_templates: ['all']
```

For more information see [awx.awx.export documentation](https://docs.ansible.com/ansible/latest/collections/awx/awx/export_module.html)  By default, all data is exported.

#### Variables
##### Required
The following variables need to be defined when running the playbook:
- controller_host_src - URL of controller to migrate data from
- controller_user_src - Ansible user of controller to migrate data from
- controller_pwd_src - Ansible password of user of controller to migrate data from

##### Optional
The following variables are optional:
- output_filename: The filename to write the exported data to. Defaults to `output_controller_assets.yml`

#### Example Usage
```bash
ansible-playbook export_controller.yml -e controller_host_src=<URL of AAP controller>  -e controller_user_src="<AAP user>" -e controller_pwd_src="<AAP password>"

```


#### Some exported data might need manual patches
Some of the data for notification_templates in the `output_filename` (Default value of `output_controller_assets.yml`) might contain variables enclosed in double curly braces (**{{  }}**)in the sections **assets.notification_templates[\*]messages.*.message**. Each **message** value must be wrapped using the **{% raw %}** and **{% endraw %}** tags  to prevent the variable from being substituted while being read by the importer.

For example:
```bash
message: `{% raw %} {{ job_friendly_name }} #{{
                    job.id }} ''{{ job.name }}'' {{
                    job.status }}: {{ url }}  {% endraw %}`
```

If you want to try an automated way to do this, use
[yq](https://github.com/mikefarah/yq) and save the results to a new file:**
```bash
yq '.assets.notification_templates[].messages.*.message |= "{% raw %}" + . + "{% endraw %}"' output_controller_assets.yml > output_controller_assets-rawdata.yml
```
However, this may also introduce entries that you will need to manually edit.  Search for any lines that contain:
```bash
message: '{% raw %}{% endraw %}'
```
and delete them.  The notification_templates data should now be valid for import.

**Then use the additional playbook parameter to use this new file for import:**
```bash
-e output_filename=output_controller_assets-rawdata.yml
```


### Import data - import_controller.yml
Playbook `import_controller.yml` can be used to migrate Ansible controller data from one deployment to another deployment by importing data from a file previously generated by `export_controller.yml`.

**NOTE: Be cautious when trying to migrate Users, especially admin since it might change your current admin password on the deployment you are importing the data to.  If you exported the Users, it is advisable to edit the `output_controller_assets.yml` data file and delete the Users `admin` entry prior to running the import.**

**For example, using [yq](https://github.com/mikefarah/yq) you can remove the admin entry and save it to a new file:**
```bash
yq 'del(.assets.users[] | select(.username == "admin"))' output_controller_assets.yml > output_controller_assets-noadmin.yml
```

**Then use the additional playbook parameter to use this new file for import:**
```bash
-e output_filename=output_controller_assets-noadmin.yml
```

#### Variables
##### Required
The following variables need to be defined when running the playbook:
- controller_host_dest - URL of controller to migrate data to
- controller_user_dest - Ansible user oauth token of controller to migrate data to
- controller_pwd_dest - Ansible password of user of controller to migrate data to

##### Optional
The following variables are optional:
- output_filename: The filename to read the imported data from. Defaults to output_controller_assets.yml

#### Example Usage
```bash
ansible-playbook import_controller.yml -e controller_host_dest=<URL of AAP controller>  -e controller_user_dest="<AAP user>" -e controller_pwd_dest="<AAP password>"

```

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
