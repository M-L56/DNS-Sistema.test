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
In this file, parameters are specified that affect the general behavior of the server. It is essential for defining the overall operation and policies of the DNS server.

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

Is required for this project set `dnssec-validation` to `yes`.
(We will enable or disable dnssec, as the auto option may have problems with the router.)

```bash
  options{
    dnssec-enable yes;
      dnssec-validation yes; 
  };  
```

Servers will allow recursive queries only to computers on the network `127.0.0.0/8` and on the network `192.168.57.0/24`.

To set the IPs we use ACL (Access Control List) outside the options.

```bash
  acl "trusted"{
    127.0.0.0/8;
    192.168.57.0/24;
  };
```

To active the recursivity we add two lines inside options.

```bash
  options{
    recursion yes;
    allow-recursion { trusted; };
  };  
```

### named.conf.local
This file is in `/etc/bind` as we can see before, so I copy on my vagrant, inside the files folder.

```bash
  cp /etc/bind/named.conf.local /vagrant/files
```

In this file, the zones for which the server is authoritative are specified, whether they are **direct zones** (name-to-IP resolution) or **reverse zones** (IP-to-name resolution). It is key to managing the specific DNS zones and records on the server.

Inside **files** folder, we will create two folders, one for the configuration of the master and other for the configuration of the slave.

We need to add the path inside the VagrantFile

```ruby
  master.vm.provision "shell", name: "master-dns" ,inline: <<-SHELL
        cp -v /vagrant/files/named /etc/default/
        cp -v /vagrant/files/named.conf.options /etc/bind
        cp -v /vagrant/files/master/named.conf.local /etc/bind
        systemctl restart named
      SHELL

  slave.vm.provision "shell", name: "salve-dns" ,inline: <<-SHELL
      cp -v /vagrant/files/named /etc/default/
      cp -v /vagrant/files/named.conf.options /etc/bind
      cp -v /vagrant/files/slave/named.conf.local /etc/bind
      systemctl restart named
    SHELL
```

#### MASTER named.conf.local

The master server will be **tierra.sistema.test** and will have authority over the _direct_ and _reverse_ zone.

***Direct Zone***

In `/var/lib/bind/system.test.dns`, there will be located the resources records for the **tierra.sistema.test** zone.

```bash
  zone "tierra.sistema.test" {
    type master;
    file "/var/lib/bind/sistema.test.dns";
  };
```
***Reverse Zone***

This `/var/lib/bind/sistema.test.rev`, indicates the location of the file that contains the resource records for the reverse zone.

```bash
  zone "57.168.192.in-addr.arpa"{
      type master;
      file "/var/lib/bind/sistema.test.rev";
  };
```

#### SLAVE named.conf.local

The slave server will be **venus.sistema.test** and will have **tierra.sistema.test** as a master.

```bash
  zone "venus.sistema.test" {
      type slave;
      masters {192.168.57.103; };
  };
```