# jellyfin_ansible_lxd

## Launch an LXD container

```bash
lxc launch images:ubuntu/22.04 jellyfin
```

Clone this repo

```bash
https://github.com/jonnypeace/jellyfin_ansible_lxd.git
```

move into the git host directory

```bash
cd jellyfin_ansible_lxd/host
```

Create an ansible vault password

```bash
ansible-vault create passwd.yml
```

and paste something like this but with your host password for the sudo user

```bash
---
my_cluster_sudo_pass: My_Password
```

Remember to change the password. The "my_cluster_sudo_pass" variable will be used in further playbooks, so don't change this unless you change the playbook.

Edit the inventory file, notable sections...
* ansible_user - this is the sudo user for the host ansible will be connecting to
* jellyfin_host section - update the ip address of the LXD host and user

Edit the initialise_jellyfin_host.yaml - this also adds intel quicksync GPU acceleration. if not required, comment or delete the lines associated with this.

Notable areas here include the variables..
* jelly_cont - this is the name of the container you created at the beginning
* nfs_mnt - Creates a directory for the host to mount an nfs share
* nfs_share - This is the ip address and path that your nfs share comes from

If you just need to share a drive that's already connected via USB etc, then you could remove some of this, but i've included it as it's something i use.

Now you can run this command which will hopefully get the host and containe ready...

```bash
ansible-playbook -i inventory initialise_jellyfin_host.yaml --ask-vault-pass
```

The update_reboot.yaml will update your host... these become more useful perhaps the more servers / containers you have running.

If all is successful, lets move to the container directory in this repo.

## Container playbook

Ok for this you don't need ssh passwords, as you connect via the lxd host. This might be a good timeto explore...

```bash
lxc config set core.trust_password mypassword
lxc remote add lxd_host_ip mylxd
lxc remote switch mylxd

So, lxd_host_ip will be the server ip which will be running your lxd containers
mylxd is just the name of the host, this can be anything, but you'll use this to switch.

lxc remote list

This will show you available lxd machines to connect to, including your local desktop/laptop.
```

The first command is run on the LXD host and you set a password for your laptop/desktop to connect.
The second command is run on your laptop/desktop and you will be prompted for that password.

Edit the hosts.yaml - Since you don't need ssh for this it makes it a little more straight forward.

The main section for your lxd server host is under the children, you can leave the rest alone. I've written a section for mylxd, as per example above, and under mylxd the container we have runnung if you've followed along with the name i gave the container, is jellyfin (under host). 

In the jellyfin.yaml, there is a hosts section for the container name, hence jellyfin.

This part is fairly straight forward, it sets up jellyfin and intel quicksync should be ready

```bash
ansible-playbook -i hosts.yaml jellyfin_node.yaml 
```
This should be the container ready. Check the ip of the container and append :8096 to the ip.

so...

```bash
lxc list
```
will provide the ip address, go to the web browser and enter the ip, i.e. 192.168.1.44:8096

You should be greeted with the setup guide.

the nodes_update.yaml will update your jellyfin container for you and reboot the container if necessary.
