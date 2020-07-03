# Ansible configurations

This repository will contain the different Ansible configurations needed for the servers.

This will help easily create new servers, update different ones, gather information or create backups.

To test out locally a docker-container has been added. If you want to test things make sure the container is running by running :

`docker-compose up`

The container is a debian buster with ssh installed and a `exploit` user.

To connect through SSH to the docker container run :

`ssh -i ./id_rsa exploit@172.40.1.1`

To check if ansible can connect run :

`ansible dev -m ping -vvv -i hosts.ini`

The default playbook is set to install docker and docker-compose to the dev container. To run it run :

`ansible-playbook playbook.yml -i hosts.ini`

This playbook makes use of multiple roles :
- oh-my-zsh: This will install oh-my-zsh and configure the user settings
- ssh-banner: customize the banner on ssh login and add a variable
- docker: This will install and run Docker, it also installs docker-compose
- docker-test: This will run a docker hello world and show the output to make sure Docker runs
- traefik: This will generate the docker-compose and the configuration files for traefik. It will also create the external `web` network and run the container
- maria: This will generate the docker-compose for maria and setup the ENV for the develop, staging and production environments
- portainer: This will generate the docker-compose for portainer, connect to traefik and setup the url
- reboot: make the server reboot after finishing to apply certain changes (like switching nftables to iptables)

You can test just some of them by using the `-t` option:

`ansible-playbook playbook.yml -i hosts.ini -t docker,oh-my-zsh`

## Variables

The variables are set in vars/vars.yml some of these are secret. In order for this to work you need to create a secret vars/audit.yml file with :

`ansible-vault create vars/vault.yml`

This will ask you to set a password. Then it will ask you to fill in the vault.yml file, it needs the following variables :

```yaml
---
vault_traefik:
  basic_auth_user: User:$usr$pass$word
  url: traefik.localhost
  letsencrypt:
    email: contact@me.com

vault_portainer:
  url: portainer.localhost
 
vault_maria:
  production:
    user_password: password
    root_password: password
  staging:
    user_password: password
    root_password: password
  develop:
    user_password: password
    root_password: password 
```

To run the playbook you now need to add `--ask-vault-pass`:

`ansible-playbook playbook.yml -i hosts.ini --ask-vault-pass`

To not have to type in the password in dev mode there is a group_vars/dev.yml file that contains some default values for the variables inside the vault but without being encrypted

## FAQ

**The containers can not connect to the internet**
> In debian 10 we need to set nftables to iptables, this is done by the role but the server needs to be rebooted afterwards

**Portainer keeps asking for the basic auth when creating an admin user**
> To set the admin user you need to comment out the auth middleware label inside the docker compose

**Maria will not let me connect to it using the password I have set in the vars/vault**
> since maria 14.3 the root user does not use the password, you need to login with the root user account and without a password
> `sudo mysql -uroot` from inside the container will get you logged in

**After cloning the repo GitHub complains about a published private SSH kay**
> The public key is juste here to connect to the local docker container it is not used on any real servers

**I can not see my ssh login banner**
> In order to have it appear you need to either run `sudo systemctl restart ssh.service` on your server or reboot it

## TODO

- multiple OSes not just debian 10
- Make sure maria waits before trying to trigger to user rights
- add more default roles !!
- Maybe once more OSes are defined add a local host to be able to generate local dev envs rapidly and not just limit this to the testing of servers
- Anything that come into your mind !
 

