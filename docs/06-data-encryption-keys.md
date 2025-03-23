### 개요

Kubernetes는 클러스터 상태, 애플리케이션 구성, **Secrets** 등 다양한 중요 데이터를 **etcd**에 저장합니다. 이 중 **민감한 정보**는 유휴 상태(디스크에 저장된 상태)에서 암호화하는 기능을 Kubernetes는 제공합니다.

이 실습에서는 다음 작업을 수행합니다:

- **base64 인코딩된 암호화 키**를 생성
- Kubernetes에서 사용할 **encryption-config.yaml** 파일을 작성
- 이를 **컨트롤 플레인 노드**로 복사

> **주의:**  
> 이 튜토리얼에서는 실질적인 **API 서버 설정에서 암호화를 활성화하지 않습니다.**  
> 즉:
> - 암호화 키와 설정 파일은 생성하지만,
> - `kube-apiserver.service` 설정에 아래 옵션을 **포함하지 않습니다**:
> ```bash
> --encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \
> ```

실제 환경는 아래 문서를 참고하여 암호화 설정을 적용할 수 있습니다:

---

### 1. 암호화 키 생성

다음 명령을 통해 32바이트 길이(256-bit)의 무작위 키를 생성하고 base64로 인코딩합니다:

```bash
export ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

생성된 키를 확인하려면:

```bash
echo $ENCRYPTION_KEY
```

---

### 2. 암호화 설정 파일 작성

`configs/encryption-config.yaml` 템플릿을 기반으로 실제 설정 파일을 생성합니다. 이 설정 파일은 Kubernetes API 서버가 Secrets를 암호화/복호화하는 방법을 정의합니다.

> 템플릿에는 `${ENCRYPTION_KEY}` 환경 변수가 사용되므로, `envsubst` 명령어로 해당 변수를 치환합니다.

```bash
envsubst < configs/encryption-config.yaml > encryption-config.yaml
```

**예시 템플릿 (`configs/encryption-config.yaml`)**:
```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
    - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
```

이 과정을 통해 생성된 `encryption-config.yaml`에는 고유한 암호화 키가 포함됩니다.

---

### 3. 암호화 설정 파일 배포

실제 튜토리얼에서는 API 서버에 이 파일을 적용하지 않지만, 연습 목적으로 컨트롤 플레인 노드(`server`)에 복사해 봅니다:

```bash
scp encryption-config.yaml root@server:~/
```

> 운영 환경에서는 보통 이 파일을 `/var/lib/kubernetes/encryption-config.yaml` 경로에 저장하고, API 서버 설정에 포함시켜 사용합니다.

---

### 결론

이 실습에서는 다음 작업을 완료했습니다:

1. 보안성 높은 base64 암호화 키를 생성
2. Kubernetes에서 사용할 수 있는 암호화 설정 파일(`encryption-config.yaml`) 작성
3. 설정 파일을 컨트롤 플레인 노드에 복사

> 주의: 
> 이 단계는 암호화 구성을 이해하기 위한 **참고용**입니다.

**다음 단계**  
**[Bootstrapping the etcd Cluster](07-bootstrapping-etcd.md)**로 이동하여, 컨트롤 플레인을 위한 etcd 데이터 저장소를 구성해 보세요.
