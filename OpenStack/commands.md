openstack aggregate list 

openstack aggregate create openstack-ssd-hosts
openstack aggregate delete openstack-ssd-hosts
	
openstack aggregate host add openstack-ssd-hosts compute1
openstack aggregate remove host openstack-ssd-hosts compute1

openstack aggregate show openstack-ssd-hosts

openstack aggregate set --propery ssd=true openstack-ssd-hosts

openstack create flavor --vcpu 1 --ram 512 --disk 5 --public t1.micro

nova flavor-key t1.micro set ssd=true

openstack flavor list 

openstack avilablity zone list 

openstack aggregate create --zone zone1 zone-aggregate

openstack aggregate add host zone-aggregate compute-02

openstack availability zone list

opensack availablity zone show zone-aggregate

### Galnce image create
```
glance image-create --name "cirros" \
  --file cirros-0.4.0-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --visibility=public
  ```

  
