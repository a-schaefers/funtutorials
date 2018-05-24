# This guide aims to help Funtoo users configure and administrate "KVM" (a hypervisor) and "Qemu" (an emulator) by using the "Libvirt" toolkit along with the GUI front-end "Virt-Manager" and finally the "Spice" protocols in order to run graphical virtual machines.

## An overview according to libvirt.org, the [libvirt] project:

* is a toolkit to manage virtualization platforms
* is accessible from C, Python, Perl, Java and more
* is licensed under open source licenses
* supports KVM, QEMU, Xen, Virtuozzo, VMWare ESX, LXC, BHyve and more
* targets Linux, FreeBSD, Windows and OS-X
* is used by many applications

## Check for KVM hardware support
#### To use KVM, one will need to verify the processor supports Intel Vt-x or AMD-V technology and that the necessary virtualization features are enabled within the BIOS. The following command should reveal if your hardware has virtualization enabed:

~~~~
LC_ALL=C lscpu | grep Virtualization
~~~~

## Kernel configuration
#### The default Funtoo kernel, "debian-sources", has the needed KVM virtualization and virtual networking features enabled by default and will not require any reconfiguration. Non debian-sources users will need to verify the necessary kernel features are turned on in order to run KVM virtual machines and use virtual networking. (*a link to the funtoo KVM page should go here, but the funtoo KVM page is outdated currently Wed May 23 12:09:53 PDT 2018, see https://www.funtoo.org/Talk:KVM*)

## Installing libvirt
#### Optionally build libvirt with policykit support which will allow non-root users to authenticate as root in order to manage VM's and will also allow members of the libvirt group to manage VM's without using the root password.
~~~~
echo 'app-emulation/libvirt policykit' >> /etc/portage/package.use
~~~~

#### For desktop VM usage, It is recommended to build app-emulation/qemu with spice support, (Qemu will be pulled-in by emerging libvirt), for improved graphical and audio performance, clipboard-sharing and directory-sharing.
~~~~
echo 'app-emulation/qemu spice' >> /etc/portage/package.use
~~~~

#### It will likely ask to make some USE flag changes to the file /etc/portage/package.use -- if the changes look good, go ahead and add the changes it needs in order to be emerged.
~~~~
emerge -av libvirt
~~~~

##### After libvirt is finished compiling, you will have installed libvirt and pulled-in all of its' necessary dependencies, such as app-emulation/qemu and also net-firewall/ebtables and net-dns/dnsmasq for the default NAT/DHCP networking.

## Enabling the libvirtd service

#### Start the libvirtd service:
~~~~
rc-service libvirtd start
~~~~
	
#### Add the libvirtd service to the openrc default runlevel:
~~~~
rc-update add libvirtd
~~~~

## Enabling the "default" libvirt NAT

#### Set the virsh net "default" (the default libvirt NAT) to be autostarted by libvirtd	
~~~~
virsh net-autostart default
~~~~

#### Start the "default" virsh network
~~~~
virsh net-start default
~~~~

#### Restart the libvirtd service to ensure everything has taken effect.
~~~~
rc-service libvirtd restart
~~~~

#### You can verify the default NAT is up is up using ifconfig:
~~~~
ifconfig
~~~~

~~~~
...
virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
        ether 52:54:00:74:7a:ac  txqueuelen 1000  (Ethernet)
        RX packets 6  bytes 737 (737.0 B)
        RX errors 0  dropped 5  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
...
~~~~

#### You may also notice the "default" libvirt NAT inserts its' own additional iptables rules automatically upon every libvirtd restart:
~~~~
iptables -S
~~~~

~~~~
...
-A FORWARD -d 192.168.122.0/24 -o virbr0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -s 192.168.122.0/24 -i virbr0 -j ACCEPT
-A FORWARD -i virbr0 -o virbr0 -j ACCEPT
-A FORWARD -o virbr0 -j REJECT --reject-with icmp-port-unreachable
-A FORWARD -i virbr0 -j REJECT --reject-with icmp-port-unreachable
-A FORWARD -d 192.168.122.0/24 -o virbr0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -s 192.168.122.0/24 -i virbr0 -j ACCEPT
-A FORWARD -i virbr0 -o virbr0 -j ACCEPT
-A FORWARD -o virbr0 -j REJECT --reject-with icmp-port-unreachable
-A FORWARD -i virbr0 -j REJECT --reject-with icmp-port-unreachable
-A FORWARD -d 192.168.122.0/24 -o virbr0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -s 192.168.122.0/24 -i virbr0 -j ACCEPT
-A FORWARD -i virbr0 -o virbr0 -j ACCEPT
-A FORWARD -o virbr0 -j REJECT --reject-with icmp-port-unreachable
-A FORWARD -i virbr0 -j REJECT --reject-with icmp-port-unreachable
-A OUTPUT -o virbr0 -p udp -m udp --dport 68 -j ACCEPT
-A OUTPUT -o virbr0 -p udp -m udp --dport 68 -j ACCEPT
-A OUTPUT -o virbr0 -p udp -m udp --dport 68 -j ACCEPT`
~~~~

## Most virsh commands require root privileges

#### Libvirt VM's are managed using the "virsh" cli tools and the GUI front-end "Virt-Manager". Using these tools will require root privileges unless they are built with policykit support enabled.

#### as noted from man virsh(1),
> Most virsh commands require root privileges to run due to the communications channels used to talk to the hypervisor.  Running as non root will return an error.

#### Running as root, here are some example virsh commands:
~~~~
virsh list --all  # show the status of all virtual machines
virsh start foo   # start the virtual machine "foo"
virsh destroy foo # shutdown virtual machine "foo"
~~~~

#### If libvirt was built with polictykit support, non-root users can run the same example virsh commands by addressing qemu:///system and authenticating as root via policykit:
~~~~
virsh --connect qemu:///system list --all  # show the status of all virtual machines
virsh --connect qemu:///system start foo   # start the virtual machine "foo"
virsh --connect qemu:///system destroy foo # shutdown virtual machine "foo"
~~~~

## Passwordless, non-root VM administration

#### If libvirt was built with policykit support, you may want to add your user to the additional "libvirt" group in order to administrate Virtual Machines without authenticating as root. You will need log out and back in for these changes to take affect.
~~~~
gpasswd -a $USER libvirt
~~~~

## Tell Qemu where the UEFI bios firmware is located
#### /etc/libvirt/qemu.conf
~~~~
nvram = [
    "/usr/share/edk2-ovmf/OVMF_CODE.fd:/usr/share/edk2-ovmf/OVMF_VARS.fd"
]
~~~~

## Using Virt-Manager to create and configure VM templates

#### Libvirt VM Templates are configured using XML and it is strongly recommended to install Virt-Manager in order to ease the virtual machine creation and configuration process. It may also be desirable to revisit the "virsh" commands later on, as it may be necessary to use them in order to make some advanced changes to XML templates that are not shown in the GUI Virt-Manager application.

#### Make sure the "gtk" use flag is enabled for Virt-Mananger, as it is a graphical application, and also "polictykit" unless you plan to run it as root, and then begin the emerge process:
~~~~
echo 'app-emulation/virt-manager gtk policykit' >> /etc/portage/package.use
~~~~
~~~~
emerge -av virt-manager
~~~~
##### Once again it is likely to need further USE flag changes to /etc/portage/package.use -- if the changes look good, go ahead and add the changes it needs in order to be emerged.

#### At this point Virtual Machine Creation and administration should be relatively intuitive using the Virt-Manager GUI tool. What follows will be screenshots and guidelines for creating a Windows 10 Virtual Machine using Virt-Manager and the additional spice guest drivers:
 
## Using Virt-Manager

![00virt-manager.png](https://github.com/a-schaefers/funtutorials/blob/master/libvirt/00virt-manager.png)
[[File:00virt-manager.png|681px|Virt-Manager]]
[[File:01createvm.png|400px|Creating a new Virtual Machine Template]]
[[File:02nostorage.png|681px|Creating a new Virtual Machine Template]]
[[File:03isopoolname.png|370px|Creating a new Virtual Machine Template]]
[[File:04isopoollocation.png|443px|Creating a new Virtual Machine Template]]
[[File:05isopoolimage.png|750px|Creating a new Virtual Machine Template]]
[[File:06isoloaded.png|470px|Creating a new Virtual Machine Template]]
[[File:07memcpu.png|400px|Creating a new Virtual Machine Template]]
[[File:08allocstore.png|400px|Creating a new Virtual Machine Template]]
[[File:09customizebefore.png|400px|Creating a new Virtual Machine Template]]
[[File:10choosebios.png|816px|Creating a new Virtual Machine Template]]
[[File:11newsatacdrome.png|525px|Creating a new Virtual Machine Template]]
[[File:11redoingstorage.png|525px|Creating a new Virtual Machine Template]]
[[File:12begininstallation.png|681px|Creating a new Virtual Machine Template]]

![01createvm.png](https://github.com/a-schaefers/funtutorials/blob/master/libvirt/01createvm.png)

Click on File > New Virtual Machine > Local install media (ISO image or CDROM) > select "Forward"

### Choose "Use ISO image:" and select "Browse"

![02nostorage.png](https://github.com/a-schaefers/funtutorials/blob/master/libvirt/02nostorage.png)

### Select the "+" to "Add pool"
##### Note: on first use, you should create a dedicated ISO "pool" which will be a directory on your filesystem where you can store all of your ISO files. After creating the ISO pool and moving your ISO images into the directory, and then you can browse for your ISO image to use for the virtual machine install.

### Give the ISOPOOL a name and location

![03isopoolname.png](https://github.com/a-schaefers/funtutorials/blob/master/libvirt/03isopoolname.png)

![04isopoollocation.png](https://github.com/a-schaefers/funtutorials/blob/master/libvirt/04isopoollocation.png)

### Move the ISO images to be used for VM installation to the new ISOPOOL.
#### In my case, I have downloaded a Microsoft Windows 10 ISO install image from https://www.microsoft.com/en-us/software-download/windows10ISO
~~~~
mv ~/Downloads/Win10_1709_English_x64.iso ~/ISOPOOL
~~~~
#### Now upon refresh of ISOPOOL in Virt-Manager, it can be selected. Go ahead and select the ISO image and "Choose Volume", then hopefully it will automatically detect the operating system, otherwise uncheck the tick box and find the operating system you are trying to install and choose "Forward."

![05isopoolimage.png](https://github.com/a-schaefers/funtutorials/blob/master/libvirt/05isopoolimage.png)

![06isoloaded.png](https://github.com/a-schaefers/funtutorials/blob/master/libvirt/06isoloaded.png)

### Allocate Memory and CPU to the VM

![07memcpu.png](https://github.com/a-schaefers/funtutorials/blob/master/libvirt/07memcpu.png)

### Allocate Storage (or not) to the VM (This can be easily changed later)

![08allocstore.png](https://github.com/a-schaefers/funtutorials/blob/master/libvirt/08allocstore.png)

### Customize Configuration Before Install

![09customizebefore.png](https://github.com/a-schaefers/funtutorials/blob/master/libvirt/09customizebefore.png)

#### At this point it may be a good idea to check the box "Customize Configuration Before Install" because that will allow you to choose which bios firmware will be used, which is an important feature that can only be configured once per template. After you choose a bios template on the next screen, you will be unable to change bios' without starting the process all over again. Choosing to customize before install also has many other benefits that I will try to go over next, briefly.

### Choose Bios

![10choosebios.png](https://github.com/a-schaefers/funtutorials/blob/master/libvirt/10choosebios.png)

#### The default is to use seabios, which is a legacy bios. OVMF (a uefi bios) is also available. When deciding on a chipset, i440FX is for emulation of older BIOS chipsets, and Q35 is for emulation of newer BIOS chipsets. After you select your bios and choose "begin installation", you will not be able to change the bios without creating a new template again from the beginning. (All of the other options can be changed again later, but bios selection is permanenant for the template)

##### It is usually wise to keep things simple and use the legacy seabios unless you have a reason to need the UEFI bios (e.g. For pci-passthrough of GPU's, or working on UEFI or secure boot applications...)

### Other considerations in the customization menu:
* Q35 BIOS will not work with IDE, remove the IDE CDROM and IDE Storage and Select "Add Hardware" and add under "Storage" select SATA and add a new SATA cdrom and SATA harddrive instead. At this time it is important for most users to decide if they want to use the "qcow2" format which allows for snapshotting with rollbacks and rollforwards and is additionally sparse provisioned to only use the space it needs, or to choose "raw" which will have none of these things, but may have better performance under some circumstances.

![11newsatacdrom.png](https://github.com/a-schaefers/funtutorials/blob/master/libvirt/11newsatacdrom.png)

##### Adding a new, SATA cdrom Device and loading it with a windows 10 ISO image.

* Note for ZFS users: While selecting Storage drives this is an excellent time to choose the disk "Device type" of "SCSI" which will use the Redhat passthrough "Virtio Scsi" disk controller for improved I/O performance, and you can create a sparse provisioned ZVOL with "zfs create -s -V 100G tank/VOLNAME" which can be addressed in this section of the Virt-Manager Manage-box as "/dev/zvol/tank/VOLNAME". The virtio guest driver for taking advantage of the "Virtio Scsi" disk controller is included by default in Linux and FreeBSD kernels. Windows will be unable to see the Virtio Scsi storage disk and unable to install until you load the Redhat Virtio Scsi Driver during the Windows install process. (Windows installers include a menu with the ability to load drivers during the install process.) The RedHat Virtio Scsi driver for windows is available as an [ISO image](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/) 

![11redoingstorage.png](https://github.com/a-schaefers/funtutorials/blob/master/libvirt/11redoingstorage.png)

##### Adding a new, SCSI Storage Device, and addressing it to a preconfigured ZVOL

* Virt-Manager configures by default to offer the Spice server with QXL video and ich6 sound in addition to two USB redirectors and a shared clipboard. These are all generally good things to have for desktop / graphical operating systems.

* You can return to the customization menu at any time from the main Virt-Manager by selecting the virtual machine in question and choosing "open" and then selecting from the dropdown menu via "View > Details** to reconfigure the VM's general options.

##### Troubleshooting: Be sure to check your devices are enabled and in the proper order in the "boot device order" section of the "boot options" if you're unable to get beyond the bios menu!

#### When you are ready to begin, choose "Begin Installation"

![12begininstallation.png](https://github.com/a-schaefers/funtutorials/blob/master/libvirt/12begininstallation.png)

#### After successful installation of a desktop operating system, you still will need to install and enable the app-emulation/spice-vdagent service on every graphical guest machine. If it is a Windows guest machine, you can download a spice executeable from the official [spice downloads](https://www.spice-space.org) page.


~~~~
See also:
https://www.qemu.org
https://www.linux-kvm.org
https://libvirt.org
https://virt-manager.org
https://www.spice-space.org
~~~~
