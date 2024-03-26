## LXC server installation

```
apt update
apt install lxc

vi /etc/lxc/default.conf

lxc-create -n mycontainer -t ubuntu

lxc-start -n mycontainer
lxc-stop -n mycontainer
lxc-restart -n mycontainer

lxc-ls --fancy
lxc-info -n mycontainer

lxc-attach -n mycontainer

lxc-copy -n source_container -N new_container

lxc-snapshot -n mycontainer -1 snapshot0  #create snapshot

lxc-snapshot -n mycontainer -r snapshot0   #restore snapshot
```