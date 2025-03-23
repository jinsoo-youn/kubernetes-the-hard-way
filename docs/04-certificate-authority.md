### 개요

이 실습에서는 간단한 **x509 인증서**를 구성하여, Kubernetes 클러스터 구동에 필요한 TLS 인증서를 발급하고 관리하는 과정을 다룹니다. 구체적으로 다음을 수행합니다:

1. **인증 기관(CA) 생성** – 자체 서명된 루트 CA 생성
2. **Kubernetes 구성 요소**(kube-apiserver, kube-controller-manager, kube-scheduler, kubelet, kube-proxy) 및 `admin` 사용자, 그리고 `service-accounts` 등을 위한 **인증서 발급**
3. **발급된 인증서**를 적절한 노드(`server`, `node-0`, `node-1`)에 배포

> **참고:** 여기서는 편의상 자체 서명(Self-Signed) CA를 사용하지만, 실제 운영 환경에서는 Vault, 외부 PKI 서비스, 기업용 CA 등을 이용해 인증서 및 개인 키를 더 안전하게 관리하는 방법을 권장합니다.

### 사전 준비

- OpenSSL이 설치된 **jumpbox** (예: `apt-get -y install openssl`)
- `server`, `node-0`, `node-1`에 대한 SSH 접근 권한 (디렉터리 생성 및 파일 복사가 가능해야 함)
- 각 Kubernetes 구성 요소의 인증서 정보를 정의한 **`ca.conf` 파일**

---

### Certificate Authority

1. **`ca.conf` 파일 검토**  
   이 파일에는 OpenSSL을 통한 인증서 발급 시 필요한 구성(주체명, Subject Alternative Names 등)이 정의되어 있습니다.
   ```bash
   cat ca.conf
   ```
   모든 내용을 완벽히 이해할 필요는 없지만, OpenSSL의 인증서 요청과 확장이 어떻게 설정되는지 파악하는 것이 중요합니다.

2. **CA 개인 키 및 자체 서명 인증서 생성**:

   ```bash
   {
     openssl genrsa -out ca.key 4096
     openssl req -x509 -new -sha512 -noenc \
       -key ca.key -days 3650 \
       -config ca.conf \
       -out ca.crt
   }
   ```
    - `ca.key`: CA 개인 키
    - `ca.crt`: 루트 CA 인증서(약 10년간 유효)

   > **주의:** 운영 환경에서는 `ca.key` 파일을 매우 안전하게 관리해야 합니다. 이 키를 입수한 누구라도 CA 서명 권한을 남용할 수 있습니다.

---

### Client와 Server 인증서 생성

여기서는 다음 대상에 대해 별도의 인증서를 생성합니다:

- **Kubernetes 구성 요소**: kube-apiserver, kube-controller-manager, kube-scheduler, kubelet, kube-proxy
- **Admin 사용자**: Kubernetes API 서버에 접속할 때 사용하는 클라이언트 인증서
- **Service accounts**: 서비스 어카운트 토큰의 서명/검증에 사용되는 키 쌍

1. **인증서 목록 정의**:

   ```bash
   certs=(
     "admin" 
     "node-0" 
     "node-1"
     "kube-proxy"
     "kube-scheduler"
     "kube-controller-manager"
     "kube-api-server"
     "service-accounts"
   )
   ```

2. **인증서 생성**:

   ```bash
   for cert in ${certs[*]}; do
     openssl genrsa -out "${cert}.key" 4096

     # CA 서명을 위한 인증서 요청 생성 (CSR)
     openssl req -new -key "${cert}.key" -sha256 \
       -config "ca.conf" -section ${cert} \
       -out "${cert}.csr"
     
     # ca.crt와 ca.key로 CSR 서명
     openssl x509 -req -days 3653 -in "${cert}.csr" \
       -copy_extensions copyall \
       -sha256 -CA "ca.crt" \
       -CAkey "ca.key" \
       -CAcreateserial \
       -out "${cert}.crt"
   done
   ```

   각 대상은 다음 파일을 갖게 됩니다:
    - 개인 키(`<name>.key`)
    - 인증서 요청(`<name>.csr`)
    - CA로 서명된 인증서(`<name>.crt`)

   생성된 파일을 확인하려면:
   ```bash
   ls -1 *.crt *.key *.csr
   ```

---

### 인증서 배포

이제 생성된 인증서를 해당 노드에 복사해야 합니다. 운영 환경에서는 보안 솔루션(암호화 전송, 시크릿 관리자 등)을 이용하여 안전하게 파일을 관리하고 전송하는 것이 좋습니다.

1. **워커 노드**(`node-0`, `node-1`)에 배포:

   ```bash
   for host in node-0 node-1; do
     ssh root@${host} mkdir -p /var/lib/kubelet/
     
     # CA 인증서
     scp ca.crt root@${host}:/var/lib/kubelet/
     
     # 각 노드별 키 쌍
     scp ${host}.crt root@${host}:/var/lib/kubelet/kubelet.crt
     scp ${host}.key root@${host}:/var/lib/kubelet/kubelet.key
   done
   ```

2. **컨트롤 플레인 노드**(`server`)에 배포:

   ```bash
   scp \
     ca.key ca.crt \
     kube-api-server.key kube-api-server.crt \
     service-accounts.key service-accounts.crt \
     root@server:~/
   ```
    - `ca.key`, `ca.crt`: 이후 추가 서명이 필요할 수 있으므로 컨트롤 플레인 노드에 배포
    - `kube-api-server.key`, `kube-api-server.crt`: kube-apiserver의 TLS 인증서
    - `service-accounts.key`, `service-accounts.crt`: 서비스 어카운트 토큰 서명용 키 쌍

> **참고:** `kube-proxy`, `kube-controller-manager`, `kube-scheduler`, `kubelet` 등 각 구성 요소별 인증서는 이후 단계에서 kubeconfig 파일을 생성할 때 사용됩니다.

---

### 결론

지금까지 다음 작업을 완료했습니다:

1. **자체 서명 CA**(루트 인증서 `ca.crt`와 개인 키 `ca.key`) 생성
2. Kubernetes 구성 요소별로 **개별 인증서**(`admin`, `node-0`, `node-1`, `kube-proxy`, `kube-scheduler`, `kube-controller-manager`, `kube-api-server`, `service-accounts`) 발급
3. 해당 인증서들을 **적절한 머신**(컨트롤 플레인 `server` 및 워커 노드)에 배포

**다음 단계**  
**[Generating Kubernetes Configuration Files for Authentication](05-kubernetes-configuration-files.md)** 문서로 이동하여, 각 구성 요소와 사용자를 위한 kubeconfig 파일을 생성해 보세요.