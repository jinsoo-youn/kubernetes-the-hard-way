### 개요

이 실습에서는 Kubernetes 클러스터에 **Calico CNI** 플러그인을 설치합니다.  
Calico는 Pod 간 통신을 가능하게 하고, **VXLAN**이나 **IP-in-IP** 등의 캡슐화 방식을 지원합니다.

> 🔍 이번 실습에서는 **VXLAN 기반의 Calico**를 설치합니다. BGP 구성이 없는 환경에서 적합합니다.

---

### 참고 자료

다양한 Calico 설치 방법은 다음 공식 GitHub 저장소에서 확인할 수 있습니다:  
👉 [https://github.com/projectcalico/calico/tree/master/manifests](https://github.com/projectcalico/calico/tree/master/manifests)

---

### 1단계: 컨트롤 플레인 노드에 접속

```bash
ssh root@server
```

---

### 2단계: Calico VXLAN 매니페스트 다운로드 및 적용

VXLAN 기반의 Calico 설치 매니페스트를 다운로드합니다:

```bash
wget -O calico-vxlan.yaml https://raw.githubusercontent.com/projectcalico/calico/master/manifests/calico-vxlan.yaml
```

클러스터에 적용:

```bash
kubectl apply -f calico-vxlan.yaml
```

이 명령어는 다음을 설치합니다:

- **calico-node DaemonSet** (각 워커 노드에 배포됨)
- **calico-kube-controllers** (IP 풀 및 정책 관리)

---

### 3단계: 기본 IP Pool 수정

Calico가 사용하는 기본 IP 풀(`default-ipv4-ippool`)을 클러스터 CIDR에 맞게 수정합니다.

현재 IP 풀을 추출:

```bash
kubectl get ippools default-ipv4-ippool -o yaml > ippool.yaml
```

`blockSize`와 `cidr` 값을 수정:

```bash
sed -i 's/blockSize:.*/blockSize: 24/g' ippool.yaml
sed -i 's@cidr:.*@cidr: 10.200.0.0/16@g' ippool.yaml
```

변경 내용을 반영:

```bash
kubectl apply -f ippool.yaml
```

> 📝 `10.200.0.0/16` 범위를 사용하고, 각 노드에 `/24` 단위의 블록이 할당되도록 설정합니다.

---

### 4단계: Calico 설치 확인

`kube-system` 네임스페이스에서 Calico 관련 Pod 상태를 확인합니다:

```bash
kubectl get pods -n kube-system
```

예상 출력:

```text
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-57758d645c-wk86r   1/1     Running   0          7m15s
calico-node-8b4z4                          1/1     Running   0          7m15s
calico-node-99wrq                          1/1     Running   0          7m15s
```

이제 각 노드가 `NotReady` 상태에서 `Ready` 상태로 전환되며, Pod 간 통신이 가능해집니다.

---

### 결론

지금까지 다음 작업을 완료했습니다:

- VXLAN 기반의 Calico 설치
- 기본 IP 풀의 CIDR 및 블록 크기 설정
- Calico Pod 정상 동작 확인

**다음 단계**  
**[kubectl 원격 접속 구성하기 (11-configuring-kubectl.md)](11-configuring-kubectl.md)** 문서로 이동하여 외부에서 클러스터를 제어하는 방법을 배워봅니다.
