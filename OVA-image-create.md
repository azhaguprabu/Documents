### Qcow2 to vmdk convert
```
qemu-img convert -O qcow2 -c source.qcow2 destination.qcow2    #shring the file usage
qemu-img convert -f qcow2 -O vmdk source.qcow2 source-image.vmdk
```

OVF File generate
```

```
