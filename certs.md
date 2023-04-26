# Certificates

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
  
  WORKER0_HOST=e03b0619b63c.mylabserver.com
  WORKER0_IP=172.31.17.23
  WORKER1_HOST=e03b0619b64c.mylabserver.com
  WORKER1_IP=172.31.31.246
  
  CONTROLLER0_HOST=
  CONTROLLER0_IP=
  CONTROLLER1_HOST=
  CONTROLLER1_IP=
  
  
  ```
cat > ${WORKER0-HOST}-csr.json <<EOF
{
  "CN": "system:node:${WORKER0-HOST}",
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

cat > ${WORKER1-HOST}-csr.json <<EOF
{
  "CN": "system:node:${WORKER1-HOST}",
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
