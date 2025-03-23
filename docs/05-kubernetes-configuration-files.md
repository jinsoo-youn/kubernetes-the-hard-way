### 개요

이 실습에서는 **kubeconfig** 파일(쿠버네티스 구성 파일)을 생성합니다. 
이는 `kubelet`, `kube-proxy`, 그리고 클러스터 관리자(`admin`) 등 여러 클라이언트가 Kubernetes API 서버에 안전하게 접속하기 위해 필요한 인증 정보와 API 서버 위치 등을 포함합니다.

> **팁:** 본 문서의 모든 명령은 [TLS 인증서 생성](04-certificate-authority.md) 단계를 진행했던 디렉터리 내에서 실행해야 합니다.

### 1. 클라이언트 인증 설정

다음 대상에 대한 kubeconfig 파일을 생성합니다:

- **Kubelet**(각 노드: `node-0`, `node-1`)
- **kube-proxy**
- **kube-controller-manager**
- **kube-scheduler**
- **admin** (사람/운영자 역할)

각 kubeconfig 파일에는 다음 정보가 담깁니다:

- **클러스터 정보**(CA 인증서, API 서버 주소)
- **사용자 자격 증명**(각 구성 요소별 인증서/키)
- **컨텍스트**(특정 클러스터와 사용자 매핑)

이하 예시는 `kubectl config set-*` 명령어를 사용해 kubeconfig를 생성 및 수정하는 방법을 보여줍니다.

---

### 2. Kubelet Kubeconfig

**kubelet**은 `system:node:<hostname>`이라는 사용자 이름으로 API 서버에 인증해야 합니다. 이를 통해 Kubernetes [Node Authorizer](https://kubernetes.io/docs/admin/authorization/node/)가 노드를 식별하고 올바르게 권한 부여를 수행할 수 있습니다.

> **참고:** 아래 예시에는 `--embed-certs=true`를 사용하여 인증서와 키를 kubeconfig에 직접 포함하고 있습니다. 파일 경로를 참조하는 방법을 선호한다면 이 옵션을 제거하고 `--certificate-authority`, `--client-certificate`, `--client-key` 대신 파일 경로를 명시해 주세요.

```bash
for host in node-0 node-1; do
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443 \
    --kubeconfig=${host}.kubeconfig

  kubectl config set-credentials system:node:${host} \
    --client-certificate=${host}.crt \
    --client-key=${host}.key \
    --embed-certs=true \
    --kubeconfig=${host}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:${host} \
    --kubeconfig=${host}.kubeconfig

  kubectl config use-context default \
    --kubeconfig=${host}.kubeconfig
done
```

**생성된 파일**:
- `node-0.kubeconfig`
- `node-1.kubeconfig`

---

### 3. kube-proxy Kubeconfig

**kube-proxy**는 워커 노드에서 실행되며, Service 및 Endpoint 정보를 가져와 로드 밸런싱과 서비스 디스커버리를 수행하려면 API 서버에 접근할 자격 증명이 필요합니다.

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.crt \
    --client-key=kube-proxy.key \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default \
    --kubeconfig=kube-proxy.kubeconfig
}
```

**생성된 파일**:
- `kube-proxy.kubeconfig`

---

### 4. kube-controller-manager Kubeconfig

**kube-controller-manager**는 Pod, Service, Endpoint, ReplicaSet 등의 리소스를 관리하기 위해 API 서버와 통신해야 하며, 이에 적절한 인증 정보를 가져야 합니다.

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.crt \
    --client-key=kube-controller-manager.key \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default \
    --kubeconfig=kube-controller-manager.kubeconfig
}
```

**생성된 파일**:
- `kube-controller-manager.kubeconfig`

---

### 5. kube-scheduler Kubeconfig

**kube-scheduler**는 API 서버에 스케줄링 대상인 노드 정보, Pod 정보 등을 요청하기 위해 kubeconfig를 사용합니다.

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.crt \
    --client-key=kube-scheduler.key \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default \
    --kubeconfig=kube-scheduler.kubeconfig
}
```

**생성된 파일**:
- `kube-scheduler.kubeconfig`

---

### 6. admin Kubeconfig

**admin** 사용자는 클러스터 관리자가 주로 사용하는 계정입니다. 서버(localhost)나 로컬 워크스테이션에서 `kubectl`을 사용해 클러스터에 접근할 때 이 kubeconfig를 활용합니다.

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default \
    --kubeconfig=admin.kubeconfig
}
```

**생성된 파일**:
- `admin.kubeconfig`

---

### 7. Kubernetes 구성 파일 배포

생성된 kubeconfig들을 해당 노드에 배포해야 합니다.

#### 7.1 워커 노드(kubelet, kube-proxy)

```bash
for host in node-0 node-1; do
  ssh root@${host} "mkdir -p /var/lib/{kube-proxy,kubelet}"

  scp kube-proxy.kubeconfig \
    root@${host}:/var/lib/kube-proxy/kubeconfig

  scp ${host}.kubeconfig \
    root@${host}:/var/lib/kubelet/kubeconfig
done
```

- 각 **kubelet**은 `<hostname>.kubeconfig`를 사용합니다.
- 노드별 **kube-proxy**는 `kube-proxy.kubeconfig` 파일을 사용합니다.

#### 7.2 컨트롤 플레인(`server`)

```bash
scp \
  admin.kubeconfig \
  kube-controller-manager.kubeconfig \
  kube-scheduler.kubeconfig \
  root@server:~/
```

- **admin.kubeconfig**: 컨트롤 플레인 노드에서 직접 클러스터 관리를 할 때 사용
- **kube-controller-manager**, **kube-scheduler**: 해당 서비스들이 필요한 kubeconfig 파일

---

### 결론

이로써 다음 작업을 완료했습니다:

1. **kubelet**, **kube-proxy**, **kube-controller-manager**, **kube-scheduler**, **admin**에 대한 kubeconfig 파일 생성
2. 생성된 kubeconfig를 각 노드(워커 노드, 컨트롤 플레인)에 배포

**다음 단계**  
**[Generating the Data Encryption Config and Key](06-data-encryption-keys.md)**를 진행하여 etcd에 저장되는 중요한 데이터를 암호화하는 방법을 배워 보세요.