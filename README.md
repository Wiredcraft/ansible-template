# Ansible Template

Each project usually needs a collection of tasks to build, deploy and perform regular operations during its lifetime.

## Usage

2 folders are present in this repository:

- `ansible-example`; offers a view with (fake) data, hosts and vars to illustrate the conventions
- `ansible`; should be blend enough to be copied in place in any project and be the start of the devops related work for the ansible playbooks and tasks.

## Conventions

The conventions are the following:

- An `ansible` folder at the root of the primary repository
- Adapted `.gitignore` files to handle propely ansible (roles, logs, retry files)
- 1 inventory file per environment (e.g. `inventory.dev`, `inventory.staging`)
- One `main.yml` playbook to run the whole setup / provisioning from end-to-end
- A collection of `setup-*.yml` and `deploy-*.yml` files
- Groups named `all-dev`, `all-staging`, `all-prod` to share variables per environment, 
- `vars.dev` and `vars.staging` files at the root
- `ansible.cfg` at the root
- Vaults when needing to protect data - no password should be kept in-clear
- Project specific roles prefixed with the project name in `roles`
- `requirements.yml` file with the list of roles to install
- A comprehensive README file in the `ansible` foldere to explain how things are to be ran

## Details

### Ansible folder & configuration

Each project need its collection of scripts / tasks to do the devops related operations. 

You must choose in accordance with the project owner the repository most suitable to store this folder.

This folder will contain ALL the tasks / playbooks / config needed to build everything for the project; from the dev environment until the regular release of the code in production.

Make sure you add an `ansible.cfg` file just for the sake of having your best practices config available at all time; caching, ssh agent, etc.


### Gitignore

We do not want to store useless data, logs, community roles, be sure you update the `.gitignore` file. See the one from this repo, it covers some edge cases.

### Inventory files

Let's call an inventory an inventory and agree on the naming convention `inventory.ENV`.

`ENV` is then to be one of either `dev`, `staging`, `prod` and so on.

The inventory files must share the **same groups** indenpendently of the environment. Playbooks and tasks, should only "care" about those groups and should never address an individual host.

See the example in the `ansible-example` folder.

### Playbooks

Ansible is able to handle idempotence, but it may sometime be quite slow - due to network or number of tasks. So we prefer to split playbooks and use `include:` whenever possible.

Several playbooks are usually found depending of the projects:

- `main.yml`: is the full A to Z playbook, doing everything from setup to deployment. It usually calls other plugins to do the work
- `setup-*.yml` playbooks: handle the setup (installation of the services and so-on), 
- `deploy-*.yml` playbooks: handle the deployment (depoyment of the code per-se). 

Each of those playbooks must be able to run "independently".

**Note**: Some playbooks are only meant to be executed on subset of the architecture, yet need to be able to refer to other components (e.g. an APP server needs to know the details of the DB servers - even when the task is about deploying the code). You will then need to use the **ping trick** to ensure your inventories are "pre-loaded" when you execute commands. See the example in the `setup-app.yml` playbook in the `ansible-example` folder.

### Variables

Variables comes at various times and from various sources (non-weigthed list):
- `host_vars` and `group_vars` folder
- playbooks and tasks
- roles / vars and default folders
- extra files and command lines

For `host_vars` and `group_vars` - prefer the use of **folders** instead of files to store host and groups vars. It will simplify you life when you need to add the support for other files, or vault.

#### The `all-XXX` groups

The `all` folder in `group_vars` is a catch-all group and will get its values offered to all the hosts defined. 
We often use additional similar groups - but applied at the environement level (prod vs. staging vs. dev).
The naming convention for those files are `all-ENV`; e.g.
- `all-dev` for the dev environment
- `all-staging` for the staging environment

This is however **NOT** a built-in feature of ansible, and we need to define those groups explicitely in the inventory files.

```
# Define the all-dev group as a group of .. groups
[all-dev:children]
group1
group2
```

#### The extra vars

Variables defined in the `group_vars` and `host_vars` are applied to hosts - they are **NOT** available globally. To have variables available globally - and not be tied to a specific host, you need to pass them as an extra var.

Either pass them as `-e "my_global_var=toto"` or via a file `-e @vars.dev`. We prefer the file approach for reason of clarity.

Such approach allows then to use conditional `when:` even at the playbook level, before even having "selected" a pool of servers.


- Run the playbook as such: `ansible-playbook -i inventory.dev -e @vars.dev main.yml`
- `vars.dev`; define the vars specific to the dev environment

```
---
env: dev
```

- `main.yml`; without having yet selected any host

```
---
- include: dev-only.yml
  when: env is defined and env == "dev"
```

- `dev-only.yml`; a random playbook aimed at being executed only on dev

```
- hosts: all
  tasks:
    - name: something to run on all the hosts, for dev only
      ping: {}
```

### Vaults

Variables set in vaults are encrypted; the resulting vault file is changing 100% of the time upon even the smallest change. It can be confusing. 

To limit the amount of change, you will want to create several vault files; that are having different period of a refresh. Some values will never change, other may change on a weekly / monthly basis. 

Typically, you will move long lasting variables together. 

e.g. Split the SSL certs outside

```
group_vars/all-staging/vault
group_vars/all-staging/vault-ssl-cert
```
