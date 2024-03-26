### cinder list disk 
```
openstack volume list --all-project
```

### cinder delete disk
```
openstack volume delete <volume-id> --force|--purge
```

### cinder set state
```
openstack volume set --state avialble|detached
```