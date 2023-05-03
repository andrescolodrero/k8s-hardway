# Install 
The cfssl and cfssljson

wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssl \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssljson
  
  chmod +x cfssl cfssljson
sudo mv cfssl cfssljson /usr/local/bin/


# Certificate AUthority

```
  {

cat > ca-config.json <<EOF
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
EOF

cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IS",
      "L": "Reykjavik",
      "O": "Kubernetes",
      "OU": "CA"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

}
  ```
  
  # Client and Server Certificates
```  
  {

cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IS",
      "L": "Reykjavik",
      "O": "system:masters",
      "OU": "andrescolodrero"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin

}
  ```
  
  # Kubelet Client Cert
  We need to generate hostnames certs. IN this example i have 2 workers and im assing them a variable:
  ```
  WORKER0_HOST=worker1.internal.cloudapp.net
  WORKER0_IP=10.0.0.10
  WORKER1_HOST=worker2.internal.cloudapp.net
  WORKER1_IP=10.0.0.9
  
  CONTROLLER0_HOST=
  CONTROLLER0_IP=
  CONTROLLER1_HOST=
  CONTROLLER1_IP=
   ```
   And then create the client certs for workers
  
  ```
cat > ${WORKER0_HOST}-csr.json <<EOF
{
  "CN": "system:node:${WORKER0_HOST}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IS",
      "L": "Iceland",
      "O": "system:nodes",
      "OU": "andrescolodrero"
    }
  ]
}
EOF

cat > ${WORKER1_HOST}-csr.json <<EOF
{
  "CN": "system:node:${WORKER1_HOST}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IS",
      "L": "Iceland",
      "O": "system:nodes",
      "OU": "andrescolodrero"
    }
  ]
}
EOF



cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${WORKER0_IP},${WORKER0_HOST} \
  -profile=kubernetes \
  ${WORKER0_HOST}-csr.json | cfssljson -bare ${WORKER0_HOST}


cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${WORKER1_IP},${WORKER1_HOST} \
  -profile=kubernetes \
  ${WORKER1_HOST}-csr.json | cfssljson -bare ${WORKER1_HOST}

  ```
  # Controller Manager Certs
  Generate the `kube-controller-manager` client certificate and private key:

```
{

cat > kube-controller-manager-csr.json <<EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IS",
      "L": "Reykjavik",
      "O": "system:kube-controller-manager",
      "OU": "andrescolodrero"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager

}
```

### The Kube Proxy Client Certificate

Generate the `kube-proxy` client certificate and private key:

```
{

cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IS",
      "L": "Reykjavik",
      "O": "system:node-proxier",
      "OU": "andrescolodrero"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy

}
```

### The Scheduler Client Certificate

Generate the `kube-scheduler` client certificate and private key:

```
{

cat > kube-scheduler-csr.json <<EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IS",
      "L": "Reykjavik",
      "O": "system:kube-scheduler",
      "OU": "andrescolodrero"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler

}
```


### The Kubernetes API Server Certificate

IN Order to access the API Server, we need to provide all IPs and hostnames to this cert. This certificate wiill be also used for etcd to authenticate the nodes

IP within k8s itself, private ip of controllers: Access the API from all controllers NODES / Add load balancer too to validat ethe cert in that case / Localhost to access locally / internal kubernetes (used from inside k8s cluster too)
```
CERT_HOSTNAME=10.32.0.1,172.31.23.20,e03b0619b61c.mylabserver.com,172.31.26.41,e03b0619b62c.mylabserver.com,172.31.27.46,e03b0619b66c.mylabserver.com,127.0.0.1,localhost,kubernetes.default

(for azure)
CERT_HOSTNAME=10.32.0.1,controller1,10.0.0.4,controller2,10.0.0.5,loadbalancer,10.0.0.6,andreslb.northeurope.cloudapp.azure.com,20.223.183.180,127.0.0.1,localhost,kubernetes.default

cat > kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IS",
      "L": "Reykjavik",
      "O": "Kubernetes",
      "OU": "andrescolodrero"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${CERT_HOSTNAME} \
  -profile=kubernetes \
  kubernetes-csr.json | cfssljson -bare kubernetes

}
```
## The Service Account Key Pair

The Kubernetes Controller Manager leverages a key pair to generate and sign service account tokens as described in the [managing service accounts](https://kubernetes.io/docs/admin/service-accounts-admin/) documentation.
```
{

cat > service-account-csr.json <<EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IS",
      "L": "Reykjavik",
      "O": "Kubernetes",
      "OU": "andrescolodrero"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account

}
```

## Distribute the Client and Server Certificates

Copy certificayes over the servers

Controllers:
scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem service-account-key.pem service-account.pem  -- user/server

Workers:
scp ca.pem e03b0619b64c.mylabserver.com-key.pem e03b0619b64c.mylabserver.com.pem


