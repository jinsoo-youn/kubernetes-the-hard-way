### 개요

Kubernetes 구성 요소는 **상태를 저장하지 않는(stateless)** 구조입니다. 클러스터 상태 및 모든 핵심 데이터를 저장하기 위해 외부에 **일관성 있는 키-값 저장소**가 필요하며, 이 역할을 수행하는 것이 바로 [**etcd**](https://github.com/etcd-io/etcd)입니다.

> 🔍 **참고:**  
> 원본 Kubernetes the Hard Way 가이드는 **3개의 노드로 구성된 etcd 클러스터**를 구성하지만,  
> 이 튜토리얼에서는 단순화를 위해 **단일 노드 etcd 인스턴스**를 `server` VM에서 실행합니다.

---

### 사전 준비

etcd를 구성하기 위해 필요한 파일들을 jumpbox에서 컨트롤 노드(`server`)로 복사합니다:

```bash
scp \
  downloads/etcd-v3.4.27-linux-amd64.tar.gz \
  units/etcd.service \
  root@server:~/
```

복사 후 `server` 인스턴스에 SSH로 접속합니다:

```bash
ssh root@server
```

---

### 1. etcd 바이너리 설치

압축을 해제하고 실행 파일을 시스템 경로로 이동시킵니다:

```bash
{
  tar -xvf etcd-v3.4.27-linux-amd64.tar.gz
  mv etcd-v3.4.27-linux-amd64/etcd* /usr/local/bin/
}
```

---

### 2. etcd 서버 구성

필요한 디렉터리를 생성하고 권한을 설정합니다:

```bash
{
  mkdir -p /etc/etcd /var/lib/etcd
  chmod 700 /var/lib/etcd
}
```

TLS 통신을 위한 인증서를 복사합니다:

```bash
cp ca.crt kube-api-server.key kube-api-server.crt /etc/etcd/
```

이 인증서는 Kubernetes API 서버와 etcd 간의 보안 통신을 위해 사용됩니다.

> 💡 **참고:**  
> 여기서는 `kube-api-server`의 키/인증서를 재사용하고 있지만,  
> 실제 환경에서는 etcd용으로 별도의 인증서를 발급하는 것이 좋습니다.

`etcd.service` systemd 유닛 파일을 이동합니다:

```bash
mv etcd.service /etc/systemd/system/
```

유닛 파일 내에서는:
- etcd 이름이 호스트명과 일치하도록 설정되어 있어야 하며
- 로컬 호스트(127.0.0.1)만 리스닝하도록 설정됩니다
- `/etc/etcd` 경로의 TLS 인증서를 사용하는 것이 핵심입니다

---

### 3. etcd 서비스 시작

systemd 데몬을 재로딩하고 etcd를 활성화 및 실행합니다:

```bash
{
  systemctl daemon-reload
  systemctl enable etcd
  systemctl start etcd
}
```

> 🚀 이제 etcd는 시스템 서비스로 실행되고 있어야 합니다.

---

### 4. etcd 상태 확인

`etcdctl` 명령어를 사용하여 클러스터 상태를 확인합니다:

```bash
etcdctl member list
```

출력 예시:
```text
6702b0a34e2cfd39, started, server, http://127.0.0.1:2380, http://127.0.0.1:2379, false
```

`started` 상태의 멤버가 1개 확인되면, etcd가 정상적으로 동작 중인 것입니다.

---

### 결론

이 실습을 통해 다음을 완료했습니다:

- `server` 노드에 `etcd` 설치 및 구성
- systemd 유닛으로 `etcd` 서비스 실행
- `etcdctl`을 이용한 클러스터 상태 확인

**다음 단계**  
다음으로 넘어가 **[Bootstrapping the Kubernetes Control Plane](08-bootstrapping-kubernetes-controllers.md)**에서 API 서버 및 컨트롤러 컴포넌트를 구성해 봅니다.