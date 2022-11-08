# Ansible Collection - automationiberia.casc_setup


## Roles

|Role Name|Description|
|:---:|:---:|
|filetree_create| Create a basic Configuration as Code filetree to setup the day zero|
|gitlab_setup| Create the repository, Upload the content crated, for example, by filtree_create role of this collection, Create and protect branches and configure the webooks|
|github_setup| WIP |

### Role Variables

You can see Role variables in the README of each role

## Steps to setup CasC

Install this collection and configure a playbook to use it (there is an example in this repository inside the example directory. You need to change the variables defined in the playbook itself or use extra-vars).

Launch the playbook to init git and create the initial basic content. To to that, use tags git-init, filetree_create, git-push and git-branches:

```
ansible-playbook gitlab_setup.yml --tags git-init,filetree_create,git-push,git-branches
```

Git repository is ready to launch the first CD and set the controller configured from CasC approach. As day 0, it is needed to do it from CLI to, after that, configure the pipelines, webhooks and everthing needed to have a complete CasC for the controller.

Clone the created repository and follow the steps in readme:
