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

LC_ALL=C lscpu | grep Virtualization

## Kernel configuration
#### The default Funtoo kernel, "debian-sources", has the needed KVM virtualization and virtual networking features enabled by default and will not require any reconfiguration. Non debian-sources users will need to verify the necessary kernel features are turned on in order to run KVM virtual machines and use virtual networking. (*a link to the funtoo KVM page should go here, but the funtoo KVM page is outdated currently Wed May 23 12:09:53 PDT 2018, see https://www.funtoo.org/Talk:KVM*)

## Installing libvirt
#### Optionally build libvirt with policykit support which will allow non-root users to authenticate as root in order to manage VM's and will also allow members of the libvirt group to manage VM's without using the root password.
echo 'app-emulation/libvirt policykit' >> /etc/portage/package.use

#### For desktop VM usage, It is recommended to build app-emulation/qemu with spice support, (Qemu will be pulled-in by emerging libvirt), for improved graphical and audio performance, clipboard-sharing and directory-sharing.
echo 'app-emulation/qemu spice' >> /etc/portage/package.use

#### It will likely ask to make some USE flag changes to the file /etc/portage/package.use -- if the changes look good, go ahead and add the changes it needs in order to be emerged.
emerge -av libvirt
##### After libvirt is finished compiling, you will have installed libvirt and pulled-in all of its' necessary dependencies, such as app-emulation/qemu and also net-firewall/ebtables and net-dns/dnsmasq for the default NAT/DHCP networking.

## Enabling the libvirtd service

#### Start the libvirtd service:
rc-service libvirtd start
	
#### Add the libvirtd service to the openrc default runlevel:
rc-update add libvirtd

## Enabling the "default" libvirt NAT

#### Set the virsh net "default" (the default libvirt NAT) to be autostarted by libvirtd	
virsh net-autostart default
#### Start the "default" virsh network
virsh net-start default
#### Restart the libvirtd service to ensure everything has taken effect.
rc-service libvirtd restart

#### You can verify the default NAT is up is up using ifconfig:
ifconfig

`...
virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
        ether 52:54:00:74:7a:ac  txqueuelen 1000  (Ethernet)
        RX packets 6  bytes 737 (737.0 B)
        RX errors 0  dropped 5  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
...`

#### You may also notice the "default" libvirt NAT inserts its' own additional iptables rules automatically upon every libvirtd restart:
iptables -S

...
> -A FORWARD -d 192.168.122.0/24 -o virbr0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
> -A FORWARD -s 192.168.122.0/24 -i virbr0 -j ACCEPT
> -A FORWARD -i virbr0 -o virbr0 -j ACCEPT
> -A FORWARD -o virbr0 -j REJECT --reject-with icmp-port-unreachable
> -A FORWARD -i virbr0 -j REJECT --reject-with icmp-port-unreachable
> -A FORWARD -d 192.168.122.0/24 -o virbr0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
> -A FORWARD -s 192.168.122.0/24 -i virbr0 -j ACCEPT
> -A FORWARD -i virbr0 -o virbr0 -j ACCEPT
> -A FORWARD -o virbr0 -j REJECT --reject-with icmp-port-unreachable
> -A FORWARD -i virbr0 -j REJECT --reject-with icmp-port-unreachable
> -A FORWARD -d 192.168.122.0/24 -o virbr0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
> -A FORWARD -s 192.168.122.0/24 -i virbr0 -j ACCEPT
> -A FORWARD -i virbr0 -o virbr0 -j ACCEPT
> -A FORWARD -o virbr0 -j REJECT --reject-with icmp-port-unreachable
> -A FORWARD -i virbr0 -j REJECT --reject-with icmp-port-unreachable
> -A OUTPUT -o virbr0 -p udp -m udp --dport 68 -j ACCEPT
> -A OUTPUT -o virbr0 -p udp -m udp --dport 68 -j ACCEPT
> -A OUTPUT -o virbr0 -p udp -m udp --dport 68 -j ACCEPT

## Most virsh commands require root privileges

#### Libvirt VM's are managed using the "virsh" cli tools and the GUI front-end "Virt-Manager". Using these tools will require root privileges unless they are built with policykit support enabled.

#### as noted from man virsh(1),
> Most virsh commands require root privileges to run due to the communications channels used to talk to the hypervisor.  Running as non root will return an error.

#### Running as root, here are some example virsh commands:
`virsh list --all  # show the status of all virtual machines
virsh start foo   # start the virtual machine "foo"
virsh destroy foo # shutdown virtual machine "foo"`

#### If libvirt was built with polictykit support, non-root users can run the same example virsh commands by addressing qemu:///system and authenticating as root via policykit:
`virsh --connect qemu:///system list --all  # show the status of all virtual machines
virsh --connect qemu:///system start foo   # start the virtual machine "foo"
virsh --connect qemu:///system destroy foo # shutdown virtual machine "foo"`

## Passwordless, non-root VM administration

#### If libvirt was built with policykit support, you may want to add your user to the additional "libvirt" group in order to administrate Virtual Machines without authenticating as root. You will need log out and back in for these changes to take affect.
gpasswd -a $USER libvirt

## Using Virt-Manager to create and configure VM templates

#### Libvirt VM Templates are configured using XML and it is strongly recommended to install Virt-Manager in order to ease the virtual machine creation and configuration process. It may also be desirable to revisit the "virsh" commands later on, as it may be necessary to use them in order to make some advanced changes to XML templates that are not shown in the GUI Virt-Manager application.

#### Make sure the "gtk" use flag is enabled for Virt-Mananger, as it is a graphical application, and also "polictykit" unless you plan to run it as root, and then begin the emerge process:
echo 'app-emulation/virt-manager gtk policykit' >> /etc/portage/package.use
Emerge -av virt-manager
##### Once again it is likely to need further USE flag changes to /etc/portage/package.use -- if the changes look good, go ahead and add the changes it needs in order to be emerged.




See also:
https://www.qemu.org
https://www.linux-kvm.org
https://libvirt.org
https://virt-manager.org
https://www.spice-space.org


(Note: you still will need to install and enable the app-emulation/spice-vdagent service on every graphical guest machine. If it is a Windows guest machine, you can download a spice executeable from the official [spice downloads](https://www.spice-space.org) page.
