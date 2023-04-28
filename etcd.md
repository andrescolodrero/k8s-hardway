# Boostrap etcd

etcd is a distributed key/value store to provide a reliable way to stora data across clusters.

Data is synced between all machines of the cluster.
Store:
- Cluster state

etcd is installed only on the controller nodes.

# Installation
```
wget -q --show-progress --https-only --timestamping \
  "https://github.com/etcd-io/etcd/releases/download/v3.4.25/etcd-v3.4.25-linux-amd64.tar.gz"

tar -xvf etcd-v3.4.25-linux-amd64.tar.gz
sudo mv etcd-v3.4.25-linux-amd64/etcd* /usr/local/bin/
mkdir /etc/etcd
mkdir /var/lib/etcd

Move the certs:
mv kubernetes.pem kubernetes-key.pem ca.pem /etc/etcd
```

ON CONTROLLER1:
```
cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd
ETCD_NAME=controller1
INTERNAL_IP=(private ip)
INITIAL_CLUSTER= (provide the initial server for now)

INITIAL_CLUSTER=controller1=https://172.31.28.168:2380,controller2=https://172.31.27.174:2380
```
ON CONTROLLER2:
```
cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd
ETCD_NAME=controller2
INTERNAL_IP=(private ip)
INITIAL_CLUSTER= (provide the initial server for now)

INITIAL_CLUSTER=controller1=https://172.31.28.168:2380,controller2=https://172.31.27.174:2380
```
## Create Service
```
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster ${INITIAL_CLUSTER} \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

NOTE: repeat on COntroller2, changing Internal-IP

## Start etcd
```

sudo systemctl daemon-reload
  sudo systemctl enable etcd
  sudo systemctl start etcd
```  
## Test
```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```  
  result:
  6eefc175575366d, started, controller1, https://172.31.28.168:2380, https://172.31.28.168:2379, false
fdc14688fa59dd0e, started, controller2, https://172.31.27.174:2380, https://172.31.27.174:2379, false
root@e03b0619b61c:/home/cloud_user# 

Possible to send/retrieve data.
``` 
On controller 1:
sudo ETCDCTL_API=3 etcdctl put msg1 "hello" \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
  
  On COntroller2:
  sudo ETCDCTL_API=3 etcdctl get msg1 \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem

msg1
hello
``` 
