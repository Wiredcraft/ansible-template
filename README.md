# Ansible Template

Each project usually needs a collection of tasks to build, deploy and perform regular operations during its lifetime.

## Conventions

The conventions are the following:

- An `ansible` folder at the root of the primary repository
- Adapted `.gitignore` files to handle propely ansible (roles, logs, retry files)
- 1 inventory file per environment (e.g. `inventory.dev`, `inventory.staging`)
- One `main.yml` playbook to run the whole setup / provisioning from end-to-end;
  - this playbook is usually not called all the time and sub-files are prefered for speed reason.
- A collection of `setup-*.yml` and `deploy-*.yml` files;
  - to handle either the setup (installation of the services and so-on), 
  - or the deploy (deployment of the code per-se) on the boxes.
- Similar to the group `all`, we use groups named `all-dev`, `all-staging`, `all-prod` to share variables per environment. 
  - This groups are defined in the inventory files; they are **NOT** automatically created
- `vars.dev` and `vars.staging` files at the root
  - this allows to have variables available before even having checked any host. Useful for environement's type of vars; or to use in `when: ` within a playbook
  - they need to be loaded manually via `-e @vars.dev` on playbook execution
- `ansible.cfg` just for the sake of having your best practices config available at all time; caching, ssh agent, etc.
- Use vaults when needing to protect data - no password should be kept in-clear
  - split the data storage in various vault files if needed - data you know will not change often vs. frequently updated ones. E.g. `vault-ssl-cert` vs. `vault`