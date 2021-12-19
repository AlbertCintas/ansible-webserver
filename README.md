[![Yamllint CI](https://github.com/AlbertCintas/ansible-webserver/workflows/Yamllint%20CI/badge.svg)](https://github.com/AlbertCintas/ansible-webserver/actions/workflows/lint.yml)
[![Molecule CI](https://github.com/AlbertCintas/ansible-webserver/workflows/Molecule%20CI/badge.svg)](https://github.com/AlbertCintas/ansible-webserver/actions/workflows/molecule.yml)

# Ansible Nginx roles

### Summary
This repository contains Ansible roles to deploy an Nginx webserver along a list of vhosts (server blocks) and their web content from a Github repository.
The pipeline contains a mix of Ansible playbooks, Ansible Tower and Github Actions.
A live instance of Ansible Tower is currently running in a Kubernetes in OVH Cloud, using this repository as project backend, so these playbooks can be live tested, along the related actions.

The roles (and helpers) allow to:
* Provision EC2 instances on AWS and store the hosts into a dynamic inventory.
* Install and configure Nginx on Debian/RedHat based hosts.
* Configure one or several vhosts in Nginx, including the creation of SSL certificates and automatic registry od DNS entries in CloudFlare.
* Deploy a website for each configured vhost from a github tagged release. Deploys triggered by webhooks in Tower, called from Github.

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
> :warning: This repository is just a way of quickly showcasing some abilities and way of thinking on the basis of a requested task. In no way this is intended to be any kind of productive pipeline design or suggestion. Many things have been done having best practices on mind, while in many other parts corners have been cut for the sake of simplicity or were made just for showoff while having not much sense (ie. provisioning from a deploy pipeline).

### Roles
There are two roles in the repository:
* One role installs and configures Nginx, using custom or default values. This role can also configure Nginx server blocks (vhosts), including SSL certificates and Cloudflare DNS records.
* A simple secondary role, manages the deployment of websites from Github repositories.

These roles work for Linux distributions based on Debian or RedHat, through the use of OS-dependend variables located in each role `vars` folder.

### Inventory
A dynamic inventory connected to AWS cloud has been configured in `inventories`. With current configuration, each time a job is run, a permissionless IAM user configured in Tower, checks for the latest status of available hosts in AWS and updates local inventory. The tags on the EC2 instances in AWS are automatically tied to Ansible groups.

### Secrets
Along the playbooks in the repository you might find some variables not defined anywhere. These secrets are stored encrypted in the Ansible Tower vault. The secrets are safely stored and can be very easily accessed through a Jinja variable using custom secrets and quickly configurable injectors. SSH keys, or AWS IAM and Cloudflare credentials are stored and shared in this way.

### Actions / CI
Two Github Actions are configured in the repository:
* First one runs Yamllint and scans all the repository to force YAML styling standards. The rules can be configured to suit use case.
* Second one runs Molecule to test the Ansible playbooks. Due to many secrets being stored only in Tower, Nginx role has trouble to be tested. For simplicity of the showcase, the test for the simple deploy role has been configured. The other one has been created, and should work if provided with the needed secrets. 