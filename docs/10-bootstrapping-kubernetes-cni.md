# Bootstrapping Kubernetes CNI - Calico

다양한 Calico 설치 방법은 아래 링크를 참고한다.
https://github.com/projectcalico/calico/tree/master/manifests

# 1. Calico vxlan 설치

```bash
ssh root@server

```

Install the operator on your cluster.

```bash
wget -O calico-vxlan.yaml https://raw.githubusercontent.com/projectcalico/calico/master/manifests/calico-vxlan.yaml

kubectl apply -f calico-vxlan.yaml
```

Change cidr block size to 24 and cidr to default ippool
```bash

kubectl get ippools default-ipv4-ippool -o yaml > ippool.yaml
sed -i 's/blockSize:.*/blockSize: 24/g' ippool.yaml
sed -i 's@cidr:.*@cidr: 10.200.0.0/16@g' ippool.yaml
kubectl apply -f ippool.yaml
```

Verify Calico installation in your cluster.
```bash
kubectl get pods -n kube-system
```

result:
```
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-57758d645c-wk86r   1/1     Running   0          7m15s
calico-node-8b4z4                          1/1     Running   0          7m15s
calico-node-99wrq                          1/1     Running   0          7m15s
```

Next: [Configuring kubectl for Remote Access](11-configuring-kubectl.md)
