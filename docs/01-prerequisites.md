아래는 **"Prerequisites"** 문서를 다듬은 개선본입니다.  
오탈자를 수정하고 문장을 더 명확하게 다듬었으며, 독자가 각 머신을 준비할 때 고려해야 할 사항과 테스트 결과 예시에 대한 설명을 추가했습니다.  
마지막으로 한글 번역본도 함께 제공했습니다.

---

## 1. 개선된 (영문) 문서

# Prerequisites

In this lab, you will review the **machine and system requirements** needed to follow this tutorial successfully.

---

## Virtual or Physical Machines

This tutorial requires **four (4)** virtual or physical machines running **Debian 12 (bookworm)** and based on the **ARM64** architecture.

Here is the recommended machine specification:

| Name    | Description            | CPU | RAM  | Storage |
|---------|------------------------|-----|------|---------|
| jumpbox | Administration host    | 2   | 2GB  | 20GB    |
| server  | Kubernetes control plane | 2 | 2GB  | 20GB    |
| node-0  | Kubernetes worker node | 2   | 2GB  | 20GB    |
| node-1  | Kubernetes worker node | 2   | 2GB  | 20GB    |

> **Note:** This tutorial is written for ARM64 architecture. If you're using x86_64, adjust binaries accordingly.

---

### Provisioning Guidance

How you provision these machines (bare metal, local VMs, cloud VMs) is up to you.  
Regardless of the environment, make sure:

- All machines meet the **hardware specs** listed above
- All machines run **Debian 12 (bookworm)**
- Machines can **communicate with each other over the network** (same subnet or VPC)

---

## System Check

Once all four machines are provisioned, verify the OS and architecture using the following command on **each** machine:

```bash
uname -mov
```

Expected output:

```text
#1 SMP PREEMPT_DYNAMIC Debian 6.1.38-4 (2024-03-08) aarch64 GNU/Linux
```

If you see:
- `aarch64` → ARM64 system
- `Debian` in the string → correct OS

You are ready to proceed.

---

**Next:** [Setting Up the Jumpbox](02-jumpbox.md)

---

## 2. 한글 번역본

# 사전 준비 사항 (Prerequisites)

이 문서에서는 본 튜토리얼을 따라가기 위해 필요한 **시스템 환경과 머신 스펙**을 확인합니다.

---

## 가상 또는 물리 머신 요구사항

이 튜토리얼은 총 **4대의 가상 머신 또는 물리 머신**을 요구하며, 운영체제는 **Ubuntu 20.04**, 아키텍처는 **x86_64** 기반이어야 합니다.

다음은 각 머신의 권장 스펙입니다:

| 이름     | 설명                        | CPU | RAM  | 스토리지 |
|----------|-----------------------------|-----|------|----------|
| jumpbox  | 관리용 호스트               | 2   | 2GB  | 20GB     |
| server   | Kubernetes 컨트롤 플레인 노드 | 2 | 2GB  | 20GB     |
| node-0   | Kubernetes 워커 노드        | 2   | 2GB  | 20GB     |
| node-1   | Kubernetes 워커 노드        | 2   | 2GB  | 20GB     |

---

### 머신 준비 시 유의사항

- 머신은 로컬 VM, 클라우드, 베어메탈 등 자유롭게 구성 가능
- 각 머신은 위 표의 하드웨어 사양과 **Ubuntu 20.04** 운영체제를 갖춰야 함
- 머신 간 네트워크 통신이 가능해야 함 (예: 동일 VPC 또는 서브넷)

---

## 시스템 확인

각 머신에서 다음 명령어를 실행해 시스템 정보를 확인합니다:

```bash
uname -mov
```

---

**다음 단계:** [Jumpbox 설정하기](02-jumpbox.md)