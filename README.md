# A working R + MySQL + Shiny setup for development/evaluation and testing purposes

With these files you can setup and provision a locally running in VMs,
MySQL server 5.6 in a separate VM, and R+Shiny in another VM

## Specs

The setup contains 2 VMs:

* MySQL node with 2GB of RAM and IP address 192.168.222.10  
* R + Shiny + rmarkdown in another node with IP address 192.168.222.11

It's been tested with Virtualbox 4.3.20, running in Windows XP, as well as Mac OS

Prerequisites :
- VirtualBox 4.3.20 
- Cygwin (It may work without Cygwin, I just never tested it without in Windows)
- Vagrant 
- vagrant-hostmanager (Install as follows, if you don't have one: "vagrant plugin install vagrant-hostmanager")
``

Clone this repository.

$ git clone https://github.com/vigorIv2/shiny_R_mysql#
``

Provision the VMs

```bash
$ cd shiny_R_mysql
$ vagrant up
```

Sometimes the provisioning throws error messages regarding the Guest Additions to the VM,
those are not fatal, ignore those and follow instructions below.

Just run:

vagrant provision mysql_shiny
then
vagrant provision web_shiny

After all processes finished, navigate to http://localhost:3838 in your browser

to access your VMs shell via SSH:

vagrant ssh mysql_shiny
or
vargant ssh web_shiny



