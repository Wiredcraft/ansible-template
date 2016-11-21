# Project

This `README` file is meant to provide relevant information to whomever is gonna run the ansible commands. 

## Pre-requisite

Need:

- Ansible 2.1.0

## Installation

- Fetch the roles

```
ansible-galaxy install -r requirements.yml -p roles
```

## Setup & deployment

- A to Z setup

```
ansible-playbook -i inventory.staging -e @vars.staging --ask-vault-pass main.yml
```

- Install only the db components

```
ansible-playbook -i inventory.staging -e @vars.staging --ask-vault-pass setup-db.yml
```

- Deploy only the app (considering the setup is done already)

```
ansible-playbook -i inventory.staging -e @vars.staging --ask-vault-pass deploy-app.yml
```
