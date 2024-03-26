### Netplan yaml files

```
network:
  version: 2
  ethernets:
    eth0:
      optional: true
      addresses: [ 192.168.0.100/24 ]
      routes:
        - to: default
          via: 192.168.0.1
      nameservers: [ 8.8.8.8 ]
    eth1:
      addresses: []
      dhcp4: false
      optional: true
  bridges:
    br0:
      interfaces: [eth1]
      dhcp4: no
      dhcp6: no


```

### ip command 
```
ip addr add 192.168.0.10/24 dev eth0

ip link set dev eth0 up

ip route add default via 192.168.1.1 dev eth0

echo "nameserver 8.8.8.8" >> /etc/resolv.conf
```

### KVM installation
```
apt install bridge-utils cpu-checker libvirt-clients libvirt-daemon qemu qemu-kvm
```