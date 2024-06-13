# 2. calico the hard way 방법으로 설치

https://docs.tigera.io/calico/3.27/getting-started/kubernetes/hardway/

```
ssh root@server
```

## The Calico datastore 

datastore 로 `ETCD`와 `Kubernetes` 둘 중 하나를 선택해서 구성할 수 있다. 
기본 값으로 `Kubernetes` 의 `CRD`를 사용하여 구성하는 것을 권장한다. 
본 가이드에서도 Kubernetes 를 datastore 로 사용하여 구성한다. 

### Custom Resources 

```
kubectl apply -f ./calico/crds.yaml
```


### calicoctl 

Calico datastore과 직접 연결하여 리소스를 관리하기 위해, Calico에서 제공해주는 툴인 `calicoctl` 을 설치한다. 

```
wget -O calicoctl https://github.com/projectcalico/calico/releases/download/v3.27.3/calicoctl-linux-amd64
chmod +x calicoctl
sudo mv calicoctl /usr/local/bin/
```

calicoctl 설정 

```
export KUBECONFIG=/path/to/your/kubeconfig
export DATASTORE_TYPE=kubernetes
```

## Configure IP pools 

```
cat > pool.yaml <<EOF
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: pool
spec:
  cidr: 10.200.0.0/16 
  ipipMode: Never
  natOutgoing: true
  disabled: false
  nodeSelector: all()
EOF
```

```bash
calicocctl create -f pool.yaml
```

check:
```bash
calicoctl get ippools
```

```text
NAME   CIDR            SELECTOR
pool   10.200.0.0/16   all()
```


## Install CNI plugin 

Provision Kubernetes user account for the plugin 

jumpbox 에서 calico-cni 인증서 및 kubeconfig 생성 

#### Create certificate

```bash
openssl req -newkey rsa:4096 \
  -keyout cni.key \
  -nodes \
  -out cni.csr \
  -subj "/CN=calico-cni"

openssl x509 -req -in cni.csr \
  -CA ca.crt \
  -CAkey ca.key \
  -CAcreateserial \
  -out cni.crt \
  -days 3653

# check 
ls -loh cni.*
```


result:
```
-rw-r--r-- 1 root 1.8K Jun 13 14:51 cni.crt
-rw-r--r-- 1 root 1.6K Jun 13 14:50 cni.csr
-rw------- 1 root 3.2K Jun 13 14:50 cni.key
```

#### Create kubeconfig

```bash
APISERVER=$(kubectl config view -o jsonpath='{.clusters[0].cluster.server}')
kubectl config set-cluster kubernetes \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=${APISERVER} \
    --kubeconfig=cni.kubeconfig

kubectl config set-credentials calico-cni \
    --client-certificate=cni.crt \
    --client-key=cni.key \
    --embed-certs=true \
    --kubeconfig=cni.kubeconfig

kubectl config set-context default \
    --cluster=kubernetes \
    --user=calico-cni \
    --kubeconfig=cni.kubeconfig

kubectl config use-context default --kubeconfig=cni.kubeconfig
```

#### Provision RBAC

```bash
kubectl apply -f - <<EOF
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: calico-cni
rules:
  # The CNI plugin needs to get pods, nodes, and namespaces.
  - apiGroups: [""]
    resources:
      - pods
      - nodes
      - namespaces
    verbs:
      - get
  # The CNI plugin patches pods/status.
  - apiGroups: [""]
    resources:
      - pods/status
    verbs:
      - patch
 # These permissions are required for Calico CNI to perform IPAM allocations.
  - apiGroups: ["crd.projectcalico.org"]
    resources:
      - blockaffinities
      - ipamblocks
      - ipamhandles
    verbs:
      - get
      - list
      - create
      - update
      - delete
  - apiGroups: ["crd.projectcalico.org"]
    resources:
      - ipamconfigs
      - clusterinformations
      - ippools
    verbs:
      - get
      - list
EOF
```

clusterrolebinding
```
kubectl create clusterrolebinding calico-cni --clusterrole=calico-cni --user=calico-cni
```

#### Install the Plugin

kubeconfig 복사
```
scp cni.kubeconfig node-1:/
```

각 워커노드에서 진행

calico-cni 실행파일을 Default 경로 (`/opt/cni/bin/`)로 이동
```bash
curl -L -o /opt/cni/bin/calico https://github.com/projectcalico/cni-plugin/releases/download/v3.27.3/calico-amd64
chmod 755 /opt/cni/bin/calico
curl -L -o /opt/cni/bin/calico-ipam https://github.com/projectcalico/cni-plugin/releases/download/v3.27.3/calico-ipam-amd64
chmod 755 /opt/cni/bin/calico-ipam
```

Create the config directory

```bash
mkdir -p /etc/cni/net.d/
```

Copy the kubeconfig from the previous section

```bash
cp cni.kubeconfig /etc/cni/net.d/calico-kubeconfig
chmod 600 /etc/cni/net.d/calico-kubeconfig
```

Write the CNI configuration 
```bash
cat > /etc/cni/net.d/10-calico.conflist << EOF 
{
  "name": "k8s-pod-network",
  "cniVersion": "0.3.1",
  "plugins": [
    {
      "type": "calico",
      "log_level": "info",
      "datastore_type": "kubernetes",
      "mtu": 0,
      "ipam": {
          "type": "calico-ipam",
          "assign_ipv4": "true",
          "assign_ipv6": "false"
      },
      "policy": {
          "type": "k8s"
      },
      "kubernetes": {
          "kubeconfig": "/etc/cni/net.d/calico-kubeconfig"
      }
    },
    {
      "type": "portmap",
      "snat": true,
      "capabilities": {"portMappings": true}
    }
  ]
}
EOF
```

check:
```bash
kubectl get nodes 
```

result:
```
NAME     STATUS   ROLES    AGE   VERSION
node-0   Ready    <none>   31h   v1.28.3
node-1   Ready    <none>   31h   v1.28.3
```

### Install Typha -> 생략 (필요없음)

### Install calico/node 

```bash
openssl req -newkey rsa:4096 \
  -keyout calico-node.key \
  -nodes \
  -out calico-node.csr \
  -subj "/CN=calico-node"
```


```bash
openssl x509 -req -in calico-node.csr \
  -CA ca.crt \
  -CAkey ca.key \
  -CAcreateserial \
  -out calico-node.crt \
  -days 3653
```

```bash
kubectl create secret generic -n kube-system calico-node-certs --from-file=calico-node.key --from-file=calico-node.crt
```

serviceaccount 생성 
```
kubectl create serviceaccount -n kube-system calico-node
```

clusterrole, clusterrolebinding 생성

```
kubectl create clusterrolebinding calico-node --clusterrole=calico-node --serviceaccount=kube-system:calico-node
```

#### Install daemon set 


