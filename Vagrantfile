# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"

  config.vm.provision "shell", inline: <<-SHELL
    apt-get  update
    apt-get install -y bind9 dnsutils
  SHELL

  config.vm.define "master" do |master|
    master.vm.network "private_network", ip: "192.168.57.103"
    master.vm.provision "shell", name: "master-dns" ,inline: <<-SHELL
      cp -v /vagrant/files/named /etc/default/
      cp -v /vagrant/files/named.conf.options /etc/bind
      cp -v /vagrant/files/master/named.conf.local /etc/bind
      cp -v /vagrant/files/sistema.test.dns /var/lib/bind
      cp -v /vagrant/files/sistema.test.rev /var/lib/bind
      chown root:bind /var/lib/bind/*
      systemctl restart named
    SHELL
  end

  config.vm.define "slave" do |slave|
    slave.vm.network "private_network", ip: "192.168.57.102"
    slave.vm.provision "shell", name: "salve-dns" ,inline: <<-SHELL
      cp -v /vagrant/files/named /etc/default/
      cp -v /vagrant/files/named.conf.options /etc/bind
      cp -v /vagrant/files/slave/named.conf.local /etc/bind
      systemctl restart named
    SHELL
  end
end
