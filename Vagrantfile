# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  # Synched folders.
  config.vm.synced_folder ".", "/vagrant", type: "rsync",
  rsync__exclude: ".git/"

  config.vm.box = "ubuntu/jammy64"

  config.ssh.insert_key = false
  config.vm.provision "file", source: "keys/config",  destination: "~/.ssh/config"
  config.vm.provision "file", source: "keys/vagrant", destination: "~/.ssh/id_rsa"
  config.vm.provision "file", source: "keys/vagrant.pub", destination: "~/.ssh/id_rsa.pub"
  config.vm.provision "shell", inline: "chmod 600 /home/vagrant/.ssh/id_rsa"
  config.vm.provision "shell", inline: "chmod 600 /home/vagrant/.ssh/id_rsa.pub"
  config.vm.provision "shell", inline: "chmod 600 /home/vagrant/.ssh/config"

  #
  # Using the IP range 192.168.56.0/21 as recommended by Virtualbox
  # https://www.virtualbox.org/manual/ch06.html#network_hostonly
  #
  config.vm.define "flik" do |flik|
    flik.vm.hostname = "flik"
    flik.vm.provider :virtualbox do |vb|
        vb.memory = 2048
        vb.cpus = 2
    end
    flik.vm.network "private_network", ip: "192.168.56.11"
    flik.vm.boot_timeout = 600
  end

  config.vm.define "hopper" do |hopper|
    hopper.vm.hostname = "hopper"
    hopper.vm.provider :virtualbox do |vb|
        vb.memory = 2048
        vb.cpus = 2
    end
    hopper.vm.network "private_network", ip: "192.168.56.12"
    hopper.vm.network "forwarded_port", guest: 8080, host: 80
    hopper.vm.boot_timeout = 600
  end

  config.vm.define "atta", primary: true do |atta|
    atta.vm.hostname = "atta"
    atta.vm.provider :virtualbox do |vb|
        vb.memory = 2048
        vb.cpus = 2
    end
    atta.vm.network "private_network", ip: "192.168.56.10"
    atta.vm.boot_timeout = 600
  end

end
