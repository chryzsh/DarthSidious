# Building a lab with VMware ESXi and Vagrant

## Lab design

ESXi 6.5 installed on a physical box, with multiple VMs on an isolated virtual network. VPN access will be set up to connect straight into the network, but no domain user provided.

A virtual firewall - Sophos?

Internal virtual network

## Domain design

Active directory with OU, GPOs, hardening blabla

## Server plan

| Hostname | Role | OS |
| :--- | :--- | :--- |
| DC01 | Domain controller | Server 2012 R2 |
| FS01 | File server | Server 2008 R2 |
| WEB01 | Web server | Server 2016 Tech Eval |
| WS01 | Workstation | W10 Enterprise |
| WS02 | Workstation | W7 Enterprise |
| CENT01 | Annoying Linux box | CentOS 7.4 |

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
* VMware Workstation 1x.x
* Web browser to do management from
* Vagrant
* Vagrant VMware ESXi plugin
* Vagrant Reload Provisioner
* Vagrant WinRM Syncedfolders
* VMware ovftool
* Windows Server 2008 R2
* Windows Server 2012 R2
* Windows Server 2016 Tech Evaluation
* Windows 7 Enterprise Edition
* Windows 10 Enterprise Edition
* CentOS 7.4

#### Future additions

* Sharepoint
* Office 2013 or 2016
* Microsoft Exchange

### Installing Vagrant

Install Vagrant and the vagrant VMware ESXi plugin.

[https://www.vagrantup.com/downloads.html](https://www.vagrantup.com/downloads.html)

[josenk/vagrant-vmware-esxi: A Vagrant plugin that adds a vmware ESXi provider support.](https://github.com/josenk/vagrant-vmware-esxi)

[aidanns](https://github.com/aidanns)/[vagrant-reload](https://github.com/aidanns/vagrant-reload)

[Cimpress-MCP](https://github.com/Cimpress-MCP)/[vagrant-winrm-syncedfolders](https://github.com/Cimpress-MCP/vagrant-winrm-syncedfolders)

This plugin requires `ovftool`from VMware. Download from VMware website.

[https://www.vmware.com/support/developer/ovf/](https://www.vmware.com/support/developer/ovf/)

```
vagrant plugin install vagrant-vmware-esxi
vagrant plugin install vagrant-winrm-syncedfolders
vagrant plugin install vagrant-reload
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

Troubleshooting write speeds with SSD: [https://communities.vmware.com/thread/554004](https://communities.vmware.com/thread/554004)

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
2. Click Add standard switch, name it vSwitch1
3. Don't assign 
4. Click port group, ADD port group. 
5. Give it the name `Lab Network`, asign it to `VLAN 0`, assign it to `vSwitch0`which is the default virtual switch.

### Preparing base images for every OS

Deploying to Vagrant and applying things like powershell config during deployment will be a lot easier if the VMs are prepped. This process must be repeated for every VM, which is a drag, but it only has to be done once.

* Make a new directory and call it PrepSever2016. Copy the entire directory of the VM `.vagrant.d/boxes/repoNameOfVM` to a new directory.
* Before booting the VM in Workstation, set up a file share, because transfering files to the box is necessary.
* If not possible, set up a network adapter so you can host the files on a local web server or on Github so you can download them to the box.
* Proceed to boot the box in VMware workstation and prepare the following:

#### 1. Fix accounts

Enable the local Administrator account and delete the Vagrant account by doing

* Control panel -&gt; User accounts -&gt; Manage another account -&gt; Administrator -&gt; Set a password for the Administrator account -&gt; Log out
* Log in as Administrator using the new password, go into Control panel -&gt; Users -&gt; Remove User Acccounts -&gt; Delete the Vagrant account -&gt; Click delete files

#### 2. Install VMware tools

Do it through the VMware workstation interface. Should be self explanatory.

#### 3. Windows Update

* Use this WU.ps1 script to download and install updates for the operating system.
* Open powershell.exe as an Administrator and run `Import-Module C:\Users\Administrator\Desktop\WU.ps1`
* This must potentially be performed numerous times with several reboots until there are no more updates to apply. Just keep running it until it says there are no more updates.

#### 4. Installing .Net framework

#### 4. Run Sysprep

Sysprep will be done through the XML file provided here: link

* Change the Administrator and autologin password to the correct password
* Change the time zone. Look up Microsoft time zones values here: [https://msdn.microsoft.com/en-us/library/ms912391\(v=winembedded.11\).aspx](https://msdn.microsoft.com/en-us/library/ms912391%28v=winembedded.11%29.aspx)

Perform sysprep with the following command. OOBE is Out Of Box Experience, the startup screen welcome bullshit. The script itself preps the system and enables WinRM.

`C:\Windows\system32\sysprep\sysprep.exe /generalize /oobe /shutdown /unattend:c:\users\Administrator\Desktop\sysprep.xml`

#### 5. Verification

The VM should now be shut down and we want to verify that everything works as intended.

* Go to VM -&gt; Manage -&gt; Clone -&gt; Full clone and make a full clone of the VM. \(Takes ages\)
* Boot the clone and verify that everything was set correctly.
* Shut down and delete the clone
* Make a copy of the VM you have fixed and put it in the `boxes` folder.
* Rename the folder to Server2016 or whatever name you prefer.
* If you are short on disk space, delete the original VMs downloaded from Vagrant cloud and/or clones.

## Deploying VMs with Vagrant

### Initialize repo

Initialize a repo. This, amongst other files creates the very important Vagrantfile which holds the deployment configuration.

`vagrant init`

### Vagrantfile configuration

The documentation fro the vmware esxi plugin has examples and configurations.

[https://github.com/josenk/vagrant-vmware-esxi/wiki/Vagrantfile-examle:-Single-Machine,-fully-documented](https://github.com/josenk/vagrant-vmware-esxi/wiki/Vagrantfile-examle:-Single-Machine,-fully-documented).

[https://www.vagrantup.com/docs/vagrantfile/machine\_settings.html](https://www.vagrantup.com/docs/vagrantfile/machine_settings.html)  
Each define tag is one box, so you can have multiple boxes, in fact your entire lab in just one Vagrantfile.

Set the name of the box and pointer to the box you downloaded in previous steps. The winrm parameters specify that WinRM \(powershell remote controlling boxes\) should be used for deployment. In relation to this, many powershell scripts can be added for tasks like adding a box to a domain, setting certain system parameters, in general preparing the OS so this does not become a manual job.

The esxi parameters are at the bottom. Hostname must point to the management network virtual switch interface and the password must of course be set.

```
Vagrant.configure("2") do |config|
config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define "WEB01" do |config|
    config.vm.box = "Server2016"
    config.vm.hostname="WEB01"
    config.vm.guest = :windows
    config.vm.communicator = "winrm"
    config.vm.synced_folder "C:\\Users\\chris\\Google Drive\\Hacking\\beelabs\\AD_Files", "C:\\windows\\temp", type: "winrm"
    config.vm.boot_timeout = 100
    config.vm.graceful_halt_timeout = 100
    config.winrm.timeout = 120
    config.winrm.username = "Administrator"
    config.winrm.password = "PASSWORD"
    config.winrm.transport = :plaintext
    config.winrm.basic_auth_only = true
    config.vm.provision "shell", inline: "Rename-Computer -NewName WEB01"
    config.vm.provision :reload

    config.vm.provider :vmware_esxi do |esxi|
      esxi.esxi_hostname = "10.0.0.10"
      esxi.esxi_username = "root"
      esxi.esxi_password = "PASSWORD"
      esxi_virtual_network = "Lab Network"
      esxi.esxi_disk_store = "VMs"
      esxi.guest_memsize = "2048"
      esxi.guest_numvcpus = "2"
      esxi.mac_address = ["00:50:56:3f:01:01"]
    end
  end
  end
end
```

After the configuration file has been verified run  
`vagrant status`  
and fix eventual errors  
then do deploy the machine run  
`vagrant up`  
This takes the Vagrantfile, applies it, and uses OVFtool to deploy it to the ESXi host using the aforementioned plugin.

If the box is shut down and booting it is necessary you want to up it without provisioning it, so specify the following

`vagrant up BOX01 --no-provision`

