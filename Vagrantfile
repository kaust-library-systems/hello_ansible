# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/jammy64"

  config.ssh.insert_key = false
  
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
    atta.vm.network "private_network", ip: "192.168.56.10"
    atta.vm.provider :virtualbox do |vb|
        vb.memory = 2048
        vb.cpus = 2
    end
    atta.vm.boot_timeout = 600
  end

end
