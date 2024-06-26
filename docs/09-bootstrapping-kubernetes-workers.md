# Bootstrapping the Kubernetes Worker Nodes

In this lab you will bootstrap two Kubernetes worker nodes. The following components will be installed: 
* [runc](https://github.com/opencontainers/runc)
* [container networking plugins](https://github.com/containernetworking/cni)
  * [CNI-Calico](https://docs.tigera.io/calico/latest/getting-started/kubernetes/hardway/install-cni-plugin)
* [containerd](https://github.com/containerd/containerd)
* [kubelet](https://kubernetes.io/docs/admin/kubelet)
* [kube-proxy](https://kubernetes.io/docs/concepts/cluster-administration/proxies)
  * clusterCIDR: "10.200.0.0/16" (kube-proxy-config.yaml)

## Prerequisites

Copy Kubernetes binaries and systemd unit files to each worker instance:

```bash
for host in node-0 node-1; do
  scp \
    downloads/runc.arm64 \
    downloads/crictl-v1.28.0-linux-amd64.tar.gz \
    downloads/containerd-1.7.8-linux-amd64.tar.gz \
    downloads/runc.amd64 \
    downloads/kubectl \
    downloads/kubelet \
    downloads/kube-proxy \
    configs/containerd-config.toml \
    configs/kubelet-config.yaml \
    configs/kube-proxy-config.yaml \
    units/containerd.service \
    units/kubelet.service \
    units/kube-proxy.service \
    root@$host:~/
done
```

The commands in this lab must be run on each worker instance: `node-0`, `node-1`. Login to the worker instance using the `ssh` command. Example:

```bash
ssh root@node-0
```

## Provisioning a Kubernetes Worker Node

Install the OS dependencies:

```bash
{
  apt-get update
  apt-get -y install socat conntrack ipset
}
```

> The socat binary enables support for the `kubectl port-forward` command.

### Disable Swap

By default, the kubelet will fail to start if [swap](https://help.ubuntu.com/community/SwapFaq) is enabled. It is [recommended](https://github.com/kubernetes/kubernetes/issues/7294) that swap be disabled to ensure Kubernetes can provide proper resource allocation and quality of service.

Verify if swap is enabled:

```bash
swapon --show
```

If output is empty then swap is not enabled. If swap is enabled run the following command to disable swap immediately:

```bash
swapoff -a
```

> To ensure swap remains off after reboot consult your Linux distro documentation.

Create the installation directories:

```bash
mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

Install the worker binaries:

```bash
{
  mkdir -p containerd
  tar -xvf crictl-v1.28.0-linux-amd64.tar.gz
  tar -xvf containerd-1.7.8-linux-amd64.tar.gz -C containerd
  mv runc.amd64 runc
  chmod +x crictl kubectl kube-proxy kubelet runc 
  mv crictl kubectl kube-proxy kubelet runc /usr/local/bin/
  mv containerd/bin/* /bin/
}
```

~~### Configure CNI Networking~~

~~Create the `bridge` network configuration file:~~

~~```bash~~
~~mv 10-bridge.conf 99-loopback.conf /etc/cni/net.d/~~
~~```~~

### Configure containerd

Install the `containerd` configuration files:

```bash
{
  mkdir -p /etc/containerd/
  mv containerd-config.toml /etc/containerd/config.toml
  mv containerd.service /etc/systemd/system/
}
```

### Configure the Kubelet

Create the `kubelet-config.yaml` configuration file:

```bash
{
  mv kubelet-config.yaml /var/lib/kubelet/
  mv kubelet.service /etc/systemd/system/
}
```

### Configure the Kubernetes Proxy

```bash
{
  mv kube-proxy-config.yaml /var/lib/kube-proxy/
  mv kube-proxy.service /etc/systemd/system/
}
```

### Start the Worker Services

```bash
{
  systemctl daemon-reload
  systemctl enable containerd kubelet kube-proxy
  systemctl start containerd kubelet kube-proxy
}
```

## Verification

The compute instances created in this tutorial will not have permission to complete this section. Run the following commands from the `jumpbox` machine.

List the registered Kubernetes nodes:

```bash
ssh root@server \
  "kubectl get nodes \
  --kubeconfig admin.kubeconfig"
```

```
NAME     STATUS     ROLES    AGE    VERSION
node-0   NotReady   <none>   4m5s   v1.28.3
node-1   NotReady   <none>   12s    v1.28.3
```

> CNI 설치가 안되어 있기 때문에 NODE의 상태가 NotReady 로 되어 있다. 

Next: [Bootstrapping Kubernetes CNI - Calico](10-bootstrapping-kubernetes-cni.md)
