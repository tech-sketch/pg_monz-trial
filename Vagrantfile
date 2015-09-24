# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure(2) do |config|
  config.vm.define :pgpool01 do |h1|
    h1.vm.provider "virtualbox" do |v|
      v.customize ["modifyvm", :id, "--memory", 512]
    end
    h1.vm.network :private_network, ip: "192.168.1.11"
    h1.vm.box = "ramonsnir/chef-centos-6.7"
    h1.vm.hostname = "pgpool01"
  end
  config.vm.define :pgpool02 do |h1|
    h1.vm.provider "virtualbox" do |v|
      v.customize ["modifyvm", :id, "--memory", 512]
    end
    h1.vm.network :private_network, ip: "192.168.1.12"
    h1.vm.box = "ramonsnir/chef-centos-6.7"
    h1.vm.hostname = "pgpool02"
  end
  config.vm.define :pgsql01 do |h1|
    h1.vm.provider "virtualbox" do |v|
      v.customize ["modifyvm", :id, "--memory", 512]
    end
    h1.vm.network :private_network, ip: "192.168.1.13"
    h1.vm.box = "ramonsnir/chef-centos-6.7"
    h1.vm.hostname = "pgsql01"
  end
  config.vm.define :pgsql02 do |h1|
    h1.vm.provider "virtualbox" do |v|
      v.customize ["modifyvm", :id, "--memory", 512]
    end
    h1.vm.network :private_network, ip: "192.168.1.14"
    h1.vm.box = "ramonsnir/chef-centos-6.7"
    h1.vm.hostname = "pgsql02"
  end
  config.vm.define :pgsql03 do |h1|
    h1.vm.provider "virtualbox" do |v|
      v.customize ["modifyvm", :id, "--memory", 512]
    end
    h1.vm.network :private_network, ip: "192.168.1.15"
    h1.vm.box = "ramonsnir/chef-centos-6.7"
    h1.vm.hostname = "pgsql03"
  end
  config.vm.define :zabbix do |h1|
    h1.vm.provider "virtualbox" do |v|
      v.customize ["modifyvm", :id, "--memory", 512]
    end
    h1.vm.network :private_network, ip: "192.168.1.20"
    h1.vm.box = "ramonsnir/chef-centos-6.7"
    h1.vm.hostname = "zabbix"
  end
end
