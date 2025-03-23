### 개요

이 실습에서는 **jumpbox**에서 `kubectl` 명령어를 사용해 Kubernetes 클러스터에 **원격으로 접근**할 수 있도록 설정합니다.  
`admin` 사용자의 인증서를 기반으로 kubeconfig를 생성합니다.

> 이 실습의 모든 명령은 **jumpbox**에서 실행해야 합니다.

---

### 1단계: API 서버 접근성 확인

`kubectl` 설정에 앞서, `server.kubernetes.local` 호스트명이 정상적으로 API 서버에 연결되는지 확인합니다.  
이는 이전 실습에서 `/etc/hosts`에 등록된 DNS 이름입니다.

```bash
curl -k --cacert ca.crt https://server.kubernetes.local:6443/version
```

예상 출력:

```json
{
  "major": "1",
  "minor": "28",
  "gitVersion": "v1.28.3",
  ...
}
```

---

### 2단계: Admin 사용자용 kubeconfig 생성

아래 명령어는 `admin` 사용자 인증서를 사용해 `kubectl` 설정 파일을 생성합니다:

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key

  kubectl config set-context kubernetes-the-hard-way \
    --cluster=kubernetes-the-hard-way \
    --user=admin

  kubectl config use-context kubernetes-the-hard-way
}
```

이 명령은 `~/.kube/config` 파일을 생성하며, 이 파일은 `kubectl` 명령이 기본적으로 사용하는 설정 파일입니다.  
따라서 이후에는 `--kubeconfig` 옵션 없이도 `kubectl` 명령을 사용할 수 있습니다.

---

### 3단계: kubectl 원격 연결 테스트

#### 클러스터 버전 확인

```bash
kubectl version
```

예상 출력:

```text
Client Version: v1.28.3
Server Version: v1.28.3
```

#### 클러스터 노드 상태 확인

```bash
kubectl get nodes
```

예상 출력:

```text
NAME     STATUS   ROLES    AGE   VERSION
node-0   Ready    <none>   30m   v1.28.3
node-1   Ready    <none>   35m   v1.28.3
```

정상적으로 노드 상태가 조회되면, 원격에서 클러스터를 제어할 준비가 완료된 것입니다.

---

### 결론

지금까지 다음 작업을 완료했습니다:

- jumpbox에서 API 서버 연결 테스트
- `admin` 사용자용 kubeconfig 생성
- `kubectl`로 원격에서 클러스터 조회 및 명령 실행 검증

**다음 단계**  
**[Smoke Test](12-smoke-test.md)** 실습으로 넘어가 클러스터에서 실제 워크로드가 정상적으로 실행되는지 확인합니다.