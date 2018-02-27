# Building a lab with VMware ESXi and Vagrant

## Lab design

ESXi 6.5 installed on a physical box, with multiple VMs on an isolated virtual network. VPN access will be set up to connect straight into the network, but no domain user provided.

A virtual firewall - Sophos?

## Domain design

Active directory with OU, GPOs, hardening blabla

## Prepping

### Hardware requirements

* ESXi 6.5 compatible hardware \(can use 6.0 if incompatible\)
* Minimum 32 GB RAM
* A drive for ESXi - rquires only 8 GB
* A drive for the actual VMs - 500 GB+
* A USB drive to install ESXi with - minimum 1 GB
* A separate computer to do management from

### Software requirements

* ESXi 6.5
* Web browser to do management from
* Vagrant
* Vagrant VMware ESXi plugin
* VMware ovftool
* Windows Server 2008 R2
* Windows Server 2012 R2
* Windows Server 2016 Tech Evaluation
* Windows 7 Enterprise Edition
* Windows 10 Enterprise Edition

#### Future additions

* Sharepoint
* Office 2013 or 2016
* Outlook server

### Installing Vagrant

Install Vagrant and the vagrant VMware ESXi plugin.

[https://www.vagrantup.com/downloads.html](https://www.vagrantup.com/downloads.html)

[josenk/vagrant-vmware-esxi: A Vagrant plugin that adds a vmware ESXi provider support.](https://github.com/josenk/vagrant-vmware-esxi)

This plugin requires `ovftool`from VMware. Download from VMware website.

[https://www.vmware.com/support/developer/ovf/](https://www.vmware.com/support/developer/ovf/)

```
vagrant plugin install vagrant-vmware-esxi
vagrant version
```

### Downloading operating systems in Vagrant

Using the following syntax download the required operating systems using Vagrant. Select `vmware_desktop` as provider when prompted. It is wise to choose boxes from the Vagrant cloud that doesn't have any configuration management built in; those are usually indicated by `nocm`.

    vagrant box add opentable/win-2008r2-enterprise-amd64-nocm
    vagrant box add opentable/win-2012r2-standard-amd64-nocm
    vagrant box add StefanScherer/windows_2016
    vagrant box add opentable/win-7-enterprise-amd64-nocm
    vagrant box add StefanScherer/windows_10`

[Vagrant box opentable/win-2008r2-enterprise-amd64-nocm - Vagrant Cloud](https://app.vagrantup.com/opentable/boxes/win-2008r2-enterprise-amd64-nocm)

[Vagrant box opentable/win-2012-standard-amd64-nocm - Vagrant Cloud](https://app.vagrantup.com/opentable/boxes/win-2012-standard-amd64-nocm)

[Vagrant box StefanScherer/windows\_2016 - Vagrant Cloud](https://app.vagrantup.com/StefanScherer/boxes/windows_2016)

[Vagrant box opentable/win-7-enterprise-amd64-nocm - Vagrant Cloud](https://app.vagrantup.com/opentable/boxes/win-7-enterprise-amd64-nocm)

[Vagrant box StefanScherer/windows\_10 - Vagrant Cloud](https://app.vagrantup.com/StefanScherer/boxes/windows_10)

## Installing ESXI

Download ESXI 6.5 image [https://my.vmware.com/en/group/vmware/evalcenter?p=free-esxi6](https://my.vmware.com/en/group/vmware/evalcenter?p=free-esxi6)

Use [Rufus ](http://rufus.akeo.ie/)to make a bootable USB key from the ESXI image.

Boot the lab machine from USB and install ESXi on the small drive as per instruction.

After installation, reboot the server. ESXi should now provide a DHCP-leased IP-address you can access from a web panel.

#### Troubleshooting

Troubleshooting write speeds with SSD: https://communities.vmware.com/thread/554004

ESXi 6.5 includes a new native driver \(vmw\_ahci\) for SATA AHCI controllers, but that introduces performance problems with a lot of controllers and/or disks.

Try to disable the native driver and revert to the older sata-ahci driver by running

`esxcli system moduleset--enabled=false--module=vmw_ahci`

### Enabling ESXi shell and SSH

The Vagrant ESXi plugin requires SSH to be anabled.

1. At the direct console of the ESXi host, press F2 and provide credentials when prompted.
2. Scroll to Troubleshooting Options and press Enter.
3. Choose Enable ESXi shell and Enable SSH and press Enter once on each of them
4. Press Esc until you return to the main direct console screen.

### Seting static IP for the ESXi host

1. Press F2 on the ESXi console, provide credentials when prompted

2. Configure management network -&gt; IPV4 Configuration

3. Press space on `Set static ipv4 address`

4. Press Esc until you return to the main direct console screen.

### Adding a datastore to ESXi

Add the big drive, where the virtual machines will be stored as a datastore in ESXi.

1. In the ESXi web client press `Storage`in the left side pane.
2. Just follow the instructions after selecting `New datastore`from the menu, 
3. Add a drive, give it a name like `VMs` and use the whole drive as one partition.

### Adding a network configuration to ESXi

1. Select Networking on the left side pane
2. Click port group, ADD port group. 
3. Give it the name `Lab Network`, asign it to `VLAN 0`, assign it to `vSwitch0`which is the default virtual switch.



