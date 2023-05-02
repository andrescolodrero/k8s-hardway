# Worker Nodes

Worker nodes contains:
1. Kubelet: Control the worker node and provide API for the control plane
2. Kube-proxy: Manage iptables rules for the node and provide VNet 
3. Runtime: Download and run containers: docker, containerd, etc.

# You can install the worker binaries like so. Run these commands on both worker nodes:
```

sudo apt-get -y install socat conntrack ipset

wget -q --show-progress --https-only --timestamping \
  https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.27.0/crictl-v1.27.0-linux-arm.tar.gz \
  https://storage.googleapis.com/kubernetes-the-hard-way/runsc \
  https://github.com/opencontainers/runc/releases/download/v1.1.7/runc.amd64 \
  https://github.com/containernetworking/plugins/releases/download/v1.2.0/cni-plugins-linux-amd64-v1.2.0.tgz \
  https://github.com/containerd/containerd/releases/download/v1.6.20/containerd-1.6.20-linux-amd64.tar.gz \
  https://storage.googleapis.com/kubernetes-release/release/v1.25.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.25.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.25.0/bin/linux/amd64/kubelet

sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes

chmod +x kubectl kube-proxy kubelet runc.amd64 runsc

sudo mv runc.amd64 runc

sudo mv kubectl kube-proxy kubelet runc runsc /usr/local/bin/

sudo tar -xvf crictl-v1.27.0-linux-arm.tar.gz  -C /usr/local/bin/

sudo tar -xvf cni-plugins-linux-amd64-v1.2.0.tgz -C /opt/cni/bin/

sudo tar -xvf containerd-1.6.20-linux-amd64.tar.gz -C /
```

# Configure Containerd
Containerd is the main runtime for running containers.
mkdir -p /etc/containerd
# Create the containerd config.toml:

cat << EOF | sudo tee /etc/containerd/config.toml
[plugins]
  [plugins.cri.containerd]
    snapshotter = "overlayfs"
    [plugins.cri.containerd.default_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runc"
      runtime_root = ""
    [plugins.cri.containerd.untrusted_workload_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runsc"
      runtime_root = "/run/containerd/runsc"
EOF

# Create the containerd unit file:

cat << EOF | sudo tee /etc/systemd/system/containerd.service
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStartPre=/sbin/modprobe overlay
ExecStart=/bin/containerd
Restart=always
RestartSec=5
Delegate=yes
KillMode=process
OOMScoreAdjust=-999
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF

# Configure Kubelet
K8s worker nodes agents

set a HOSTNAME environment variable that will be used to generate your config files. Make sure you set the HOSTNAME appropriately for each worker node:

HOSTNAME=$(hostname)

# use the certs and config files

sudo mv ${HOSTNAME}-key.pem ${HOSTNAME}.pem /var/lib/kubelet/

sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig

sudo mv ca.pem /var/lib/kubernetes/


