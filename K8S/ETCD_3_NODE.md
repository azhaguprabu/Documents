### ETCD Installation
```
cd /usr/local/src
wget "https://github.com/coreos/etcd/releases/download/v3.5.9/etcd-v3.5.9-linux-amd64.tar.gz"

tar -xvf etcd-v3.5.9-linux-amd64.tar.gz
mv etcd-v3.5.9-linux-amd64/etcd* /usr/local/bin

mkdir -pv /var/lib/etcd /etc/etcd

```

### Certificate Create for the ETCD cluster
```
wget csffl
wget csffl-json

openssl genrsa -out user.key 2048 

vi ca-config.json
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}

vi ca-csr.json
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IN",
      "L": "Mumbai",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Maharashtra"
    }
  ]
}

vi kubernetes.json
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IN",
      "L": "Mumbai",
      "O": "Kubernetes",
      "OU": "Kubernetes",
      "ST": "Maharashtra"
    }
  ]
}

cfssl generate the ca certificate
cfssl  generate kubernetes certificate

cp -v ca.pem kubernetes.pem kubernetes-key.pem /etc/etcd/

```

### ETCD configuration
```
vi /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos/etcd
Conflicts=etcd.service
Conflicts=etcd2.service

[Service]
Type=notify
Restart=always
RestartSec=5s
LimitNOFILE=40000
TimeoutStartSec=0

ExecStart=etcd --name 192.168.1.1 \
    --data-dir /var/lib/etcd \
    --listen-client-urls https://192.168.1.1:2379 \
    --advertise-client-urls https://192.168.1.1:2379 \
    --listen-peer-urls https://192.168.1.1:2380 \
    --initial-advertise-peer-urls https://192.168.1.1:2380 \
    --initial-cluster 192.168.1.1=https://192.168.1.1:2380,192.168.1.2=https://192.168.1.2:2380,192.168.1.3=https://192.168.1.3:2380 \
    --initial-cluster-token secrethash \
    --initial-cluster-state new \
    --client-cert-auth \
    --trusted-ca-file /etc/etcd/ca.pem \
    --cert-file /etc/etcd/kubernetes.pem \
    --key-file /etc/etcd/kubernetes-key.pem \
    --peer-client-cert-auth \
    --peer-trusted-ca-file /etc/etcd/ca.pem \
    --peer-cert-file /etc/etcd/kubernetes.pem \
    --peer-key-file /etc/etcd/kubernetes-key.pem

[Install]
WantedBy=multi-user.target
 
 ```

### ETCD service start
```
systemctl daemon-reload
systemctl start etcd
systemctl enable etcd
```

### Verify ETCD Cluster
```
HOST_1=192.168.1.1
HOST_2=192.168.1.2
HOST_3=192.168.1.3
ENDPOINTS=$HOST_1:2379,$HOST_2:2379,$HOST_3:2379
ETCDCTL_API=3 etcdctl   --cacert=/etc/etcd/ca.pem   --cert=/etc/etcd/kubernetes.pem   --key=/etc/etcd/kubernetes-key.pem --endpoints=$ENDPOINTS -w table endpoint status
ETCDCTL_API=3 etcdctl   --cacert=/etc/etcd/ca.pem   --cert=/etc/etcd/kubernetes.pem   --key=/etc/etcd/kubernetes-key.pem --endpoints=$ENDPOINTS -w table endpoint health

```
### SELinux
```
<< comming soon... >>
```

### ETCD faulty node replacement
```
HOST_1=192.168.1.1
HOST_2=192.168.1.2
ENDPOINTS=$HOST_1:2379,$HOST_2:2379
ETCDCTL_API=3 etcdctl   --cacert=/etc/etcd/ca.pem   --cert=/etc/etcd/kubernetes.pem   --key=/etc/etcd/kubernetes-key.pem --endpoints=$ENDPOINTS -w table member list
ETCDCTL_API=3 etcdctl   --cacert=/etc/etcd/ca.pem   --cert=/etc/etcd/kubernetes.pem   --key=/etc/etcd/kubernetes-key.pem --endpoints=$ENDPOINTS member remove <member ID>
ETCDCTL_API=3 etcdctl   --cacert=/etc/etcd/ca.pem   --cert=/etc/etcd/kubernetes.pem   --key=/etc/etcd/kubernetes-key.pem --endpoints=$ENDPOINTS member add 192.168.1.3 --peer-urls="https://192.168.1.3:2380"

#Modify the "new" to "existing" in service script
vi /etc/systemd/system/etcd.service
--initial-cluster-state existing

rm -rfv /var/lib/etcd/*
mkdir -pf /var/lib/etcd
chmod 0700 /var/lib/etcd
systemctl daemon-reload
systemctl start etcd
```

