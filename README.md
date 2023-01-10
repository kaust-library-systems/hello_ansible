# Learning Ansible

Learning to use Ansible

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

Creating a simple inventory file to test the installation
