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

To start with DNS I create a provision with basic files. These files are necessary that both have.

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