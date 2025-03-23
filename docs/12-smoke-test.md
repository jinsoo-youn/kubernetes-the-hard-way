### 개요

**스모크 테스트**(Smoke Test)란 Kubernetes 클러스터가 정상 동작하는지 빠르게 확인하기 위한 최소한의 테스트입니다. 여기서는 **시크릿(Secret) 암호화, 배포(Deployment), 포트 포워딩, 로그 조회, Exec, NodePort 서비스** 등을 통해 클러스터의 핵심 기능을 점검합니다.

> **팁:** 이번 실습은 주로 **jumpbox**에서 `kubectl`을 실행하며 진행합니다.

---

### 1. 데이터 암호화 확인

Kubernetes는 [시크릿(Secret)을 etcd에 암호화](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)해 저장할 수 있습니다. 이전 단계에서 `--encryption-provider-config`를 활성화했다면, 올바르게 작동하는지 확인합니다.

1. **시크릿 생성**:

   ```bash
   kubectl create secret generic kubernetes-the-hard-way \
     --from-literal="mykey=mydata"
   ```

2. **etcd 내 데이터 확인** (컨트롤 플레인 `server`에서 실행):

   ```bash
   ssh root@server "etcdctl get /registry/secrets/default/kubernetes-the-hard-way | hexdump -C"
   ```

   `k8s:enc:aescbc:v1:key1`와 같은 문자열이 포함되어 있으면 암호화가 적용된 것입니다.

---

### 2. 배포(Deployment)

#### 2.1 Nginx 배포 생성

```bash
kubectl create deployment nginx --image=nginx:latest
```

Pods 조회:

```bash
kubectl get pods -l app=nginx
```

```text
NAME                     READY   STATUS    RESTARTS   AGE
nginx-56fcf95486-c8dnx   1/1     Running   0          8s
```

> **참고:** Pod가 `ContainerCreating` 상태에서 진행되지 않거나 `CrashLoopBackOff`가 발생한다면, 노드 상태(`kubectl get nodes`)와 런타임/네트워크(CNI) 구성을 점검하세요.

---

#### 2.2 포트 포워딩

`kubectl` 명령을 사용해 로컬 호스트에서 Pod의 포트로 접근 가능함을 확인합니다.

1. **Pod 이름 확인**:

   ```bash
   POD_NAME=$(kubectl get pods -l app=nginx -o jsonpath="{.items[0].metadata.name}")
   ```

2. **로컬 8080 → Pod 80 포트 포워딩**:

   ```bash
   kubectl port-forward $POD_NAME 8080:80
   ```

   출력 예:
   ```text
   Forwarding from 127.0.0.1:8080 -> 80
   ```
3. **다른 터미널**에서 확인:

   ```bash
   curl --head http://127.0.0.1:8080
   ```
   `HTTP/1.1 200 OK` 응답이 오면 성공입니다.

4. **포워딩 중단**: `Ctrl + C`

---

#### 2.3 로그 조회

```bash
kubectl logs $POD_NAME
```

`nginx` 접속 로그 등을 확인할 수 있습니다.

---

#### 2.4 Exec

Pod 내부 명령 실행:

```bash
kubectl exec -ti $POD_NAME -- nginx -v
```

```text
nginx version: nginx/1.27.0
```

Pod 내부에서 각종 디버깅 명령을 수행할 수 있습니다.

---

### 3. 서비스

#### 3.1 NodePort 서비스로 노출

`nginx` 배포를 NodePort 서비스로 노출하여 각 노드 IP와 특정 포트로 접근할 수 있게 합니다.

```bash
kubectl expose deployment nginx --port 80 --type NodePort
```

> 클라우드 제공자 연동 없이 `LoadBalancer` 유형은 사용할 수 없습니다.

2. **노드 포트 확인**:

   ```bash
   NODE_PORT=$(kubectl get svc nginx --output=jsonpath='{.spec.ports[0].nodePort}')
   ```

3. **각 노드에서 접속 테스트**:

   ```bash
   curl -I http://node-0:${NODE_PORT}
   curl -I http://node-1:${NODE_PORT}
   ```
   `HTTP/1.1 200 OK` 응답을 확인할 수 있습니다.

---

### 결론

이 스모크 테스트를 통해 다음 사항을 확인했습니다:

1. **시크릿 암호화**(선택적) 작동 여부
2. **Deployment** 생성 및 Pod 스케줄링 정상 동작
3. **포트 포워딩**, **로그**, **Exec** 기능 정상
4. **NodePort 서비스**를 통한 외부 접근 가능

> **운영 환경 팁:**  
> 이 테스트는 기본 기능 점검에 가깝습니다. 실제 운영에서는 스토리지(PV/PVC), 로드 밸런서, 모니터링, 오토스케일링 등 추가 검증이 필요합니다.

**다음 단계**  
**[Structure and Clean Up](13-structure-and-cleanup.md)** 문서로 이동하여 쿠버네티스 클러스터 디렉토리 구조와 클러스터 정리하는 방법에 대해 가이드합니다.
