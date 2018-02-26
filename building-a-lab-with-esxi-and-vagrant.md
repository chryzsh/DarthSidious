# Building a lab with VMware ESXi and Vagrant

## Lab plan

VPN access will be set up to connect straight into the network, but no domain user provided.

## Domain plan

Domain name etc

## Prepping

### Hardware requirements

* ESXi 6.5 compatible hardware \(can use 6.0 if incompatible\)
* Enough RAM
* A separate drive for installing ESXi \(rquires only 8 GB\) and one for the actual VMs \(500 GB+\).
* A USB drive to install ESXi with
* A separate computer to do management from, in ESXi 6.5

### Installing Vagrant

Install Vagrant and the vagrant VMware ESXi plugin.

This plugin requires ovftool from VMware. Download from VMware website.

[https://www.vmware.com/support/developer/ovf/](https://www.vmware.com/support/developer/ovf/)

[https://www.vagrantup.com/downloads.html](https://www.vagrantup.com/downloads.html)

[josenk/vagrant-vmware-esxi: A Vagrant plugin that adds a vmware ESXi provider support.](https://github.com/josenk/vagrant-vmware-esxi)

```
vagrant plugin install vagrant-vmware-esxi
vagrant version
```

'How to enable ssh access on esxi'

#### Download operating systems in Vagrant

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

