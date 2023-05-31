# Learning Ansible

Learning to use Ansible with [Vagrant](https://www.vagrantup.com/)

## Vagrant

Vagrant enables the creation of predictable development environments. That is, not only you will always have the same environment, but you can share with colleagues, and save a lot of time trying why something working in one team's computer, but not on the other. Vagrant can use several virtualization mechanisms, called _providers_, and the most common is Virtualbox.

To [install](https://developer.hashicorp.com/vagrant/downloads) Vagrant on Windows, just download and run the installer. To install on Linux system, you will add the repository

> Note: on Windows system a reboot is required after installing Vagrant.

Once installed, initialize the Vagrant script by running

```
vagrant init
```

This will create a boot script called [`Vagrantfile`](https://developer.hashicorp.com/vagrant/docs/vagrantfile) with can further configured for the project. The _init_ accepts as parameter the [_box_](https://app.vagrantup.com/boxes/search), that is, the operating system and release to use. For example, to create a VM with [Rocky Linux 9](https://rockylinux.org/):

```
vagrant init rockylinux/9
```

or to create Ubuntu VM with Ubuntu 22.04 LTS 64 bits:

```
vagrant init ubuntu/jammy64
```
## Configuring the Virtual Machine


### Multiple Machines

#### SSH Config

Althought not secure for production, for development purpose we can use Vagrant ssh keys. Also for convinience, we create a secondary interface and assign a private IP.

This intra-Vagrant setup can be [confusing](https://stackoverflow.com/questions/27005400/vagrant-multiple-machines-inter-ssh-key-authentication), but it seems the important thing is to use [Vagrant ssh keys](https://github.com/hashicorp/vagrant/tree/master/keys) to [configure ssh](https://www.vagrantup.com/docs/vagrantfile/ssh_settings.html) in the Vagrant file.

```
  config.ssh.insert_key = false
  config.vm.provision "file", source: "keys/config",  destination: "~/.ssh/config"
  config.vm.provision "file", source: "keys/vagrant", destination: "~/.ssh/id_rsa"
  config.vm.provision "file", source: "keys/vagrant.pub", destination: "~/.ssh/id_rsa.pub"
  config.vm.provision "shell", inline: "chmod 600 /home/vagrant/.ssh/id_rsa"
  config.vm.provision "shell", inline: "chmod 600 /home/vagrant/.ssh/id_rsa.pub"
  config.vm.provision "shell", inline: "chmod 600 /home/vagrant/.ssh/config"

  config.vm.define "flik" do |flik|
  (...)
    flik.vm.network "private_network", ip: "192.168.56.11"
  end

  config.vm.define "hopper" do |hopper|
  (...)
    hopper.vm.network "private_network", ip: "192.168.56.12"
  end

  config.vm.define "atta", primary: true do |atta|
  (...)
    atta.vm.network "private_network", ip: "192.168.56.10"
  end
```

## What is Ansible?

Ansible is an automation and orchestration tool.

> The name "Ansible" is a refence to a communication device created by Ursula K. Le Guin in her book "Rocannon’s World" (Ace Books, 1966). In the book, the device Ansible can send communication faster than the speed of light. Ansible cofounder Michael DeHaan took the name Ansible from Card’s book Ender’s Game (Tor, 1985). In that book, the ansible was used to control many remote ships at once, over vast distances. Think of it as a metaphor for controlling remote servers.[^ansuprun]

Ansible is a described as _configuration manager_ like Puppet, Salt and Chef. With a configuration manager, you describe the _state_ of the server, not the steps to reach that state. Ansible can also be use for _orchestration_, that is, when you need to specify a specific order: shutdown the database server before the webserver, for example.

## Installing on Ubuntu

Installing [Ansible on Ubuntu](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-ubuntu) system

Creating the PPA

```
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

## Inventory File

The [inventory](https://docs.ansible.com/ansible/latest/inventory_guide/index.html) is the most basic building block in Ansible. Nothing happens without as inventory. The default localtion of the inventory is the `hosts` file in the Ansible installation directory. Using the SSH information from Vagrantfile above we create an inventory

```
vagrant@atta:~$ cat /etc/ansible/hosts
atta      ansible_ssh_host=192.168.56.10   ansible_connection=local
flik      ansible_ssh_host=192.168.56.11
hopper    ansible_ssh_host=192.168.56.12

[nodes]
hopper
flik

vagrant@atta:~$
```

With the inventory in place, we can test the Ansible installation, by trying to access (module `ping`) the `[nodes]` group in the inventory: `hopper` and `flik`. If successful, Ansible will respond with `pong`.

```
vagrant@atta:~$ ansible nodes -m ping
flik | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
hopper | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
vagrant@atta:~$
```

Next we can start using Ansible. But before, a quick introduction to the work horse of Ansible world: modules. There are modules for pretty much every task.

## Modules

Modules provide the tools for working with Ansible. There are [modules for almost everything](https://docs.ansible.com/ansible/latest/collections/index_module.html), but we will be using most the [builtin modules](https://docs.ansible.com/ansible/latest/collections/index_module.html#ansible-builtin):

`apt`
: install packages using the `apt` command.

`copy`
: copy files to the remote hosts.

`service`
: to manage services, that is, start, stop, reload, etc., services like Apache or MySQL.

## Using Ansible

Ansible can be used in two ways: [_ad-hoc_](https://docs.ansible.com/ansible/latest/command_guide/intro_adhoc.html#introduction-to-ad-hoc-commands) and [_playbooks_](https://docs.ansible.com/ansible/latest/playbook_guide/index.html#). In the ad-hoc mode, Ansible is like a cluster shell, where it executes the command on all noders defined in the command line, which here, will be the `nodes` group define above

For example to install (module `apt`) `vim` on tbe nodes

```
vagrant@atta:~$ ansible nodes --become -m apt -a "name=vim state=present"
flik | SUCCESS => {
(...)
    "changed": false
}
hopper | SUCCESS => {
(...)
    "changed": false
}
vagrant@atta:~$
```

Where `--become` means to [become another user](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_privilege_escalation.html#), which the default is `root`, and the default privilege escalation is `sudo`. So the command above is equivalent of running `apt install vim` on each node.

Another way, and probably the way that Ansible is used most, is via a _playbook_. A playbook is a script for doing something, installing a package, starting or stopping a service, etc.

To show how to use a playbook, we will create one to install the `vim` editor, the same think we did with the _ad-hoc_ command before. The _play_ looks like this:

```yaml
---
- name: Install vim editor
  hosts: nodes
  tasks:
    - name: Installing vim editor
      become: yes
      apt:
        name: vim
        state: present
```

The structure of a _play_ always is same

1. A `name` for the play. Not obrigatory, but is always a good idea.
1. Which `hosts` are the target of the playbook.
1. The `tasks` to be executed on the hosts
   1. It's a good idea to `name` the task too.
   1. the `become` is for _privilege escalation_, that is, to run the task with `sudo`.
   1. Use the module `apt` to install package
      1. The `name` of the package.
      1. The `state` of the package, in general `present` to install or `absent` to uninstall.

Running this _play_ will produce the following output

```
vagrant@atta:~$ ansible-playbook /vagrant/plays/vim_install.yml

PLAY [Install vim editor] ********************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************
ok: [flik]
ok: [hopper]

TASK [Installing vim editor] *****************************************************************************************************
ok: [hopper]
ok: [flik]

PLAY RECAP ***********************************************************************************************************************
flik                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
hopper                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

vagrant@atta:~$
```

We can identfy from the _play_

1. The _play_ level name that we gave
1. The `Gathering Facts` is a Ansible task that collects information about the host: name, IP, OS, version, and much more.
   1. We can see the nodes where the task is running
1. The `name` of the task, and again, the hosts where the task is running.
1. Finally a `play recap`, where we see how many tasks succeed, how many failed, etc.
