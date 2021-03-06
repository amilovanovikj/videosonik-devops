# Videosonik App Build and Release Automation

This repository contains Ansible and Docker scripts that automate the build and deploy process of a [Java Spring](https://github.com/amilovanovikj/videosonik-backend) and [React](https://github.com/amilovanovikj/videosonik-frontend) app, as well as Ansible scripts for automating the server configuration process. The original Java Spring backend app can be found at [vavramovski/videosonik-new](https://github.com/vavramovski/videosonik-new) and the original React frontend app can be found at [vavramovski/videosonik-frontend](https://github.com/vavramovski/videosonik-frontend). 

Huge thanks to [vavramovski](https://github.com/vavramovski) for letting me use these two apps for my project.

## What is implemented

- All package and deploy operations for the two apps, as well as server setup, are done using [Ansible playbooks](https://github.com/amilovanovikj/videosonik-devops/tree/master/playbooks).
- Docker images (that utilize multi-stage builds) are created for both the [backend](https://github.com/amilovanovikj/videosonik-backend/blob/master/Dockerfile) and [frontend](https://github.com/amilovanovikj/videosonik-frontend/blob/master/Dockerfile) apps, and are deployed using [Docker compose](https://github.com/amilovanovikj/videosonik-devops/tree/master/playbooks/templates/docker-compose.yml.j2) to multiple containers, load balanced by [Nginx](https://github.com/amilovanovikj/videosonik-devops/blob/master/playbooks/templates/nginx.conf.j2).
- Five VMs (created using this [Vagrantfile](https://github.com/amilovanovikj/videosonik-devops/blob/master/Vagrantfile)) are used for hosting the five different types of servers:
  - **Backend**: The Java Spring backend web app.
  - **Frontend**: The React frontend web app.
  - **Database**: The PostgreSQL database for the backend app.
  - **Build**: The server used to build the Docker images from the Git repositories of the apps and push them to a private Docker registry.
  - **Repository**: The machine hosting a private Docker registry for the Docker images, as well as a private Git server for this repository and the repositories for the FE and BE apps.
- The Ansible playbooks include:
  - [all-configure.yml](https://github.com/amilovanovikj/videosonik-devops/blob/master/playbooks/all-configure.yml): Install Docker and Python SDK for Docker on all hosts, enable insecure communication with private Docker registry.
  - [build-configure.yml](https://github.com/amilovanovikj/videosonik-devops/blob/master/playbooks/build-configure.yml): Install Git on the 'build' host and set up SSH communication to the private Git server.
  - [repository-configure.yml](https://github.com/amilovanovikj/videosonik-devops/blob/master/playbooks/repository-configure.yml): Run Git server and Docker registry on the 'registry' host as Docker containers, set up SSH communication between Ansible master and Git server.
  - [repository-restart.yml](https://github.com/amilovanovikj/videosonik-devops/blob/master/playbooks/repository-restart.yml): Restart the Docker containers of the private Git server and Docker registry.
  - [frontend-package.yml](https://github.com/amilovanovikj/videosonik-devops/blob/master/playbooks/frontend-package.yml): Pull frontend repository from private Git server, use Jinja2 templates to change default app configuration, build a Docker image from the frontend app and push it to the private Docker registry. Runs on 'build' host.
  - [backend-package.yml](https://github.com/amilovanovikj/videosonik-devops/blob/master/playbooks/backend-package.yml): Pull backend repository from private Git server, use Jinja2 templates to change default app configuration, build a Docker image from the backend app and push it to the private Docker registry. Runs on 'build' host.
  - [frontend-deploy.yml](https://github.com/amilovanovikj/videosonik-devops/blob/master/playbooks/frontend-deploy.yml): Deploy a Docker container of the frontend app on the 'frontend' host from the previously built Docker image, using Docker compose. Multiple instances of the app are started, alongside an Nginx load balancer to distribute the traffic between them.
  - [backend-deploy.yml](https://github.com/amilovanovikj/videosonik-devops/blob/master/playbooks/backend-deploy.yml): Deploy a Docker container of the backend app on the 'backend' host from the previously built Docker image, using Docker compose. Multiple instances of the app are started, alongside an Nginx load balancer to distribute the traffic between them.
  - [database-deploy.yml](https://github.com/amilovanovikj/videosonik-devops/blob/master/playbooks/database-deploy.yml): Run the PostgreSQL service on the 'database' host in a Docker container, create the necessary database and user for database access. These properties are hidden in an Ansible vault file.
- [Ansible roles](https://github.com/amilovanovikj/videosonik-devops/tree/master/roles) are used for installing Docker, Git and other auxilary packages on the servers.
- Various [Ansible templates](https://github.com/amilovanovikj/videosonik-devops/tree/master/playbooks/templates) are used to inject variables at playbook runtime.
- [Ansible vault](https://github.com/amilovanovikj/videosonik-devops/tree/master/playbooks/vault) is used to store secret variables.

## Replicate the setup

**Note**: In order to replicate the setup for running these scripts you would need Ansible 2.9 and Vagrant 2.2.14 installed on the host running these scripts (a Linux machine). Also, this setup requires a minimum of 8GB of RAM and 10GB free storage space.
***

First, create a common folder and clone all three repos in it:
```bash
mkdir videosonik && cd videosonik
git clone https://github.com/amilovanovikj/videosonik-backend.git
git clone https://github.com/amilovanovikj/videosonik-devops.git
git clone https://github.com/amilovanovikj/videosonik-frontend.git
```
Next, you will need to provision the 5 VMs using Vagrant:
```bash
cd videosonik-devops && vagrant up
```
After this command finishes you will have 5 CentOS 7 VMs up and running. You will first need to run the all-configure.yml playbook, followed by repository-configure.yml:
```bash
ansible-playbook playbooks/all-configure.yml
ansible-playbook playbooks/repository-configure.yml
```
This sets the stage for working with the local Git and Docker server, as well as deploying to the frontend, backend and database servers. To have these repositories pushed to the local Git server use the following commands:
```bash
cd ..
cd videosonik-backend && git remote set-url origin ssh://git@192.168.32.14:2222/git-server/repos/videosonik-backend.git
cd .. && git clone --bare videosonik-backend videosonik-backend.git
yes | scp -i videosonik-devops/.vagrant/machines/instance14/virtualbox/private_key -r videosonik-backend.git/ vagrant@192.168.32.14:/home/vagrant/git-server/repos
rm -rf videosonik-backend.git/

cd videosonik-devops && git remote set-url origin ssh://git@192.168.32.14:2222/git-server/repos/videosonik-devops.git
cd .. && git clone --bare videosonik-devops videosonik-devops.git
yes | scp -i videosonik-devops/.vagrant/machines/instance14/virtualbox/private_key -r videosonik-devops.git/ vagrant@192.168.32.14:/home/vagrant/git-server/repos
rm -rf videosonik-devops.git/

cd videosonik-frontend && git remote set-url origin ssh://git@192.168.32.14:2222/git-server/repos/videosonik-frontend.git
cd .. && git clone --bare videosonik-frontend videosonik-frontend.git
yes | scp -i videosonik-devops/.vagrant/machines/instance14/virtualbox/private_key -r videosonik-frontend.git/ vagrant@192.168.32.14:/home/vagrant/git-server/repos
rm -rf videosonik-frontend.git/
```
The last configuration step is to run the following Ansible command to configure the build server:
```bash
cd videosonik-devops
ansible-playbook playbooks/build-configure.yml
``` 
Now that all servers are set up and running, before building the app you must edit the [Ansible vault file](https://github.com/amilovanovikj/videosonik-devops/blob/master/playbooks/vault/secret-config.yml) (recreate with your own vault password) and specify the following variables:
```yaml
postgres_user:    *Username for the PostgreSQL connection*
postgres_pass:    *Password for the PostgreSQL connection*
admin_username:   *Username for the admin account on the Videosonik app*
admin_password:   *Password for the admin account on the Videosonik app*
admin_email:      *Email for the admin account on the Videosonik app*
jwt_secret:       *The secret key used by the JSON Web Token in Java*
```
Finally, you are ready to build and deploy the application:
```bash
ansible-playbook playbooks/frontend-package.yml
ansible-playbook --ask-vault-pass playbooks/backend-package.yml
ansible-playbook --ask-vault-pass playbooks/database-deploy.yml
ansible-playbook playbooks/backend-deploy.yml
ansible-playbook playbooks/frontend-deploy.yml
```
Note that if you make any changes to the frontend or backend apps, you will have to push them to the private Git repositories and run the package and deploy scripts once more.

Additionally, if you don't destroy the VMs after use, next time you want to start the application you can use the [repository restart playbook](https://github.com/amilovanovikj/videosonik-devops/blob/master/playbooks/repository-restart.yml) to start the Docker containers for the local Git server and Docker registry:
```bash
ansible-playbook playbooks/repository-restart.yml
```
If you ever restart the VMs, you'll be able to package and deploy the apps only after running this playbook. 

That's it! You now have the Videosonik web store application deployed on multiple Docker containers. :grin:
