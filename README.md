# DNS sistema.test

## Data of the problem
###  Network
Every machine there would be in the network **192.168.57.0/24**.

### Machines
We will manage inside the vagrantfile the two Debian Linux.

**Debian2** _(Tierra)_ will be the master.

**Debian1** _(Venus)_ will be the slave.

| Machines | FQDN | IP |
| :---: | :---: | :---: |
| Imaginary Linux | mercurio.sistema.test  | 192.168.57.101 |
| Debian 1  | venus.sistema.test | 192.168.57.102 |
| Debian 2  | tierra.sistema.test | 192.168.57.103 |
| Imaginary Windows  | marte.sistema.test | 192.168.57.104 |

## VagrantFile
To start we create a provision for the two machines. In them, we install [Bind9](https://www.isc.org/bind/).

Later we create the master and slave machines with all the information that we knew.

To start with DNS I create a provision with basic files. These files are necessary that both have them.

```ruby
    master.vm.provision "shell", name: "master-dns" ,inline: <<-SHELL
      cp -v /vagrant/named /etc/default/
      cp -v /vagrant/named.conf.options /etc/bind
      systemctl restart named
    SHELL

    slave.vm.provision "shell", name: "salve-dns" ,inline: <<-SHELL
      cp -v /vagrant/named /etc/default/
      cp -v /vagrant/named.conf.options /etc/bind
      systemctl restart named
    SHELL
```

## DNS Config Files
### named

To find this file it´s in `/etc/default`, so I copy it on my vagrant.

```bash
  cp /etc/default/named /vagrant/files
```

With this file, we establish the IPv4 protocol so that it only listens there.

```bash
  # startup options for the server
  OPTIONS="-u bind -4"
```
Inside the VagratFile I change the path of these files in both machines.

```ruby
    master.vm.provision "shell", name: "master-dns" ,inline: <<-SHELL
        cp -v /vagrant/files/named /etc/default/
        cp -v /vagrant/named.conf.options /etc/bind
        systemctl restart named
      SHELL

    slave.vm.provision "shell", name: "salve-dns" ,inline: <<-SHELL
          cp -v /vagrant/files/named /etc/default/
          cp -v /vagrant/named.conf.options /etc/bind
          systemctl restart named
      SHELL
```

### named.conf
This file is in `/etc/bind`. Not necessary in our case.

This file is simply used to group the configuration files that we will use. These 3 includes refer to the 3 different files where we will have to perform the real configuration, located in the same directory.

```bash
  include "/etc/bind/named.conf.options";
  include "/etc/bind/named.conf.local";
  include "/etc/bind/named.conf.default-zones";
```

### named.conf.options
This file is in `/etc/bind` as we can see before, and this one is important, so I copy on my vagrant, inside the files folder.

```bash
  cp /etc/bind/named.conf.options /vagrant/files
```

I change the path in the VagrantFile to be correct.
```ruby
  master.vm.provision "shell", name: "master-dns" ,inline: <<-SHELL
      cp -v /vagrant/files/named /etc/default/
      cp -v /vagrant/files/named.conf.options /etc/bind
      systemctl restart named
    SHELL

  slave.vm.provision "shell", name: "salve-dns" ,inline: <<-SHELL
      cp -v /vagrant/files/named /etc/default/
      cp -v /vagrant/files/named.conf.options /etc/bind
      systemctl restart named
    SHELL
```

This file is used to configure the DNS server. 
Is required for this project set `dnssec-validation` to `yes`.
(We will enable or disable dnssec, as the auto option may have problems with the router.)

```bash
  options{
    dnssec-enable yes;
      dnssec-validation yes; 
  };  
```
