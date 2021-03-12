# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :inetRouter => {
    :box_name => "centos/7",
    :net => [
               {ip: '10.10.10.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
            ]
  },
  :inetRouter2 => {
    :box_name => "centos/7",
    :net => [
               {ip: '10.11.11.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net1"},
               {ip: '192.168.2.2', adapter: 3, netmask: "255.255.255.0"},            
            ]
  },
  :centralRouter => {
    :box_name => "centos/7",
    :net => [
               {ip: '10.10.10.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
               {ip: '10.11.11.2', adapter: 3, netmask: "255.255.255.252", virtualbox__intnet: "router-net1"},
               {ip: '192.168.0.20', adapter: 4, netmask: "255.255.255.0", virtualbox__intnet: "hw-net"},
            ]
  },
  :centralServer => {
    :box_name => "centos/7",
    :net => [
               {ip: '192.168.0.10', adapter: 2, netmask: "255.255.255.0", virtualbox__intnet: "hw-net"},
            ]
  },
}

Vagrant.configure("2") do |config|
  config.vbguest.auto_update = false
  config.vm.provider "virtualbox" do |vb|
    vb.memory = 256
    vb.cpus = 1
  end
  
  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|
        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s
        
        boxconfig[:net].each do |ipconf|
            box.vm.network "private_network", ipconf
        end
        
        box.vm.provision "shell", inline: <<-SHELL
            mkdir -p ~root/.ssh
            cp ~vagrant/.ssh/auth* ~root/.ssh
        SHELL
        
       
        end
      end
  end
end