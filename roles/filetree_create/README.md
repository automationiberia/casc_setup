Role Name
=========

Create a basic Configuration as Code filetree to setup the day zero

Requirements
------------
N/A

Role Variables
--------------

|Variable Name|Default Value|Description|
|:---:|:---:|:---:|
|gitlab_project_temp_dir|/tmp/gitlab-clone|Temporary work directory to create filetree configuration for CasC |
|org_dir_structure_parent|- { path: "collections", mode: '0755' }<br>- { path: "group_vars", mode: '0755' }<br>- { path: "group_vars/all", mode: '0755' }<br>- { path: "{{ dir_orgs_vars }}/{{ gitlab_project }}", mode: '0755' }<br>- { path: "{{ dir_orgs_vars }}/{{ gitlab_project }}/env", mode: '0755' }<br>- { path: "{{ dir_orgs_vars }}/{{ gitlab_project }}/env/common", mode: '0755' }|Basic structure for a CasC repository|
|org_dir_structure_common|- { path: "controller_credential_types.d", mode: '0755', controller_list: controller_credential_types}<br>- { path: "controller_groups.d", mode: '0755', controller_list: controller_groups }<br>- { path: "controller_inventories.d", mode: '0755', controller_list: controller_inventories }<br>- { path: "controller_job_templates.d", mode: '0755', controller_list: controller_job_templates }<br>- { path: "controller_organizations.d", mode: '0755', controller_list: controller_organizations }<br>- { path: "controller_projects.d", mode: '0755', controller_list: controller_projects }<br>- { path: "controller_roles.d", mode: '0755', controller_list: controller_roles }<br>- { path: "controller_schedules.d", mode: '0755', controller_list: controller_schedules }<br>- { path: "controller_teams.d", mode: '0755', controller_list: controller_teams }<br>- { path: "controller_workflow_job_templates.d", mode: '0755', controller_list: controller_workflows }|Structure for common objects in controller/tower|
|org_dir_structure_env|- { path: "controller_credentials.d", mode: '0755' , controller_list: controller_credentials }<br>- { path: "controller_execution_environments.d", mode: '0755', controller_list: controller_execution_environments }<br>- { path: "controller_users.d", mode: '0755', controller_list: controller_users}<br>- { path: "controller_hosts.d", mode: '0755', controller_list: controller_hosts }<br>- { path: "controller_instance_groups.d", mode: '0755', controller_list: controller_instance_groups }<br>- { path: "controller_inventory_sources.d", mode: '0755', controller_list: controller_inventory_sources }<br>- { path: "controller_settings.d", mode: '0755', controller_list: controller_settings }|Structure for particular env objects in controller/tower|
|template_playbook_list|- { template: cd-gitlab-webhook-trigger.yml.j2, dest:  cd-gitlab-webhook-trigger.yml }<br>- { template: config-controller-filetree.yml.j2, dest: config-controller-filetree.yml }<br>- { template: drop_diff.yml.j2, dest: drop_diff.yml }<br>- { template: gitlab-ci.yml.j2, dest: .gitlab-ci.yml }<br>- { template: requirements.yml.j2, dest: collections/requirements.yml }<br>- { template: README.md.j2, dest: README.md }<br>- { template: gitignore.j2, dest: .gitignore }<br>- { template: gitlab_webhook.yml.j2, dest: gitlab_webhook.yml }|List of playbooks which will be generated|
|gitlab_project|N/A|Name of gitlab project|
|gitlab_branches|N/A|List with gitlab branches|
|automation_hub|ToDo|Hostname of Automation Hub|
|controller_dev|N/A|Hostname of Controller for Dev|
|controller_pro|N/A|Hostname of Controller for Pro|




Dependencies
------------

N/A

Example Playbook
----------------

```
---
- hosts: localhost
  connection: local
  gather_facts: no
  vars:
    gitlab_project: gitlab_project
    gitlab_branches:
      - prod_casc
      - dev_casc

  tasks:

    - name: Create dir structure
      ansible.builtin.include_role:
        name: filetree_create
        apply:
          tags: filetree_create
      tags: filetree_create
```

License
-------

BSD

Author Information
------------------

- silvinux <silvinux7@gmail.com>
- adonis <adonis@redhat.com>
