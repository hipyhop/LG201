# LG201

Table of contents
=================

  * [Prerequisites](#prerequisites)
  * [Solution](#solution)

## Prerequisites
To run these examples the following must be configured on the local host:
  * [Vagrant](https://www.vagrantup.com/downloads.html)
  * [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
  * [Ansible](http://docs.ansible.com/ansible/latest/intro_installation.html)

All solutions were implemented on OS X Sierra and due to time constraints have not been tested on any other platforms.
The benefit of using virtualisation technologies here means it should work on other platforms with no changes, however YMMV.

## Solution

Launching the three required vagrant guests and provisioning them with the required configuration is as simple as running:

```
vagrant up
```

**What does this do?**

  1. Launches three `ubuntu/trusty64` guests within the same private network and creates an Ansible inventory file.
  1. This inventory files groups two hosts as web servers, and the other as a load balancer.
  1. Then the Ansible playbook `common.yml` is used to provision these hosts. See this file for more information on the provisioning process.
