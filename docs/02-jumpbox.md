### 개요(Overview)

이 실습에서는 네 대의 머신 중 하나를 **jumpbox**로 지정합니다. 이 jumpbox는 튜토리얼 전반에 걸쳐 Kubernetes 클러스터 설정을 위한 관리용 머신으로 사용됩니다. 여기서는 일관성을 위해 전용 jumpbox를 사용하지만, 필요하다면 macOS나 Linux가 설치된 다른 호스트에서도 이 과정을 수행할 수 있습니다.

### 사전 준비(Prerequisites)

- Debian/Ubuntu 계열 OS(Ubuntu 20.04 이상 등)에서 동작하는 머신(이 문서에서는 jumpbox로 언급)
- jumpbox에 접속할 수 있는 SSH 접근 권한
- jumpbox에 접속하기 위한 유효한 SSH 키
- Kubernetes 바이너리를 다운로드하고 저장할 수 있는 충분한 디스크 공간(최소 1GB 이상 권장)
- 필요한 패키지와 바이너리를 다운로드하기 위한 인터넷 연결

> **참고:** 본 튜토리얼에서는 편의를 위해 모든 명령을 `root` 사용자로 실행합니다. 실제 운영 환경에서는 적절한 권한을 갖춘 일반 사용자 계정을 사용하는 것을 권장합니다.

---

### 1. Jumpbox에 접속

다음 명령을 사용해 SSH 키로 jumpbox에 접속합니다:

```bash
ssh -i path/to/your/ssh/key ubuntu@<jumpbox-ip>
```

`root` 사용자로 전환합니다:

```bash
sudo su -
```

`root` 사용자로 전환되었는지 확인합니다:

```bash
whoami
```

출력 예시:
```text
root
```

---

### 2. 명령줄 도구 설치

튜토리얼 전반에서 필요한 필수 유틸리티를 설치합니다. (선택적으로) 패키지 목록을 업데이트한 뒤, 유틸리티들을 설치하세요:

```bash
apt-get update
apt-get -y install wget curl vim openssl git
```

이 도구들은 바이너리 다운로드, 편집, 리포지토리 클론, 인증서 관리를 위해 필요합니다.

---

### 3. Kubernetes The Hard Way 리포지토리 클론

튜토리얼과 관련된 설정 파일, 템플릿이 들어 있는 리포지토리를 로컬로 복제합니다:

```bash
git clone --depth 1 https://github.com/jinsoo-youn/kubernetes-the-hard-way.git
cd kubernetes-the-hard-way
```

이 디렉터리(`/root/kubernetes-the-hard-way`)에서 이후 모든 작업을 진행합니다. 작업 도중 디렉터리가 혼동되면 다음 명령으로 현재 디렉터리를 확인하세요:

```bash
pwd
```

---

### 4. 바이너리 다운로드

이제 Kubernetes와 관련된 다양한 바이너리를 다운로드하여 jumpbox에 보관합니다. 이렇게 하면 클러스터 내 다른 머신에서 여러 번 다운로드하지 않아도 됩니다.

1. **downloads 디렉터리 생성**:

   ```bash
   mkdir downloads
   ```

2. **다운로드 목록 확인**:

   ```bash
   cat downloads.txt
   ```

   이 파일에는 필요한 Kubernetes 바이너리와 관련 도구들의 다운로드 URL이 나열되어 있습니다.

3. **바이너리 다운로드**:

   ```bash
   wget -q --show-progress \
     --https-only \
     --timestamping \
     -P downloads \
     -i downloads.txt
   ```

   총 용량이 약 625MB 이상이므로 인터넷 속도에 따라 시간이 다소 걸릴 수 있습니다. 다운로드가 끝나면 다음 명령으로 파일을 확인합니다:

   ```bash
   ls -loh downloads
   ```

   예시 출력으로 `kube-apiserver`, `kube-controller-manager`, `kube-scheduler`, `kubectl`, `kubelet` 등의 파일이 보입니다.

> **팁:** 운영 환경에서는 다운로드한 파일들의 무결성을 확인하기 위해 체크섬을 검증하는 것이 좋습니다.

---

### 5. `kubectl` 설치

`kubectl`은 Kubernetes 공식 CLI 클라이언트로, 클러스터 구성 후에 API 서버를 비롯한 컨트롤 플레인과 상호 작용할 때 사용합니다.

1. **`kubectl` 실행 권한 부여 및 이동**:

   ```bash
   {
     chmod +x downloads/kubectl
     cp downloads/kubectl /usr/local/bin/
   }
   ```

2. **설치 확인**:

   ```bash
   kubectl version --client
   ```

   예시 출력:
   ```text
   Client Version: v1.28.3
   Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
   ```

---

### 결론

이제 jumpbox 환경이 다음과 같이 구성되었습니다:

- 핵심 CLI 도구(`wget`, `curl`, `vim`, `openssl`, `git`) 설치
- `downloads/` 디렉터리에 Kubernetes 바이너리 다운로드
- `/usr/local/bin/` 경로에 `kubectl` 설치 완료

이 상태에서 Kubernetes the Hard Way 튜토리얼의 나머지 단계를 진행할 준비가 되었습니다.

**다음 단계**  
다음 실습: **[Provisioning Compute Resources](03-compute-resources.md)** 로 이동하세요.