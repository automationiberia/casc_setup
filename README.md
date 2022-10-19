# Ansible Collection - automationiberia.casc_setup


## Roles

|Role Name|Description|
|:---:|:---:|
|||
|||

### Role Variables

|Roles|Variable Name|Default Value|Required|Description|
|:---:|:---:|:---:|:---:|:---:|
||||||
||||||
||||||


## Steps to setup CasC

Install this collection and configure a playbook to use it (playbook example in this repo inside example directory, you need to change variables inside or use extra-vars).

Launch playbook to configure git and create the initial filetree. To to that, use tags git-init and filetree_create:

```
ansible-playbook gitlab_setup.yml --tags git-init,filetree_create,git-push,git-branches
```

You already have your git repository ready to launch your first CD and get the controller configured from CasC approach. As day 0, you need to do it from CLI to, after that, configure the pipelines, webhooks and everthing needed to have a complete CasC for your controller.

Clone your repository and create a day 0 branch:

```
git clone git@gitlab.testing.com:acme/ansible-controller/org-example.git

git checkout -b day0
```

Configure controller access group_vars for day 0. If you want upload to git, you must encyrpt them with vault. Remember that they are only for day 0 from CLI so they are not actually needed in git for CasC flow:

```
vi group_vars/ado-dev/configure_connection_controller_credentials.yml
vi group_vars/ado-prd/configure_connection_controller_credentials.yml

ansible-vault encrypt group_vars/casc-dev/configure_connection_controller_credentials.yml
ansible-vault encrypt group_vars/casc-prd/configure_connection_controller_credentials.yml
```

Configure credentials needed for casc (There are variables to change in the following examples). Remember that you need to save your credentials propertly so you must encrypt credentials with vault before push to git: 

- controller_credentials_galaxy.yml with Ansible Hub credentials 
```
---
controller_credentials:
  - name: "{{orgs  }} {{ env }} Automation Hub Community Repository"
    description: ""
    credential_type: "Ansible Galaxy/Automation Hub API Token"
    organization: "{{ orgs }}"
    inputs:
      url: "https://<demo-hub1-dev.testing.com>/api/galaxy/content/community/"
      token: "<tokenid>"

  - name: "{{ orgs }} {{ env }} Automation Hub Published Repository"
    description: ""
    credential_type: "Ansible Galaxy/Automation Hub API Token"
    organization: "{{ orgs }}"
    inputs:
      url: "https://<demo-hub1-dev.testing.com>/api/galaxy/content/published/"
      token: "<tokenid>"

  - name: "{{ orgs }} {{ env }} Automation Hub RH Certified Repository"
    description: ""
    credential_type: "Ansible Galaxy/Automation Hub API Token"
    organization: "{{ orgs }}"
    inputs:
      url: "https://<demo-hub1-dev.testing.com>/api/galaxy/content/rh-certified/"
      token: "<tokenid>"

```

- controller_credentials_scm.yml
```
---
controller_credentials:
  - name: "{{ orgs }} {{ env }} Gitlab Credential"
    description: "{{ orgs }} {{ env }} Gitlab Credential"
    credential_type: "Source Control"
    organization: "{{ orgs }}"
    inputs:
      username: 'aap-demo'
      ssh_key_data: |
        -----BEGIN OPENSSH PRIVATE KEY-----
        m/42P+vOFTp8oRvLPFnC32+Q5vXB1MqWFmm9sLl4ZB5iz11wbOTkhcpcnlFWNuj8KPjALK
        ZuwwRvc+CnkxSJw83fyk/eD/ubIT387ttROHie8PbBTzvlbAAAAMEA5uTVTYPth5JEnFc/
        8EM/moACiMnJdBAOUt2a94gmyrVgzHtqEiyY36SmqCYf5wYDmauftW2SCzEvwSuKOjwjzQ
        J05b9QjhjB3vRvE91mZxgEqLK8vAzYlF1rUlbbogwR0isuRp6sbxkqM/+Wj5/Z3MxHL3rh
        -----END OPENSSH PRIVATE KEY-----
...
```

- controller_credentials_vault.yml

```
controller_credentials:
  - name: "{{ orgs }} {{ env }} Vault Credentials"
    description: "{{ orgs }} {{ env }} Vault Credentials"
    credential_type: "Vault"
    organization: "{{ orgs }}"
    inputs:
      vault_password: "redhat00"
...
```

- controller_credentials.yaml
```
---
controller_credentials:
  - name: "{{ orgs }} {{ env }} AAP Credentials"
    description: "aap_credentials"
    credential_type: "Red Hat Ansible Automation Platform"
    organization: "{{ orgs }}"
    inputs:
      host: "{{ controller_hostname }}"
      username: "{{ controller_username }}"
      password: "{{ controller_password }}"
      verify_ssl: "{{ controller_validate_certs }}"
...
```

```
ansible-vault encrypt orgs_vars/adonis-example/env/casc-dev/controller_credentials.d/controller_credentials_galaxy.yml
ansible-vault encrypt orgs_vars/adonis-example/env/casc-prd/controller_credentials.d/controller_credentials_galaxy.yml
ansible-vault encrypt orgs_vars/adonis-example/env/casc-dev/controller_credentials.d/controller_credentials_scm.yml
ansible-vault encrypt orgs_vars/adonis-example/env/casc-prd/controller_credentials.d/controller_credentials_scm.yml
ansible-vault encrypt orgs_vars/adonis-example/env/casc-dev/controller_credentials.d/controller_credentials_vault.yml
ansible-vault encrypt orgs_vars/adonis-example/env/casc-prd/controller_credentials.d/controller_credentials_vault.yml
ansible-vault encrypt orgs_vars/adonis-example/env/casc-dev/controller_credentials.d/controller_credentials.yml
ansible-vault encrypt orgs_vars/adonis-example/env/casc-prd/controller_credentials.d/controller_credentials.yml
```

Configure inventory to use group_vars:

```
$ cat inventory
[casc-dev]
demo-ctr1-dev.testing.com

[casc-prd]
demo-ctr1-prd.testing.com
```

Configure ansible.cfg and collections/requirements.yml to install the collections:

```
$ cat ansible.cfg
[defaults]
collections_path = ./collections

[galaxy]
server_list = rh-certified_repo,community_repo

[galaxy_server.rh-certified_repo]
url=https://demo-hub1-dev.testing.com/api/galaxy/content/rh-certified/
token=<tokenid>

[galaxy_server.community_repo]
url=https://demo-hub1-dev.testing.com/api/galaxy/content/community/
token=<tokenid>

cat collections/requirements.yml
---
collections:
  - name: redhat_cop.controller_configuration
  - name: ansible.controller
...

ansible-galaxy collection install -r collections/requirements.yml -p collections/
```

Launch config-controller-filetree.yml for day 0:

```
ansible-playbook config-controller-filetree.yml -i inventory -l casc-dev -e '{orgs: org-example, dir_orgs_vars: orgs_vars, env: casc-dev}'
ansible-playbook config-controller-filetree.yml -i inventory -l casc-prd -e '{orgs: org-example, dir_orgs_vars: orgs_vars, env: casc-prd}'
```

Push your changes to git

```
git add -A
git commit -m "AAP CasC day 0"
git push origin day0
```

Merge day0 into casc-dev and afterwards, casc-dev into casc-pro. Wait until every CI/CD pipelines finish.

Remove old branch and create a new branch to setup webhooks which it will remove after that. Install required collections:

```
git checkout casc-dev
git branch --delete day0
git checkout -b configure_webhooks
ansible-galaxy collection install community.general -p collections/
```

Configure variables inside gitlab_webhook.yml to setup webhook in DEV and after that configure it to setup in PRO:

```
vi gitlab_webhook.yml
ansible-playbook gitlab_webhook.yml -i inventory -l casc-dev
vi gitlab_webhook.yml
ansible-playbook gitlab_webhook.yml -i inventory -l casc-pro
```

Check in git the webhooks created

