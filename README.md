# rpi-provisioning

Provisioning a Raspberry Pi 3B with ansible.

The Raspberry Pi is used to record audio.

## Requirements

Requires a default installation of [ubuntu server on a micro SD card](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#1-overview),
and the latest [ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
version on a machine which will be used to run the provisioning.

## What it does

- Disables/removed unwanted services and packages.
- Removes the default user and creates a new user (using your username).
- Configures SSH and firewall to only allow access from local network.
- Configures WiFi connection.

## Getting started

First, configure your `~/.ssh/config` file in order to set the `rpi` host :

```
# ~/.ssh/config
Host rpi
    HostName 192.168.1.x # use `$ nmap -sn 192.168.1.0/24` to figure out
                         # the IP address
    User <your_username>
    ForwardAgent yes
```

Then, copy and configure the [`vars/rpi.yml.dist`](/vars/rpi.yml.dist) file to
`vars/rpi.yml`.

## Provisioning

When provisioning for the first time, you'll have to use the default `ubuntu`
user as it's the only one on the rpi. So make sure you've already logged into
the rpi with this user first. Then, when running the provisioning,
this default user will be removed and the new one will be created. Once it is,
you should use it.

Start by sending your public ssh key for the default user :

```bash
$ ssh-copy-id rpi
```

Then provision the rpi to create your user :

```bash
$ ansible-playbook -i hosts -l rpi users.yml -t create_matching_user --ask-become-pass
```

Now edit your `~/.ssh/config` file to replace the default user with your
username, and check that you can ssh into the rpi :

```bash
$ ssh rpi
```

Set your new password when prompted (the default one is your username), and
finally copy your public ssh key to be able to ssh into the rpi without having
to enter a password :

```bash
$ ssh-copy-id rpi
```

Then remove the default user from the rpi :

```bash
$ ansible-playbook -i hosts -l rpi users.yml -t remove_default_user --ask-become-pass
```

Now, you can continue the provisioning :

```bash
# will provision the rpi with the remaining tasks (using your user)
$ ansible-playbook -i hosts -l rpi system.yml
```
