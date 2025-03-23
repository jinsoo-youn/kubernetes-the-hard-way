### 개요

이 실습에서는 Kubernetes의 **워커 노드**인 `node-0`과 `node-1`을 부트스트랩합니다.  
각 노드에는 다음과 같은 구성 요소들이 설치됩니다:

- `runc`: 컨테이너 런타임의 하위 계층
- `CNI plugins`: 네트워크 구성 (Calico는 이후 단계에서 설치)
- `containerd`: 컨테이너 런타임 인터페이스
- `kubelet`: 각 노드에서 Pod 관리
- `kube-proxy`: 네트워크 프록시 및 포트 포워딩

> 이 튜토리얼에서는 클러스터 네트워크 CIDR로 `10.200.0.0/16` 을 사용합니다 (`kube-proxy-config.yaml` 기준)

---

### 사전 준비

**jumpbox**에서 각 워커 노드로 필요한 파일을 복사합니다:

```bash
for host in node-0 node-1; do
  scp \
    downloads/runc.amd64 \
    downloads/crictl-v1.28.0-linux-amd64.tar.gz \
    downloads/containerd-1.7.8-linux-amd64.tar.gz \
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

각 워커 노드에 SSH 접속:

```bash
ssh root@node-0
```

---

### 1. OS 종속 패키지 설치

```bash
apt-get update
apt-get -y install socat conntrack ipset
```

- `socat`: `kubectl port-forward` 지원
- `conntrack`, `ipset`: kube-proxy 및 네트워크에 필요

---

### 2. Swap 비활성화

Kubernetes는 **swap 사용을 비활성화해야** 안정적인 자원 할당이 가능합니다.

swap 상태 확인:

```bash
swapon --show
```

swap이 활성화된 경우:

```bash
swapoff -a
```

> ⚠️ `/etc/fstab`에서 swap 항목을 주석 처리하여 재부팅 시에도 swap이 비활성화되도록 설정할 수 있습니다.

---

### 3. 디렉터리 생성

```bash
mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

---

### 4. 워커 바이너리 설치

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

---

### 5. containerd 설정

```bash
mkdir -p /etc/containerd/
mv containerd-config.toml /etc/containerd/config.toml
mv containerd.service /etc/systemd/system/
```

---

### 6. kubelet 설정

```bash
mv kubelet-config.yaml /var/lib/kubelet/
mv kubelet.service /etc/systemd/system/
```

---

### 7. kube-proxy 설정

```bash
mv kube-proxy-config.yaml /var/lib/kube-proxy/
mv kube-proxy.service /etc/systemd/system/
```

---

### 8. 서비스 실행

```bash
{
  systemctl daemon-reload
  systemctl enable containerd kubelet kube-proxy
  systemctl start containerd kubelet kube-proxy
}
```

---

### 9. 클러스터 노드 상태 확인 (jumpbox에서 실행)

```bash
ssh root@server \
  "kubectl get nodes --kubeconfig admin.kubeconfig"
```

출력 예시:

```text
NAME     STATUS     ROLES    AGE    VERSION
node-0   NotReady   <none>   4m     v1.28.3
node-1   NotReady   <none>   1m     v1.28.3
```

> CNI(Calico)를 아직 설치하지 않았기 때문에 상태는 `NotReady`로 표시됩니다.

---

### 결론

이제 다음을 완료했습니다:

- `runc`, `containerd`, `kubelet`, `kube-proxy` 설치
- 서비스 실행 및 활성화
- 워커 노드가 Kubernetes API 서버에 등록되었음을 확인

**다음 단계**  
**[Bootstrapping Kubernetes CNI - Calico](10-bootstrapping-kubernetes-cni.md)** 로 이동하여 클러스터 네트워크를 구성합니다.
