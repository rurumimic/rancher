# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "generic/centos7"
  config.vm.box_check_update = false

  config.vm.define vm_name = "rancher" do |config|
    config.vm.hostname = "rancher"
    config.vm.network "forwarded_port", guest: 8080, host: 8080, host_ip: "127.0.0.1"
    config.vm.network :private_network, ip: "172.17.177.11"

    config.vm.provider "virtualbox" do |vb|
      vb.cpus = 4
      vb.memory = 4096
    end
  end

  config.vm.define vm_name = "db" do |config|
    config.vm.hostname = "db"
    config.vm.network "forwarded_port", guest: 3306, host: 3306, host_ip: "127.0.0.1"
    config.vm.network :private_network, ip: "172.17.177.12"

    config.vm.provider "virtualbox" do |vb|
      vb.cpus = 1
      vb.memory = 1024
    end
  end

  config.vm.provision "shell", inline: <<-SHELL
    # Trust your enterprise certificates
    if [ -d "/vagrant/ca-trust" ]; then
      cp /vagrant/ca-trust/* /etc/pki/ca-trust/source/anchors
      update-ca-trust
    fi
  SHELL

end
