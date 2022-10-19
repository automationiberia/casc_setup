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

Install this collection and configure a playbook to use it (playbook example in this repo inside example directory).

Launch playbook to init git and create the initial filetree. To to that, use tags git-init and filetree_create:

```
ansible-playbook gitlab_setup.yml --tags git-init,filetree_create,git-push,git-branches
```

You already have your git repository ready to launch your first CD and get the controller configured from CasC approach. As day 0, you need to do it from CLI to, after that, configure the pipelines, webhooks and everthing needed to have a complete CasC for your controller. 