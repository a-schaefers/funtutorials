According to libvirt.org, the libvirt project is:

"
    is a toolkit to manage virtualization platforms
    is accessible from C, Python, Perl, Java and more
    is licensed under open source licenses
    supports KVM, QEMU, Xen, Virtuozzo, VMWare ESX, LXC, BHyve and more
    targets Linux, FreeBSD, Windows and OS-X
    is used by many applications
"

What follows is a tutorial which aims to assist a Funtoo user to configure and administer "KVM" (a hypervisor) and "Qemu" (an emulator) by using the "Libvirt" toolkit along with the GUI front-end "Virt-Manager" and finally the "Spice" protocols in order to run graphical virtual machines.

To use KVM, one will need to make sure that the processor supports Intel Vt-x or AMD-V technology and that the necessary virtualization features are enabled within the BIOS.

The default Funtoo Linux kernel, "debian-sources", already has the needed KVM virtualization and virtual networking features enabled by default and will not require any reconfiguration. Non debian-sources users should verify the necessary kernel features are enabled to run KVM virtual machines and vhost-net networking as outlined here: https://wiki.gentoo.org/wiki/QEMU#Kernel

Install libvirt
	emerge -av libvirt

At this point it will likely ask to make some USE flag changes to the file /etc/portage/package.use -- if the changes look good, go ahead and add the changes it needs in order to be emerged. Also see the "TIP" that follows for more USE changes that you may want to make before finally emerging libvirt:

TIP: It is recommended to use the Red Hat "Spice" tools for improved graphical and audio performance, clipboard-sharing and directory-sharing. So add the spice USE flag for the qemu package:
	echo 'app-emulation/qemu spice' >> /etc/portage/package.use
(Note, you still will need to install and enable the app-emulation/spice-vdagent service on every graphical guest machine. If it is a Windows guest machine, you can download a spice executeable installer from the spice website.)

For non-root, passwordless VM administration by using polkit for authentication, enable the "policykit" USE flag.
	echo 'app-emulation/libvirt policykit' >> /etc/portage/package.use

If you are planning to use ZFS for backend storage, you should also add the zfs flag to libvirt.
	echo 'app-emulation/libvirt zfs' >> /etc/portage/package.use

After it is all finished compiling, we will have installed libvirt and pulled-in all of its' necessary dependencies, including net-firewall/ebtables and net-dns/dnsmasq for the default NAT/DHCP networking.

Start the libvirtd service:
	rc-service libvirtd start
	
Set the virsh net "default" (the default libvirt NAT internal networking) to be autostarted by libvirtd	
	virsh net-autostart default

Add the libvirtd service to the default runlevel so that it persists on reboots:
	rc-update add libvirtd

At this point, it is possible to begin creating, starting, and destroying VM's using libvirt's "virsh" commands as root or with your user and sudo.

E.g. As root you will run virsh commands as:
	virsh start foo   # start the virtual machine "foo"
	virsh list --all  # show all virtual machines
	virsh destroy foo # shutdown virtual machine "foo"

If you built libvirt with the "polictykit" USE flag enabled, add your system administrator, "wheel" group user to the additional "libvirt" group in order to administrate Virtual Machines without using a password:
	gpasswd -a $USER libvirt

Now a regular user that belongs to the wheel and libvirt groups can run the same commands, modified as follows:
	virsh --connect qemu:///system start foo
	virsh --connect qemu:///system destroy foo
	virsh --connect qemu:///system list --all

It is highly recommended at this point to install virt-manager in order to ease the virtual machine creation and configuration process, as it will create and manage the libvirt xml in a way that is much more manageable. It may be desirable later on to revisit the "virsh" cli commands, and it may be necessary to make advanced changes to xml templates that are not shown in the GUI "Virt-Manager" application, but for now, Virt-Manager should suffice.

Make sure the "gtk" use flag is enabled for Virt-Mananger, as it is a graphical application, and then begin the emerge process:
	Emerge -av virt-manager
Once again it is likely to need further USE flag changes to /etc/portage/package.use -- if the changes look good, go ahead and add the changes it needs in order to be emerged.

=--to be continued...


See also:
https://www.qemu.org
https://www.linux-kvm.org
https://libvirt.org
https://virt-manager.org
https://www.spice-space.org
