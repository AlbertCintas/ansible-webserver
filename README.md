[![Yamllint CI](https://github.com/AlbertCintas/ansible-webserver/workflows/Yamllint%20CI/badge.svg)](https://github.com/AlbertCintas/ansible-webserver/actions/workflows/lint.yml)
[![Molecule CI](https://github.com/AlbertCintas/ansible-webserver/workflows/Molecule%20CI/badge.svg)](https://github.com/AlbertCintas/ansible-webserver/actions/workflows/molecule.yml)

# Ansible Nginx roles

### Summary
This repository contains Ansible roles to deploy an Nginx webserver along a list of vhosts (server blocks) and their web content from a Github repository.
The pipeline contains a mix of Ansible playbooks, Ansible Tower and Github Actions.
A live instance of Ansible Tower is currently running in a Kubernetes in OVH Cloud, using this repository as project backend, so these playbooks can be live tested, along the related actions.

A quick description of the repo:
````
ansible-webserver
    ├── .github                             # Github Actions reside here
    ├── collections                         # Just some requirements for Dynamic Inventory
    ├── inventories                         # Dynamic inventory definition
    ├── provisioners                        # IaC AWS provision of EC2 instances
    ├── roles                               # The place to store roles
    │    ├── nginx                          # This role installs and configures Nginx
    │	 │    ├── defaults					# Default values for role vars
    │	 │ 	  ├── handlers					# Handler scripts for our plays
    │	 │ 	  ├── Molecule                  # Molecule tests definition
    │	 │	  ├── tasks						# Tasks of the role
    │	 │	  ├── templates					# To store template files
    │	 │	  └── var 						# Storage for OS-dependent vars
    │    └── nginx-git-deploy				# This role deploys a website from Git
    └── deploy-webserver.yml                # Main playbook that calls roles


````

### Roles
There are two roles in the repository:
* One role installs and configures Nginx, using custom or default values. This role can also configure Nginx server blocks (vhosts), including SSL certificates and Cloudflare DNS records.
* A simple secondary role, manages the deployment of websites from Github repositories.

These roles work for Linux distributions based on Debian or RedHat, through the use of OS-dependend variables located in each role `vars` folder.

### Inventory
A dynamic inventory connected to AWS cloud has been configured. With current configuration, each time a job is run, a permissionless IAM user configured in Tower checks for the latest status of available hosts. The tags on the EC2 instances in AWS are automatically tied to Ansible groups, for easy inventory.

### Secrets
Along the playbooks in the repository you might find some variables not defined anywhere. These secrets are stored encrypted in the Ansible Tower vault. The secrets are safely stored and can be very easily accessed through a Jinja variable using custom secrets and quickly configurable injectors.

### Actions / CI
Two Github Actions are configured in the repository:
* First one runs Yamllint and scans all the repository to force YAML styling standards. The rules can be configured to suit use case.
* Second one runs Molecule to test the Ansible playbooks. Due to many secrets being stored only in Tower, Nginx role has trouble to be tested. For simplicity of the showcase, the test for the simple deploy role has been configured. The other one has been created, and should work if provided with the needed secrets. 