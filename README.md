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

Install this collection and configure a playbook to use it (there is an example in this repository inside the example directory. You need to change the variables defined in the playbook itself or use extra-vars).

Launch the playbook to init git and create the initial basic content. To to that, use tags git-init, filetree_create, git-push and git-branches:

```
ansible-playbook gitlab_setup.yml --tags git-init,filetree_create,git-push,git-branches
```

Git repository is ready to launch the first CD and set the controller configured from CasC approach. As day 0, it is needed to do it from CLI to, after that, configure the pipelines, webhooks and everthing needed to have a complete CasC for the controller.

Clone the repository and create a day 0 branch:

```
git clone git@gitlab.testing.com:acme/ansible-controller/org-example.git

git checkout -b casc-day0
```

Configure the controller access inside group_vars for day 0. If you want upload them to git, you must encyrpt them with vault. Remember that they are only for day 0 from CLI so they are not actually needed in the git repository for CasC flow:

```
vi group_vars/casc-dev/configure_connection_controller_credentials.yml
vi group_vars/casc-prd/configure_connection_controller_credentials.yml

ansible-vault encrypt group_vars/casc-dev/configure_connection_controller_credentials.yml
ansible-vault encrypt group_vars/casc-prd/configure_connection_controller_credentials.yml
```

Configure every credential needed essentially for casc (there are variables to change in the following examples). Remember that it is needed to save the credentials propertly so it must encrypt every credential with vault before push to a git repository: 

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
ansible-vault encrypt orgs_vars/org-example/env/casc-dev/controller_credentials.d/controller_credentials_galaxy.yml
ansible-vault encrypt orgs_vars/org-example/env/casc-prd/controller_credentials.d/controller_credentials_galaxy.yml
ansible-vault encrypt orgs_vars/org-example/env/casc-dev/controller_credentials.d/controller_credentials_scm.yml
ansible-vault encrypt orgs_vars/org-example/env/casc-prd/controller_credentials.d/controller_credentials_scm.yml
ansible-vault encrypt orgs_vars/org-example/env/casc-dev/controller_credentials.d/controller_credentials_vault.yml
ansible-vault encrypt orgs_vars/org-example/env/casc-prd/controller_credentials.d/controller_credentials_vault.yml
ansible-vault encrypt orgs_vars/org-example/env/casc-dev/controller_credentials.d/controller_credentials.yml
ansible-vault encrypt orgs_vars/org-example/env/casc-prd/controller_credentials.d/controller_credentials.yml
```

Configure galaxy credentials to use it for our organization:

```
$ cat orgs_vars/org-example/env/common/controller_organizations.d/controller_organizations.yaml

---
controller_organizations:
  - name: "adonis-demo"
    description: "Organization adonis-demo"
    galaxy_credentials:
      - "{{ orgs }} {{ env }} Automation Hub Community Repository"
      - "{{ orgs }} {{ env }} Automation Hub Published Repository"
      - "{{ orgs }} {{ env }} Automation Hub RH Certified Repository"

```

Configure inventory to use group_vars:

```
$ cat inventory
[casc-dev]
demo-ctr1-dev.testing.com

[casc-prd]
demo-ctr1-prd.testing.com
```

Configure ansible.cfg and collections/requirements.yml files to install the collections:

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

$ cat collections/requirements.yml
---
collections:
  - name: redhat_cop.controller_configuration
  - name: ansible.controller
...

$ ansible-galaxy collection install -r collections/requirements.yml -p collections/
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
git push origin casc-day0
```

Merge casc-day0 into casc-dev and afterwards, casc-dev into casc-pro. Wait until every CI/CD pipelines finish.

Remove old branch and create a new branch to setup webhooks which it will remove after that. Install required collections:

```
git checkout casc-dev
git branch -D casc-day0
git pull
git checkout -b configure_webhooks
ansible-galaxy collection install community.general -p collections/
ansible-galaxy collection install automationiberia.casc_setup -p collections/
```

Configure variables inside gitlab_webhook.yml (or use extra-vars) to setup webhook in DEV and after that configure it to setup in PRO:

```
vi gitlab_webhook.yml
ansible-playbook gitlab_webhook.yml -i inventory -l casc-dev
vi gitlab_webhook.yml
ansible-playbook gitlab_webhook.yml -i inventory -l casc-pro
git checkout casc-dev
git branch -D configure_webhooks
git checkout -- gitlab_webhook.yml
```

Check in git the webhooks created

