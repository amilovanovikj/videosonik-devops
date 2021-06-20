# Videosonik App Build and Release Automation

This repository contains Ansible and Docker scripts that automate the build and deploy process of a [Java Spring](https://github.com/amilovanovikj/videosonik-backend) and [React](https://github.com/amilovanovikj/videosonik-frontend) app, as well as Ansible scripts for automating the server configuration process. The original Java Spring backend app can be found at [vavramovski/videosonik-new](https://github.com/vavramovski/videosonik-new) and the original React frontend app can be found at [vavramovski/videosonik-frontend](https://github.com/vavramovski/videosonik-frontend). 

Huge thanks to [vavramovski](https://github.com/vavramovski) for letting me use these two apps for my project.

## What is implemented

- All package and deploy operations for the two apps, as well as server setup, are done using [Ansible playbooks](https://github.com/amilovanovikj/videosonik-devops/tree/master/playbooks).
- Five VMs (created using this [Vagrantfile](https://github.com/amilovanovikj/videosonik-devops/blob/master/Vagrantfile)) are used for hosting the five different types of servers:
  - **Backend**: The Java Spring backend web app.
  - **Frontend**: The React frontend web app.
  - **Database**: The PostgreSQL database for the backend app.
  - **Build**: The server used to build the Docker images from the Git repositories of the apps and push them to a private Docker registry.
  - **Repository**: The machine hosting a private Git server for the two apps and this repository, as well as a private Docker registry for the Docker images.
- The Ansible playbooks include:
  - [all-configure.yml](https://github.com/amilovanovikj/videosonik-devops/blob/master/playbooks/all-configure.yml): Install Docker and Python SDK for Docker on all hosts, enable insecure communication with private Docker registry.
  - [repository-configure.yml](https://github.com/amilovanovikj/videosonik-devops/blob/master/playbooks/repository-configure.yml): Run Git server and Docker registry on the 'registry' host as Docker containers, set up SSH communication between Ansible master and Git server.
  - [build-configure.yml](https://github.com/amilovanovikj/videosonik-devops/blob/master/playbooks/build-configure.yml): Install Git on the 'build' host and set up SSH communication to the private Git server.
  - [database-deploy.yml](https://github.com/amilovanovikj/videosonik-devops/blob/master/playbooks/database-deploy.yml): Run the PostgreSQL service on the 'database' host in a Docker container, create the necessary database and user for database access. These properties are hidden in an Ansible vault file.
  - [frontend-package.yml](https://github.com/amilovanovikj/videosonik-devops/blob/master/playbooks/frontend-package.yml): Pull frontend repository from private Git server, use Jinja2 templates to change default app configuration, build a Docker image from the frontend app and push it to the private Docker registry. Runs on 'build' host.
  - [backend-package.yml](https://github.com/amilovanovikj/videosonik-devops/blob/master/playbooks/backend-package.yml): Pull backend repository from private Git server, use Jinja2 templates to change default app configuration, build a Docker image from the backend app and push it to the private Docker registry. Runs on 'build' host.
  - [frontend-deploy.yml](https://github.com/amilovanovikj/videosonik-devops/blob/master/playbooks/frontend-deploy.yml): Deploy a Docker container of the frontend app on the 'frontend' host from the previously built Docker image, using Docker compose. Multiple instances of the app are started, alongside an Nginx load balancer to distribute the traffic between them.
  - [backend-deploy.yml](https://github.com/amilovanovikj/videosonik-devops/blob/master/playbooks/backend-deploy.yml): Deploy a Docker container of the backend app on the 'backend' host from the previously built Docker image, using Docker compose. Multiple instances of the app are started, alongside an Nginx load balancer to distribute the traffic between them.
- [Ansible roles](https://github.com/amilovanovikj/videosonik-devops/tree/master/roles) are used for installing Docker, Git and other auxilary packages on the servers.
- Various [Ansible templates](https://github.com/amilovanovikj/videosonik-devops/tree/master/playbooks/templates) are used to inject variables at playbook runtime.
- [Ansible vault](https://github.com/amilovanovikj/videosonik-devops/tree/master/playbooks/vault) is used to store secret variables.

