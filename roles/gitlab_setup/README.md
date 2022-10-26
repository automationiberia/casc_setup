Role Name
=========

Create the repository, Upload the content crated, for example, by filtree_create role of this collection, Create and protect branches and configure the webooks.

Requirements
------------

Collection community.general to configure webhooks

Role Variables
--------------

|Variable Name|Default Value|Requiered for tasks|Description|
|:---:|:---:|:---:|:---:|
|gitlab_api_url|N/A|create_branch.yml<br>gitlab_init.yml<br>gitlab_webhook.yml<br>| Api URL of gitlab|
|gitlab_api_token|N/A|create_branch.yml<br>gitlab_init.yml<br>gitlab_webhook.yml|Api Token of gitlab|
|gitlab_group|N/A|create_branch.yml<br>gitlab_init.yml<br>gitlab_webhook.yml<br>|Group of gitlab|
|gitlab_subgroup|N/A|create_branch.yml<br>gitlab_init.yml<br>gitlab_webhook.yml|Subgroup of gitlab|
|gitlab_project|N/A|create_branch.yml<br>gitlab_init.yml<br>gitlab_push.yml<br>gitlab_webhook.yml|Project of gitlab|
|gitlab_validate_certs|N/A|create_branch.yml<br>gitlab_init.yml<br>gitlab_webhook.yml|Validate Certs for gitlab|
|gitlab_api_user|N/A|gitlab_init.yml|Api user of gitlab|
|gitlab_branches|N/A|create_branch.yml|List of branches|
|gitlab_default_branch|N/A|create_branch.yml<br>gitlab_init.yml|Default branch|
|gitlab_http_protocol|N/A|gitlab_init.yml|Protocol of gitlab (http or https)|
|controller_hostname|N/A|gitlab_webhook.yml|Hostname of controller/tower|
|controller_username|N/A|gitlab_webhook.yml|Username of controller/tower|
|controller_password|N/A|gitlab_webhook.yml|Password of username of controller/tower|
|controller_validate_certs|N/A|gitlab_webhook.yml|Validate certs for controller/tower|
|workflow_job_template_name|N/A|gitlab_webhook.yml|Name of Workflow to launch with the webhook configured|
|gitlab_project_temp_dir|/tmp/gitlab-clone|gitlab_init.yml<br>gitlab_push.yml|Temporary work directory to create filetree configuration for CasC|
|hook_validate_certs|false|gitlab_webhook.yml|Validate certs in the webhooks configured|
|gitlab_action_push|true|gitlab_webhook.yml|Action push for the webhook (true or false). True if you want trigger webhook in a push action|
|gitlab_action_tag|false|gitlab_webhook.yml|Action tag for the webhook (true or false). True if you want trigger webhook in a tag action|
|gitlab_visibility|private|gitlab_init.yml|Visibility of gitlab repository|


Dependencies
------------

N/A

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```
$ cat gitlab_setup.yml

---
- hosts: localhost
  connection: local
  gather_facts: no
  vars:
    gitlab_api_url: https://gitlab.testing.com
    gitlab_validate_certs: false
    gitlab_api_user: aap-demo
    gitlab_api_token: "TOKEN_GITLAB"
    gitlab_group: gitlab_group
    gitlab_subgroup: gitlab_subgroup
    gitlab_project: gitlab_project
    gitlab_visibility: internal
    gitlab_default_branch: prod_casc
    gitlab_http_protocol: http
    gitlab_branches:
      - prod_casc
      - dev_casc

  tasks:
    - name: GitLab init
      ansible.builtin.include_role:
        name: gitlab_setup
        tasks_from: gitlab_init.yml
        apply:
          tags: git-init
      tags: git-init

    - name: Create dir structure
      ansible.builtin.include_role:
        name: filetree_create
        apply:
          tags: filetree_create
      tags: filetree_create

    - name: GitLab push
      ansible.builtin.include_role:
        name: gitlab_setup
        tasks_from: gitlab_push.yml
        apply:
          tags: git-push
      tags: git-push

    - name: GitLab create branches
      ansible.builtin.include_role:
        name: gitlab_setup
        tasks_from:  create_branch.yml
        apply:
          tags: git-branches
      loop: "{{ gitlab_branches }}"
      loop_control:
        loop_var: __gitlab_branches_item
      tags: git-branches


$ cat gitlab_webhook.yml 
---
- hosts: all
  connection: local
  gather_facts: no
  vars:
    controller_username: "{{ vault_controller_username | default(lookup('env', 'CONTROLLER_USERNAME')) }}"
    controller_password: "{{ vault_controller_password | default(lookup('env', 'CONTROLLER_PASSWORD')) }}"
    controller_hostname: "{{ vault_controller_hostname | default(lookup('env', 'CONTROLLER_HOST')) }}"
    controller_validate_certs: "{{ vault_controller_validate_certs | default(lookup('env', 'CONTROLLER_VERIFY_SSL')) }}"
    gitlab_api_url: https://gitlab.testing.com
    gitlab_validate_certs: false
    gitlab_api_user: aap-demo
    gitlab_api_token: "TOKEN_GITLAB"
    gitlab_group: gitlab_group
    gitlab_subgroup: gitlab_subgroup
    gitlab_project: gitlab_project
    gitlab_action_push: true
    gitlab_action_tag: false
    gitlab_branch_filter: dev-casc
    workflow_job_template_name: "{{ gitlab_project }} CasC_AAP_Workflow"

  tasks:
    - name: GitLab Webhook
      ansible.builtin.include_role:
        name: gitlab_setup
        tasks_from: gitlab_webhook.yml
...
```

License
-------

BSD

Author Information
------------------

- silvinux <silvinux7@gmail.com>
- adonis <adonis@redhat.com>

