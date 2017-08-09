# LG201

Table of contents
=================

- [Prerequisites](#prerequisites)
  - [Local docker registry](#local-docker-registry)
- [Task 1](#task-1)
- [Solution](#solution)
  - [Building the applications with Docker](#building-the-applications-with-docker)
  - [Deploying the containers with vagrant](#deploying-the-containers-with-vagrant)
  - [Issues](#issues)
  - [Duplicated provisioning](#duplicated-provisioning)

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

This won't be covered by the Ansible automation so will need to be setup before running any commands.

If you already have a suitable docker registry, go ahead and skip this step, but change the `docker_registry` variable in the Ansible playbook files.

To set up a docker registry on the host machine run

```
$ docker run -d -p 5000:5000 --restart=always --name registry registry:2
```

**Note:** for speed, this registry is not configured securely and is not production-ready in this form. In a real-world scenario, the registry should be secured with TLS and require authentication if it's externally accessible.

## Task 1

This is the solution to task 2, to see the solution to task 1 checkout tag `task1` in this repository.

Alternatively you can view it in the browser here [https://github.com/hipyhop/LG201/tree/task1](https://github.com/hipyhop/LG201/tree/task1).

## Solution

The solution to Task 2 involves building the two application servers as Docker images, and running them as containers on the web server hosts.

I've separated this into two sections to simulate a typical CI/CD pipeline building the docker images when changes are published, and a manual deployment stage using Ansible/vagrant automation.

### Building the applications with Docker

In lieu of a CI system, such as Jenkins, I've added build.sh scripts to the forked application repository [https://github.com/hipyhop/Hello](https://github.com/hipyhop/Hello).

To simulate a build you can clone the repository and run `./build.sh` in the respective directories to build the application images. Then, if successful, push the resulting image to the specified Docker registry.

```sh
EXPORT DOCKER_TAG=latest
EXPORT DOCKER_REGISTRY=localhost:5000

./build.sh
```

Ideally these would be added to the CI/CD pipeline as build steps, hopefully with additional testing/validation steps between the building the image, and pushing it to the remote registry.

Then on a successful build the new image(s) will be ready to be deployed.


### Deploying the containers with vagrant

Launching the three required vagrant guests and provisioning them with the required configuration is as simple as running:

```
vagrant up
```

**What does this do?**

  1. Launches three `ubuntu/trusty64` guests within the same private network and creates an Ansible inventory file.
  1. This inventory files groups two hosts as docker-hosts, and the other as a load balancer.
  1. Then the Ansible playbook `common.yml` is used to provision these hosts. As Ansible files are designed to be human-readable, I recommend reading this file for more information on the provisioning process.
  1. The application servers are deployed as containers from the docker registry using the tag `latest`. Each docker_host runs one *go_app*, and one *node_app* instance.
  1. The load balancer is configured with two ACLs to route incoming L7 traffic to the appropriate backend based on the request path.

If it any point in the future more docker hosts need to be added, just edit the `N_DOCKERHOSTS` variable in the `Vagrantfile`.
The next time vagrant up is run the Ansible inventory file will be updated.

See lines 46-48 and 54-56 of `haproxy/haproxy.cfg.j2` to see how the HAProxy configuration is dynamically generated from the Ansible inventory file.

### Issues

#### Duplicated provisioning

In the current setup, vagrant runs the Ansible playbook after each host definition, resulting in a lot of duplicated effort.

This isn't a huge problem, since Ansible modules are smart about the defined tasks and can check if anything needs doing, however this still introduces a significant delay to the up-provision loop.

It looks like I'm not the only person to face this issue: [https://github.com/mitchellh/vagrant/issues/1784](https://github.com/mitchellh/vagrant/issues/1784)
A few suggestions were offered, but I couldn't find a decent solution within a sensible time..

The best way around it appears to be separating the spin-up and provisioning steps:

```
$ vagrant up --no-provision && vagrant provision
```
