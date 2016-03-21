vcc-base
===

[![Build Status](https://travis-ci.org/announce/vcc-base.svg?branch=master)](https://travis-ci.org/announce/vcc-base)


## Introduction

This Ansible playbook constructs VCC Research Sandbox into single server node.


## Requirements

#### Environment
* Host: Ubuntu 14.04 LTS (Trusty Tahr)
* Client: Mac OS X 10.11

#### Runtime
- Python v2.7
- Vagrant v1.8 (optional)
- ssh-copy-id (optional)

```bash
# Example on OS X with Homebrew
xcode-select --install \
&& brew update \
&& brew install \
  python \
  ssh-copy-id \
  Caskroom/cask/vagrant
```


## Setups

#### Setup Client

* Install Ansible and its 3rd party packages to your local client

```bash
$ pip install --requirement requirements.txt
$ ansible-galaxy install --role-file=role_packages.yml
```

* Edit ``~/.ssh/config`` corresponding to hosts (No need for Vagrant)

```bash
$ cat <<EOF >> ~/.ssh/config
Host vcc-base-init001
    User root
    HostName example.com
    IdentitiesOnly yes

Host vcc-base-general001
    User admin
    HostName example.com
EOF
```

#### Setup Vagrant Server for local development environment

* Run Vagrant commands after setting up client

```bash
$ vagrant up  # Run this for the first time
$ vagrant provision  # Run this when `vagrant up` had problem due to network error, etc
$ vagrant ssh  # Login to the Vagrant Machine
$ vagrant suspend  # Stop the Vagrant Machine
$ vagrant resume  # Start the Vagrant Machine
$ vagrant destroy --force && vagrant up  # Run this to reset everything
```

#### Setup Host Server for production environment

* Run commands below, and wait for a few hours or so (up to the server spec)
    * Use `--check` option if you prefer to run as dry-run mode

```bash
$ ssh-copy-id -i ~/.ssh/id_rsa.pub root@example.com
$ ansible-playbook site.yml --inventory-file hosts --private-key="~/.ssh/id_rsa" --limit="init_server"
$ ansible-playbook site.yml --inventory-file hosts --private-key="~/.ssh/id_rsa" --limit="general_server"
```

#### How to know what are these commands above doing?

```bash
$ ansible-playbook site.yml --list-tasks
$ ansible-playbook site.yml --list-hosts
```

## Data

* https://dl.dropboxusercontent.com/u/6998388/vcc_data/vcc-database.dump.bz2
