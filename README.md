[![Yamllint CI](https://github.com/AlbertCintas/ansible-webserver/workflows/Yamllint%20CI/badge.svg)](https://github.com/AlbertCintas/ansible-webserver/actions/workflows/lint.yml)
[![Molecule CI](https://github.com/AlbertCintas/ansible-webserver/workflows/Molecule%20CI/badge.svg)](https://github.com/AlbertCintas/ansible-webserver/actions/workflows/molecule.yml)

# Ansible Nginx roles

### Summary
This repository contains Ansible roles to provision servers, deploy in them Nginx along a list of vhosts (server blocks) and deploy their web content from a GitHub repository.
The pipeline contains a mix of Ansible playbooks, Ansible Tower and GitHub Actions.
A live instance of Ansible Tower has been deployed, using this repository as project backend, so these playbooks can be live tested, along the related actions.

The roles (and helpers) allow to:
* Provision EC2 instances on AWS and manage the hosts using a dynamic inventory.
* Install and configure Nginx on Debian/RedHat based distros. Extra distros can be added with minimal effort.
* Configure one or several vhosts in Nginx, including the creation of SSL certificates and automatic registry of DNS entries using the Cloudflare API.
* Deploy a website for each configured vhost from a GitHub tagged release. Deploys triggered by webhooks in Tower, triggered by a release publication.

The result of these steps: https://awonderfultestdomain.xyz:8080/

A quick description of the repo:
````
ansible-webserver
    ├── .github                             # GitHub Actions reside here
    ├── collections                         # Just some requirements for Dynamic Inventory
    ├── inventories                         # Dynamic inventory definition
    ├── provisioners                        # IaC AWS provision of EC2 instances
    ├── roles                               # The place to store roles
    │    ├── nginx                          # This role installs and configures Nginx
    │    │    ├── defaults                  # Default values for role vars
    │    │    ├── handlers                  # Handler scripts for our plays
    │    │    ├── Molecule                  # Molecule tests definition
    │    │    ├── tasks                     # Tasks of the role
    │    │    ├── templates                 # To store template files
    │    │    └── var                       # Storage for OS-dependent vars
    │    └── nginx-git-deploy               # This role deploys a website from Git
    └── deploy-webserver.yml                # Main playbook that calls roles


````
> :warning: This repository is just a way of quickly showcasing some abilities and way of thinking on the basis of a requested task. In no way, this is intended to be any kind of productive pipeline design or suggestion. Many things have been done having best practices in mind, while in many other parts corners have been cut for the sake of simplicity or were made just for showoff while having not much sense. Perfect idempotency, long term management or complex scenarios are not fit for some bits of code in this repo. It is known and expected.

### Roles
There are two roles in the repository:
* One role installs and configures Nginx, using custom or default values. This role can also configure Nginx server blocks (vhosts), including SSL certificates and Cloudflare DNS records.
* A simple secondary role, manages the deployment of websites from GitHub repositories.

These roles work for Linux distributions based on Debian or RedHat, through the use of OS-dependent variables located in each role's `vars` folder.

### Ansible Tower
These roles and pipelines have been configured with Ansible Tower in mind. An Ansible AWX instance has been deployed using Ansible in a Kubernetes cluster in OVH Cloud. It is currently listening for webhook requests and repository or inventory updates, using these playbooks and configurations. All configurations are stored in the repository, except for credentials and secrets, which are stored in Tower vault. More info: https://github.com/ansible/awx-operator

### Inventory
A dynamic inventory connected to AWS cloud has been configured in `inventories`. With current configuration, each time a job is run, a permission-less IAM user configured in Tower checks for the latest status of available ec2 instances and updates local inventory. The tags on the EC2 instances in AWS are automatically tied to Ansible groups locally, allowing for great flexibility.

### Secrets
Along the playbooks in the repository, you might find some variables not defined anywhere. These secrets are stored encrypted in the Ansible Tower vault. The secrets are safely stored and can be very easily accessed through a Jinja variable using custom secrets and quickly configurable injectors. SSH keys, or AWS IAM and Cloudflare credentials are stored and shared in this way.

### Actions / CI / Testing
Two GitHub Actions are configured in the repository:
* First one runs Yamllint and scans all the repository to force YAML styling standards. The rules can be configured to suit use case and run before all pull requests for mandatory styling.
* Second one runs Molecule to test the Ansible playbooks. Due to many secrets being stored only in Tower, Nginx role has trouble being tested, so for the sake of simplicity, we're only Molecule testing the simpler of the two roles, to serve as example. Both tests should work if provided with the needed secrets, through GitHub Secrets or an external shared vault.
* In related repository https://github.com/AlbertCintas/my-cv-in-html, another action for deploy is configured. In that repository, an action deploying a release through Tower, using these roles, is configured.

### Provisioning
Provisioning has been configured, allowing Ansible to interact with AWS to manage resources. IAM roles can be configured and keys and credentials safely stored and used from the vault to perform many tasks in the cloud. In our case, we are using it to deploy simple EC2 instances and any related security group, to use in our webserver example. More complex scenarios can be achieved with similar simple YAMLs. Not as potent as Terraform, but way easier.

