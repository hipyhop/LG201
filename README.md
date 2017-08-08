# LG201

Table of contents
=================

  * [Prerequisites](#prerequisites)
    * [Local Docker registry](#local-docker-registry)
  * [Task 1](#task-1)
  * [Solution](#solution)
    * [Issues](#issues)
      * [Duplicated provisioning](#duplicated-provisioning)

## Prerequisites
To run these examples the following must be configured on the local host:
  * [Vagrant](https://www.vagrantup.com/downloads.html)
  * [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
  * [Ansible](http://docs.ansible.com/ansible/latest/intro_installation.html)
  * [Docker](https://www.docker.com/get-docker)

All solutions were implemented on OS X Sierra and due to time constraints have not been tested on any other platforms.
The benefit of using virtualisation technologies here means it should work on other platforms with no changes, however YMMV.

### Local docker registry

For this example, a local docker registry will be used for speed and ease of development.

This won't be covered by the ansible automation so will need to be setup before running any commands.

If you already have a suitable docker registry, go ahead and skip this step, but change the `docker_registry` variable in the ansible playbook files.

To set up a docker registry on the host machine run

```
$ docker run -d -p 5000:5000 --restart=always --name registry registry:2
```

**Note:** for speed, this registry is not configured securely and is not production-ready in this form. In a real-world scenario, the registry should be secured with TLS and require authentication if it's externally accessible.

## Task 1

This is the solution to task 2, to see the solution to task 1 checkout tag `task1` in this repository.

Alternatively you can view it in the browser here [https://github.com/hipyhop/LG201/tree/task1](https://github.com/hipyhop/LG201/tree/task1).

## Solution

Launching the three required vagrant guests and provisioning them with the required configuration is as simple as running:

```
vagrant up
```

**What does this do?**

  1. Launches three `ubuntu/trusty64` guests within the same private network and creates an Ansible inventory file.
  1. This inventory files groups two hosts as docker-hosts, and the other as a load balancer.
  1. Then the Ansible playbook `common.yml` is used to provision these hosts. See this file for more information on the provisioning process.

If it any point in the future more docker hosts need to be added, just edit the `N_DOCKERHOSTS` variable in the `Vagrantfile`.
The next time vagrant up is run the Ansible inventory file will be updated.

See lines 33-35 of `haproxy/haproxy.cfg.j2` to see how the HAProxy configuration is dynamically generated from the Ansible inventory file.

### Issues

#### Duplicated provisioning

In the current setup, vagrant runs the Ansible playbook after each host definition, resulting in a lot of duplicated effort.

This isn't a huge problem, since Ansible modules are smart about the defined tasks and can check if anything needs doing, however this still introduces a significant delay to the up-provision loop.

It looks like I'm not the only person to face this issue: [https://github.com/mitchellh/vagrant/issues/1784](https://github.com/mitchellh/vagrant/issues/1784)
A few suggestions were offered, but I couldn't find a decent solution.

The best way around it appears to be separating the spin-up and provisioning steps:

```
$ vagrant up --no-provision && vagrant provision
```
