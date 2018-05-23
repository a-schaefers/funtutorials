# This guide aims to help a Funtoo user to configure and administrate "KVM" (a hypervisor) and "Qemu" (an emulator) by using the "Libvirt" toolkit along with the GUI front-end "Virt-Manager" and finally the "Spice" protocols in order to run graphical virtual machines.

## An overview: According to libvirt.org, the [libvirt] project:

### "is a toolkit to manage virtualization platforms
### is accessible from C, Python, Perl, Java and more
### is licensed under open source licenses
### supports KVM, QEMU, Xen, Virtuozzo, VMWare ESX, LXC, BHyve and more
### targets Linux, FreeBSD, Windows and OS-X
### is used by many applications"

## Check for KVM hardware support
#### To use KVM, one will need to make sure that the processor supports Intel Vt-x or AMD-V technology and that the necessary virtualization features are enabled within the BIOS. The following command should reveal if your hardware has virtualization enabed:

LC_ALL=C lscpu | grep Virtualization

## Kernel configuration
#### The default Funtoo kernel, "debian-sources", has the needed KVM virtualization and virtual networking features enabled by default and will not require any reconfiguration. Non debian-sources users will need to verify the necessary kernel features are turned on in order to run KVM virtual machines and virtual networking. (*link to funtoo KVM page should go here, but funtoo KVM page is outdated, see https://www.funtoo.org/Talk:KVM*)

## Install libvirt

emerge -av libvirt

#### At this point it will likely ask to make some USE flag changes to the file /etc/portage/package.use -- if the changes look good, go ahead and add the changes it needs in order to be emerged. See what follows for extra USE flag tips for libvirt and related packages:

#### For non-root, passwordless VM administration by using polkit for authentication, enable the "policykit" USE flag.
echo 'app-emulation/libvirt policykit' >> /etc/portage/package.use

#### For desktop VM usage, It is recommended toi build app-emulation/qemu with spice support for improved graphical and audio performance, clipboard-sharing and directory-sharing. (You still will need to install and enable the app-emulation/spice-vdagent service on every graphical guest machine. If it is a Windows guest machine, you can download a spice executeable installer from the spice website.)
echo 'app-emulation/qemu spice' >> /etc/portage/package.use

#### After it is finished compiling, you will have installed libvirt and pulled-in all of its' necessary dependencies, including net-firewall/ebtables and net-dns/dnsmasq for the default NAT/DHCP networking.

## Enabling the libvirtd service

#### Set the virsh net "default" (the default libvirt NAT internal networking) to be autostarted by libvirtd	
virsh net-autostart default

#### Start the libvirtd service:
rc-service libvirtd start
	
#### Add the libvirtd service to the openrc default runlevel:
rc-update add libvirtd

## Some basic virsh commands

#### At this point, it is possible to begin creating, starting, and destroying VM's using libvirt's "virsh" commands as root or with your user and sudo.

#### As root you will run virsh commands as:
virsh start foo   # start the virtual machine "foo"
virsh destroy foo # shutdown virtual machine "foo"
virsh list --all  # show all virtual machines

#### If you built libvirt with the "polictykit" USE flag enabled, add your system administrator, "wheel" group user to the additional "libvirt" group in order to administrate Virtual Machines without using a password:
gpasswd -a $USER libvirt
#### You will need log out and back in at this point for these changes to take affect.

#### Now a regular user that belongs to the wheel and libvirt groups can run the same commands, modified as follows:
virsh --connect qemu:///system start foo
virsh --connect qemu:///system destroy foo
virsh --connect qemu:///system list --all

## Using Virt-Manager to create and configure a Libvirt VM

#### Libvirt VM Templates are configured using XML and It is strongly recommended to install virt-manager in order to ease the virtual machine creation and configuration process. It may be desirable also to revisit the "virsh" commands, as it may be necessary in order to make some advanced changes to xml templates that are not shown in the GUI "Virt-Manager" application.

#### Make sure the "gtk" use flag is enabled for Virt-Mananger, as it is a graphical application, and then begin the emerge process:
Emerge -av virt-manager
#### Once again it is likely to need further USE flag changes to /etc/portage/package.use -- if the changes look good, go ahead and add the changes it needs in order to be emerged.


See also:
https://www.qemu.org
https://www.linux-kvm.org
https://libvirt.org
https://virt-manager.org
https://www.spice-space.org
