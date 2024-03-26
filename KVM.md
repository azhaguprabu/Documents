**KVM Server Tutorial and commands**
1. KVM installation
```
# yum module install virt
# dnf install qemu-img qemu-kvm libvirt libvirt-client virt-manager virt-install virt-viewer -y
```
2. Enable IOMMU on kernel
```
#vi /etc/default/grub
	GRUB_CMDLINE_LINUX= intel_iommu=on   ( append the parameter on the line ) 
# grub-mkconfig -o /boot/grub2/grub.cfg
# system reboot
```
3. validate the host for KVM 
```
# virt-host-validate
```
4. Sample virt-install command 
```
# virt-install --virt-type kvm --name myvm --vcpu 2 --ram 2048 --disk path=/var/lib/libvirt/images/disk.qcow2,size=20 --location /var/lib/libvirt/imaes/CentOS-8-dvd1.iso --network bridge=br0 --graphics none --os-type=linux --os-variant=0centos7.0 --console pty,target_type=serial --xtra-args='console=ttyS0,115200n serial' 
```
5. OS support list
```
# osinfo-query os
```
6. GUI oVirt installation
```
yum install https://resources.ovirt.org/pub/yum-repo/ovirt-release44.rpm 
yum -y module enable javapackages-tools pki-deps postgresql:12 
yum -y update yum -y install ovirt-engine
engine-setup
```
7. virsh sample command 
```
virsh start myvm
virsh shutdown myvm
virsh console myvm --force
virsh stop myvm
virsh destroy myvm
virsh dumpxml myvm > myvm.xml
virsh define myvm.xml
virsh undefine myvm
virsh domblklist myvm --> view disk image loation
virsh net-list
```
8. SR-IOV NIC passthrou
```
virsh node-dev --tree
ethtool -i ens1f0 | grep '^driver'    #2 port SFP+ NIC Card
driver: ixgbe

modinfo info ixgbe
…..
Parm:		max_vfs (Maximum number of virtual functions to allocate per physical function – default iz zero and maximum value is 63.

modprobe -r ixgbe
modprobe ixgbe max_vfs=4        # max 4 port interface create

vi /etc/modprobe.d/ixgbe.conf
options ixgbe max_vfs=4

grub2-mkconfig -o /boot/grub2/grub.cfg
grubby --update-kernel=ALL--args="intel_iommu=on ixgbe.max_vfs=4"
lspci -nn | grep "Virtual Function"

```
List the Virtual interface
```
virsh nodedev-list | grep 04 
……
pci_0000_04_00_0
pci_0000_04_00_1
pci_0000_04_10_0
pci_0000_04_10_1
pci_0000_04_10_2
pci_0000_04_10_3
pci_0000_04_10_4
pci_0000_04_10_5
pci_0000_04_10_6
pci_0000_04_10_7
```
9. Dump the network interface
```
virsh nodedev-dumpxml pci_0000_04_10_0
```
10. Create the xml file for sriov.xml and attached into Vm
```
vi sriov.xml
<interface type='hostdev' managed='yes' >
    <source>
    <address type='pci' domain='0x0000' bus='0x04' slot='0x10' function='0x0'>
    </address>
    </source>
</interface>

virsh attach-device VM1 sriov.xml --config
```
11. Types of Network
	1. Bridge
	2. Bond
	3. Team
	4. MACVLAN
	5. IPVLAN
	6. MACVTAP/IPVTAP
	7. VXLAN
	8. VETH
	9. IPOIB
12. New MAC ID generate for VM
```
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/virtualization_administration_guide/sect-virtualization-tips_and_tricks-generating_a_new_unique_mac_address
```
