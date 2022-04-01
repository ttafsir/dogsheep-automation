# Dogsheep Automation

This repo contains Ansible automation playbooks to get a personal dogsheep project up and running. The playbooks are written in [Ansible](https://www.ansible.com/) to follow instructions from Simon Willison's [Weeknotes](https://simonwillison.net/2021/Aug/22/weeknotes-dogsheep/).

The playbook will:

* Install Python, NGINX, and SQLite
* Install and Configure Certbot for NGINX
* Create a user and virtualenv for service
* Install datasette and its dependencies
* Create a datasette service
* Create database files based on your input in the [`config.yaml`](./ansible/config.yml) file
* Create NGINX configurations
* Limit access to your dogsheep instance with a username and password you can set via environment variables or through prompts
* test the generated NGINX configuration and validate your dogsheep instance is working

## Requirements

* Ansible
* Python 3.6+

**TESTED ON:** DigitalOcean Ubuntu 20.04 (LTS)

## Install Requirements

```sh
pip install -r requirements.txt
```

## :gear: Configuration

### Update your inventory

update the [`hosts`](./ansible/hosts) inventory file to point the IP address or hostname of your Digital Ocean droplet. I had already set a DNS record to point `dogsheep.ttafsir.me` to my droplet's IP address. You'll want to do the same for your other droplet.

```ini
[dogsheep_hosts]
dogsheep.ttafsir.me ansible_user=root
```

> **Note:** ansible_user is the user you want to use to connect to your droplet using SSH.

### Export Environment Variables for your Login

By default, the configuration value of `datasette_enable_auth` is set to `true` to require authentication to your instance. Set the following environment variables to set the username and password for your instance.

```sh
export DATASETTE_USERNAME=<username>
export DATASETTE_PASSWORD=<password>
```

### Update [`config.yaml`](./ansible/config.yml) for your instance

```yaml
---
dogsheep_site_title: Tafsir's personal data service
dogsheep_site_description: Personal data collection from public services
dogsheep_user: dogsheep
dogsheep_user_shell: /bin/bash
dogsheep_dbs:
  - twitter
  - swarm
  - photos
  - pocket
  - hacker-news
  - memories
```

The `dogsheep_dbs` list is the list of databases you want to create. These will be created as empty SQLite database files in `WAL mode`.


### Update your Certbot configuration

Your email address is used to register and agree to the certbot TOS.

```yaml
certbot_email: ttafsir@gmail.com
certbot_domain: dogsheep.ttafsir.me
```

> **Note:** I had to set up DNS records to point my DigitalOcean droplet to `dogsheep.ttafsir.me` before I could get certbot to work.


### Additional datasette settings

The configuration file `config.yaml` has a few additional settings that you can set to customize your instance. Any key/value pair under `datasette_settings` will be passed with the `--setting` flag to datasette when the service is created.

Below are the settings that are currently configured:

```yaml
datasette_settings:
  sql_time_limit_ms: 20000
  facet_time_limit_ms: 5000
  template_debug: 1
  force_https_urls: 1
```

## :rocket: Deployment

Assuming you have your public key setup to allow you to SSH to your droplet, you can run the following command to deploy the service.

```sh
cd ansible
ansible-playbook pb-deploy.yaml
```

Optionally, you can pass the `-k` flag to prompt for your SSH password.

```sh
cd ansible
ansible-playbook pb-deploy.yaml -k
```
