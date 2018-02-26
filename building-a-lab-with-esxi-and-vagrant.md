# Building a lab with VMware ESXi and Vagrant

## Lab plan

| Hostname | Role | OS |
| :--- | :--- | :--- |
| DC01 | Domain controller | Server 2012 R2 |
| FS01 | File server | Server 2008 R2 |
| HA01 | Hardened server | Server 2016 Tech Eval |
| WS01 | Windows 10 workstation | W10 Enterprise |
| WS02 | Windows 7 workstation | W7 Enterprise |

VPN access will be set up to connect straight into the network, but no domain user provided.

## Prepping

### Hardware requirements

ESXi 6.5 compatible hardware \(can use 6.0 if incompatible\)

Enough RAM

A separate drive for installing ESXi \(rquires only 8 GB\) and one for the actual VMs.

A USB drive to install ESXi

A separate computer to do management from

### 

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

Using the following syntax download the required operating systems using Vagrant. Select `vmware_desktop` as provider when prompted. It is wise to choose boxes from the Vagrant cloud that doesn't have a configuration, those are usually indicated by `nocm`.

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

### Prepping ESXI USB drive

Download ESXI 6.5 image [https://my.vmware.com/en/group/vmware/evalcenter?p=free-esxi6](https://my.vmware.com/en/group/vmware/evalcenter?p=free-esxi6)

Use [Rufus ](http://rufus.akeo.ie/)to make a bootable USB key from the ESXI image.

Boot the lab machine from USB and install ESXi as per instruction.

### 

## Installing ESXI

Boot from the ESXI USB, install it to the small drive.

