### 개요

이 실습에서는 단일 컨트롤러 노드(`server`)에 Kubernetes **Control Plane**을 구성합니다. 설치 및 구성될 주요 구성 요소는 다음과 같습니다:

- Kubernetes API Server
- Kube Controller Manager
- Kube Scheduler

> 🔍 **참고:**  
> 본 튜토리얼은 단일 노드로 구성된 Control Plane을 사용하며, 고가용성 구성은 다루지 않습니다.

---

### 사전 준비

**jumpbox**에서 Kubernetes 바이너리 및 설정 파일을 `server` 인스턴스로 복사합니다:

```bash
scp \
  downloads/kube-apiserver \
  downloads/kube-controller-manager \
  downloads/kube-scheduler \
  downloads/kubectl \
  units/kube-apiserver.service \
  units/kube-controller-manager.service \
  units/kube-scheduler.service \
  configs/kube-scheduler.yaml \
  configs/kube-apiserver-to-kubelet.yaml \
  root@server:~/
```

그리고 `server`에 접속:

```bash
ssh root@server
```

---

### 1. 환경 준비

Kubernetes 설정 디렉터리 생성:

```bash
mkdir -p /etc/kubernetes/config
```

---

### 2. Kubernetes 바이너리 설치

실행 권한을 부여하고 시스템 경로로 이동:

```bash
{
  chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl

  mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
}
```

---

### 3. API 서버 구성

TLS 인증서 및 암호화 설정 파일 이동:

```bash
{
  mkdir -p /var/lib/kubernetes/

  mv ca.crt ca.key \
     kube-api-server.key kube-api-server.crt \
     service-accounts.key service-accounts.crt \
     encryption-config.yaml \
     /var/lib/kubernetes/
}
```

`kube-apiserver.service` 유닛 파일 배치:

```bash
mv kube-apiserver.service /etc/systemd/system/
```

> ⚠️ **주의:**  
> `--encryption-provider-config` 플래그는 암호화를 사용하지 않는 경우 제외해야 합니다.

---

### 4. 컨트롤러 매니저 구성

kubeconfig 파일 이동:

```bash
mv kube-controller-manager.kubeconfig /var/lib/kubernetes/
```

systemd 유닛 파일 배치:

```bash
mv kube-controller-manager.service /etc/systemd/system/
```

---

### 5. 스케줄러 구성

kubeconfig 이동:

```bash
mv kube-scheduler.kubeconfig /var/lib/kubernetes/
```

스케줄러 설정 파일 이동:

```bash
mv kube-scheduler.yaml /etc/kubernetes/config/
```

유닛 파일 이동:

```bash
mv kube-scheduler.service /etc/systemd/system/
```

---

### 6. 컨트롤 플레인 서비스 시작

```bash
{
  systemctl daemon-reload

  systemctl enable kube-apiserver kube-controller-manager kube-scheduler

  systemctl start kube-apiserver kube-controller-manager kube-scheduler
}
```

API 서버가 완전히 초기화될 때까지 약 5~10초가 소요됩니다.

---

### 7. 작동 확인

`admin.kubeconfig`를 사용하여 `kubectl`로 확인:

```bash
kubectl cluster-info --kubeconfig admin.kubeconfig
```

출력 예시:

```text
Kubernetes control plane is running at https://127.0.0.1:6443
```

---

### 8. Kubelet 접근을 위한 RBAC 구성

API 서버는 각 워커 노드의 Kubelet API에 접근해야 하며, 이를 위해 RBAC 권한이 필요합니다.

RBAC 정의 파일을 적용:

```bash
kubectl apply -f kube-apiserver-to-kubelet.yaml --kubeconfig admin.kubeconfig
```

> API 서버(`kubernetes` 사용자)는 `nodes/*` 관련 리소스를 조회할 수 있는 권한을 갖게 됩니다.

출력 예시:

```text
clusterrole.rbac.authorization.k8s.io/system:kube-apiserver-to-kubelet created
clusterrolebinding.rbac.authorization.k8s.io/system:kube-apiserver created
```

---

### 9. 외부에서 API 서버 확인 (jumpbox에서 실행)

```bash
curl -k --cacert ca.crt https://server.kubernetes.local:6443/version
```

출력 예시:

```json
{
  "major": "1",
  "minor": "28",
  "gitVersion": "v1.28.3",
  ...
}
```

---

### 결론

지금까지 다음 작업을 완료했습니다:

- API 서버, 컨트롤러 매니저, 스케줄러 설치 및 실행
- `kubectl` 및 `curl`을 사용한 정상 작동 확인
- API 서버가 Kubelet에 접근할 수 있도록 RBAC 구성

**다음 단계**  
**[Bootstrapping the Kubernetes Worker Nodes](09-bootstrapping-kubernetes-workers.md)**로 이동하여, 워커 노드를 클러스터에 연결합니다.