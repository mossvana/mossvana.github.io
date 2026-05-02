---
title: "[논문 리뷰] A Comprehensive Memory Safety Analysis of Bootloaders | NDSS 2025"
excerpt: "부트로더의 공격 표면을 체계적으로 분류하고, VM 기반 퍼징 프레임워크로 38개 신규 취약점을 발견한 연구. GRUB에서만 14개, 일부는 Secure Boot 우회 가능성 확인."
date: 2025-05-03
categories: [research]
---

> **논문 정보**
>
> - **제목:** A Comprehensive Memory Safety Analysis of Bootloaders
> - **학회:** NDSS Symposium 2025 (San Diego, CA, February 24–28, 2025)
> - **저자:** Jianqiang Wang, Meng Wang (CISPA), Qinying Wang (Zhejiang Univ.), Nils Langius (Leibniz Univ. Hannover), Li Shi (ETH Zurich), Ali Abbasi, Thorsten Holz (CISPA)
> - **PDF:** [ndss-symposium.org (무료 공개)](https://www.ndss-symposium.org/wp-content/uploads/2025-330-paper.pdf)
> - **코드:** [github.com/wjqsec/bootloader](https://github.com/wjqsec/bootloader)

---

## 이 논문을 고른 이유

임베디드·하드웨어 보안 공부를 시작해보고 싶어 선정하게 되었다. 
임베디드 시스템과 하드웨어 보안 분야는 최근 피지컬 AI(Physical AI) 기술과 다양한 스마트 디바이스의 확산과 함께 그 중요성이 점점 커지고 있다. 
특히 센서와 액추에이터를 통해 현실 세계와 상호작용하는 시스템에서는 소프트웨어뿐만 아니라 부트로더와 같은 하드웨어 초기 구동 단계에서의 보안이 매우 중요한 요소로 작용한다.
이러한 관점에서 본 논문은 부트로더 단계에서 발생할 수 있는 보안 취약점을 다루고 있어, 피지컬 AI 및 임베디드 환경 전반의 신뢰성을 이해하는 데 중요한 기초를 제공한다고 판단하였다.

4대 탑 학회(IEEE S&P, USENIX, ACM CCS, NDSS) 중 NDSS 2025에 게재된 논문이고, NDSS는 전 논문 무료 공개 정책이라 PDF를 바로 볼 수 있다는 것도 선택 이유 중 하나였다.

---

## 읽기 전에 — 3가지 질문

**이 논문이 다루는 문제는 무엇인가?**

부트로더(bootloader)는 전원이 켜진 후 펌웨어(firmware)로부터 제어권을 받아 운영체제(OS)를 메모리에 올리고 실행하는 프로그램이다. 
Secure Boot 환경에서는 펌웨어가 부트로더의 서명을 검증하고, 그 다음 부트로더가 OS 이미지를 검증하는 chain of trust 구조로 동작한다. 
이 논문은 그 부트로더 자체의 메모리 안전 취약점이 얼마나 많은지, 어떤 경로로 공격 가능한지를 체계적으로 분석한다.

**왜 이 문제가 중요한가?**

부트로더는 OS가 시작되기 전, 하드웨어 위에서 직접(bare metal) 실행되며 모든 주변장치에 높은 수준의 접근 권한을 갖는다. 여기서 취약점이 터지면 Secure Boot 신뢰 체인 전체가 무너진다. 실제로 Intel, Acer, Lenovo 등 수백 개 모델에서 이미지 파싱 라이브러리 취약점으로 Secure Boot가 우회된 사례가 있으며, shim의 HTTP 구현 취약점으로 시스템 전체가 장악될 수 있었다.

**기존 방식들은 무엇이 부족했는가?**

| 기존 연구 | 한계 |
|---|---|
| BootStomp | 모바일 기기 부트로더 대상 taint analysis. storage input에만 집중 |
| Roee | 모바일 기기 부트로더만, command line input에만 집중 |
| Axtens, Starke | GRUB/Das U-Boot 퍼징 시도했지만 command-line parsing logic에만 한정 |

---

## 1. 배경 / 문제 정의

부트로더는 전원이 켜진 후 펌웨어로부터 제어권을 받아 OS를 메모리에 올리는 프로그램이다. 
Secure Boot 환경에서는 펌웨어가 부트로더를 검증하고, 부트로더가 다시 OS 이미지를 검증하는 chain of trust 구조로 동작한다. 즉 부트로더는 신뢰 체인의 핵심 고리다.

문제는 기능 확장이다. GRUB은 20종 이상의 파일 시스템을 지원하고, 배경 이미지·폰트 커스터마이징, HTTP/TFTP 파일 다운로드까지 가능하다. 코드 규모가 커질수록 취약점이 발생할 가능성도 높아진다. 

구조적으로도 취약하다. 부트로더는 bare metal 환경에서 동작하므로 표준 라이브러리나 OS의 도움 없이 모든 것을 자체 구현해야 한다. 파일 하나를 읽으려면 파일 경로 파싱, 파일 핸들러 관리, 파일 시스템, 블록 디바이스 접근, 스토리지 드라이버까지 전부 직접 구현해야 한다.

저자들이 CVE 데이터베이스에서 수집한 85개 부트로더 취약점을 분석한 결과, 공격 경로가 세 가지로 수렴했다.

| 공격 표면 | 비율 |
|---|---|
| 스토리지 | 48% |
| 네트워크 | 28% |
| 콘솔 | 14% |
| 기타(힙 할당 버그, 사이드채널 등) | 9% |

그런데 기존 연구들은 부트로더의 다양한 입력 경로 중 일부에만 초점을 맞추는 한계가 있었다. 예를 들어, BootStomp은 모바일 부트로더를 대상으로 한 taint analysis에서 저장소 입력(storage input)에만 집중하였고, Roee의 연구는 명령줄 입력(command-line input)에만 초점을 두었다. 또한 Axtens와 Starke는 GRUB와 Das U-Boot을 대상으로 퍼징을 수행했지만, 이 역시 명령줄 파싱 로직에 한정되어 있었다.

즉, 논문에서 언급한 것처럼 부트로더의 공격 표면은 단순한 명령줄 파싱을 훨씬 넘어서는 범위에 걸쳐 있다.

---

## 2. 핵심 아이디어

이 논문의 핵심 아이디어는 두 단계로 요약된다.

**첫째, 먼저 분석하고 그 결과로 도구를 설계한다.** <br>
CVE 85개를 분류해 공격 표면 세 가지(스토리지·네트워크·콘솔)를 정의하고, 그 정의를 그대로 퍼징 프레임워크의 설계 원칙으로 삼았다. 

**둘째, 부트로더를 실제 환경 그대로 퍼징한다.** <br>
기존 퍼저를 부트로더에 적용하면 두 가지 이유로 실패한다. 
부트로더는 bare-metal 환경에서 동작하기 때문에, AFL이나 libFuzzer 같은 퍼징 도구를 직접 적용하기 어렵고 기존 sanitizer도 호환되지 않는다. 또한 부트로더를 일반 프로그램처럼 변환해 퍼징을 수행하면 실제 실행 환경과 달라서, 발견된 오류가 실제에서도 재현되지 않을 수 있다.

게다가 부트로더는 공격 가능한 지점이 다양해서, 일부 기능은 한 번의 입력만으로 테스트하기 어렵고 두 가지 동작을 동시에 조작해야 한다. 예를 들어 파일 시스템을 퍼징하려면, 파일을 읽게 만들어 파싱이 일어나도록 하면서 동시에 저장 장치 접근 과정에서 데이터를 가로채 퍼징 입력을 넣어야 한다.

이러한 문제를 해결하기 위해, 하이퍼바이저(Hypervisor) 위에서 가상 머신을 실행해 부트로더가 실제와 유사한 환경에서 동작하도록 만든 뒤 퍼징을 수행하는 방식을 사용한다.

---

## 3. 방법론 / 시스템 구성

### 전체 구조

![image](/assets/images/research/스크린샷 2026-05-03 033316.png)


┌─────────────────────────────────────────┐
│              Virtual Machine            │
│  ┌───────────────────────────────────┐  │
│  │            Bootloader             │  │
│  │  [파일 작업] [네트워크] [콘솔 입력] │  │
│  │        Peripheral Data Access     │  │
│  └───────────────────────────────────┘  │
│       ↑ Fuzz Input       ↓ Coverage     │
└───────────────────────────────────────  ┘
↓                     ↓
[Fuzzing Engine] ← [Trace Decode]
(Hypervisor / kAFL)

동작 흐름은 다음과 같다.

1. **VM 안에서 부트로더를 실제 환경 그대로 실행한다.** 
   kAFL(hypervisor 기반 퍼저)로 VM 위에서 부트로더를 실제 환경 그대로 실행한다.
   실제 컴퓨터에서 부팅하는 것처럼 동작하게 만들어, 크래시가 나도 실제 환경과 동일한 결과가 나온다.

2. **부트로더가 파일·네트워크·콘솔 입력을 받으려 할 때, 중간에서 퍼즈 입력으로 바꿔치기한다. (Fuzz Input)**
   진짜 디바이스 데이터 대신 퍼저가 만든 데이터를 넣는 방식으로, 스토리지·네트워크·콘솔 세 경로 모두에 적용된다.

3. **Intel-PT가 부트로더의 코드 실행 경로를 추적한다. (Coverage)** 
   CPU에 내장된 기능으로, 어떤 코드가 실행됐는지 기록한다. 아직 실행되지 않은 경로가 있으면 그쪽으로 입력을 유도해 더 많은 코드를 탐색한다.

4. **Trace Decode → Fuzzing Engine이 다음 입력을 생성 → 반복한다.** 
   추적 결과를 분석해 새로운 입력을 만들고, 크래시가 날 때까지 이 과정을 반복한다.

5. **크래시가 발생하면 두 가지 장치로 탐지한다.**
   부트로더는 OS가 없는 환경이라 크래시가 나도 그냥 멈춰버린다. 그래서 저자들이 직접 감지 장치를 두 가지 만들었다.
   - **페이징 + 인터럽트 핸들러:** 잘못된 메모리 주소에 접근하면 예외가 발생하도록 설정해 크래시를 포착한다.
   - **커스텀 힙 sanitizer:** 힙 영역의 경계를 넘어 쓰기가 발생했는지 감시한다. CVE-2023-40547(shim) 취약점은 힙 오버플로우가 직접 크래시를 내지 않았지만, 이 sanitizer가 magic number 변조를 감지해 찾아낼 수 있었다.


### 하니스(Harness) 설계

하니스는 소스코드에 직접 삽입하는 방식으로 구현된다. 논문이 명시적으로 오픈소스 부트로더만 대상으로 한 이유가 여기에 있다.

각 공격 표면별 트리거 시퀀스는 다음과 같다.

- **스토리지:** 디바이스 발견 → 파일 시스템 마운트 → 루트 디렉토리 열기 → 파일 열거 → 읽기/쓰기 → 닫기 → 삭제 → 언마운트. 단, 이미지·폰트 같은 파일 파서는 전체 스토리지 스택을 거치면 불필요한 중복 데이터가 많아지므로, 파싱 함수를 직접 호출하는 방식으로 별도 처리한다.
- **네트워크:** 인터페이스 발견 → IP 설정 → TFTP/HTTP/ICMP 패킷 송수신 → 콜백 트리거. TCP/IP처럼 하위 계층 프로토콜은 상위 계층 패킷 전송 시 자동으로 조립된다.
- **콘솔:** 콘솔 처리 함수를 직접 호출해 퍼즈 입력을 문자열로 전달한다. Das U-Boot의 경우 `run_command_repeatable` 함수가 이에 해당한다.

퍼즈 입력이 소진되면 호출 함수에 에러 코드를 반환해 부트로더가 정상 종료하도록 처리한다. 부트로더 초기화 단계에서는 퍼즈 입력 대신 원래 데이터를 사용해야 정상적으로 환경이 구성되므로, 플래그로 두 모드를 구분한다.

### 분석 대상

오픈소스이고 최근 2년 이내 활발히 유지보수 중인 9개 부트로더를 선정했다: GRUB, Limine, Das U-Boot, barebox, CloverBootloader, Easyboot, rEFInd, systemd-boot, shim.

---

## 4. 실험 / 평가 / 결과

각 부트로더를 대상으로 **3주씩** 퍼징을 수행했다.

| 결과 | 수치 |
|---|---|
| 발견된 취약점 (총) | 39개 |
| 이 중 신규 (previously unknown) | **38개** |
| 개발자 확인 또는 패치 | 29개 |
| CVE 발급 | 5개 |
| GRUB에서 발견 | **14개** |

서버·데스크탑 환경에서 널리 사용되는 GRUB에서 최신 버전을 대상으로 3주 퍼징으로 14개가 터졌다. 논문은 이 중 일부가 *"적절히 익스플로잇될 경우 Secure Boot 우회로 이어질 수 있다(could lead to secure boot bypass if properly exploited)"*고 밝혔다. 실제 익스플로잇이 검증된 것이 아니라 가능성 수준임을 주의해서 읽어야 한다.

### 발견된 취약점 사례 (논문 Case Study)

논문은 패치가 완료된 취약점 3개를 직접 코드로 제시했다. 유형이 각각 달라서 부트로더 취약점이 어떤 패턴으로 생기는지 잘 보여준다.

---

**Listing 1 — barebox: 힙 오버플로우 (네트워크 입력)**

```c
#define PKTSIZE 1536

char *net_alloc_packet() {
    return dma_alloc(PKTSIZE);
}

int ping_reply(...) {
    ...
    packet = net_alloc_packet();
    if (!packet) return 0;
    // heap overflow here!
    memcpy(packet, pkt, ETHER_HDR_SIZE + len);
}
```

barebox의 ARP 구현에서 발견된 취약점이다. 수신된 패킷을 고정 크기(PKTSIZE = 1536바이트) 버퍼에 복사하는데, 점보 프레임처럼 패킷이 더 클 경우 `memcpy`가 버퍼 경계를 넘어 인접 메모리를 덮어쓴다. 입력 크기를 검증하지 않은 전형적인 힙 오버플로우 패턴이다.

---

**Listing 2 — Easyboot: 0으로 나누기 (스토리지 입력)**

```c
uint32_t inodes_per_group;

void loadinode(uint32_t inode) {
    ...
    // divide by zero here!
    uint32_t block_offs = ((inode - 1) / inodes_per_group) * desc_size;
    uint32_t inode_offs = ((inode - 1) % inodes_per_group) * inode_size;
}

void _start() {
    ...
    inodes_per_group = sb->s_inodes_per_group;
}
```

EXT 파일 시스템 슈퍼블록에서 읽어온 `inodes_per_group` 값을 검증 없이 그대로 나눗셈에 사용한다. 공격자가 이 값을 0으로 조작한 파일 시스템 이미지를 USB에 담아 꽂으면 divide by zero가 발생한다. Listing 1과 달리 네트워크가 아닌 스토리지 입력 경로의 취약점이다.

---

**Listing 3 — CloverBootloader: 정수 오버플로우로 인한 힙 오버플로우 (파일 파서)**

```c
EG_IMAGE* egDecodeBMP(uint8_t *FileData, size_t FileDataLength, bool WantAlpha) {
    uint32_t RealPixelWidth;
    ...
    RealPixelWidth = BmpHeader->PixelWidth > 0 ?
                     BmpHeader->PixelWidth : -BmpHeader->PixelWidth;
    ...
    uint32_t x = 0;
    // RealPixelWidth might be smaller than 2!
    for (; x <= RealPixelWidth - 2; x += 2) {
        ...
        PixelPtr->Blue = BmpColorMap[Index].Blue;
        ...
        PixelPtr++;
    }
}
```

BMP 이미지 디코더에서 발견된 취약점이다. `RealPixelWidth`가 2보다 작을 경우 `RealPixelWidth - 2`는 unsigned integer 언더플로우로 인해 매우 큰 값이 되고, 루프가 대량의 메모리를 덮어쓴다. 입력값의 범위를 검증하지 않아 생긴 정수 오버플로우가 힙 오버플로우로 이어진 사례다. 부트로더가 이미지 파일을 파싱한다는 사실 자체가 공격 표면이 된다는 걸 보여준다.

---

세 케이스를 비교하면 공통점이 보인다. 외부 입력값(패킷 크기, 파일 시스템 메타데이터, 이미지 헤더)을 신뢰하고 검증 없이 사용했다는 것이다. 부트로더가 다루는 입력의 출처가 모두 공격자가 조작 가능한 외부 장치라는 점에서, 이 패턴은 구조적으로 반복될 수밖에 없다.

---

### 실제 CVE로 등록된 취약점 (논문 Appendix A)

논문 부록에는 이번 연구로 CVE가 발급된 5건의 상세 내용이 수록되어 있다.

**CVE-2023-4692 / CVE-2023-4693 — GRUB (스토리지, Secure Boot 우회 가능성)**

NTFS 파일 시스템 파싱 로직에서 attribute 버퍼의 끝을 검증하지 않아 범위를 벗어난 메모리 접근이 발생한다. 조작된 NTFS 이미지로 힙을 덮어쓸 수 있으며, 논문은 이 취약점이 Secure Boot 우회로 이어질 가능성이 있다고 밝혔다. 이 논문에서 발견한 CVE 중 임팩트가 가장 크다.

**CVE-2020-8432 — Das U-Boot (콘솔, double-free)**

`gpt rename` 명령 실행 실패 시 힙 버퍼가 두 번 해제되는 double-free 취약점이다. 파티션 이름에 환경변수 형태의 문자열을 넣으면 sanity check 실패를 유도할 수 있고, 이 과정에서 double-free가 트리거된다. 콘솔 입력 경로로 발생한 취약점이다.

**CVE-2022-33103 — Das U-Boot (스토리지, 버퍼 오버플로우)**

squashfs 파일 시스템은 파일 이름 길이 필드를 2바이트로 정의해 최대 65535바이트까지 허용하지만, Das U-Boot는 고정 길이 버퍼를 할당한다. 일반 도구로는 긴 파일 이름을 생성할 수 없지만 퍼저는 이를 쉽게 만들어냈다.

**CVE-2019-15937 / CVE-2019-15938 — barebox (네트워크, 버퍼 오버플로우)**

NFS로 심볼릭 링크 파일을 읽을 때 서버 응답 패킷의 경로 길이를 검증하지 않고 고정 길이(2048바이트) 전역 버퍼에 직접 복사한다. 네트워크 패킷의 길이 필드는 4바이트라 이론적으로 부트로더 전체 물리 메모리를 덮어쓸 수 있다.

**CVE-2023-40547 — shim (네트워크, 힙 오버플로우)**

HTTP 헤더의 content-length 필드 값만큼 버퍼를 할당하지만, 실제 UEFI 펌웨어 API에서 받아오는 HTTP 콘텐츠가 그보다 클 수 있다. 이번 실험에서는 힙 오버플로우가 직접 크래시를 유발하지 않았지만 커스텀 힙 sanitizer의 magic number를 덮어써 탐지됐다. sanitizer 설계가 없었다면 놓쳤을 취약점이다.

---

5개 CVE를 보면 공격 경로가 스토리지 3건, 네트워크 2건으로, 논문이 CVE 데이터베이스 분석에서 도출한 공격 표면 분류(스토리지 48%, 네트워크 28%)와 일치한다. 분석 결과가 실제 발견으로 이어진 셈이다.

---

## 5. 한계점 및 아쉬운 점

**오픈소스 전용**

저자들은 소스코드를 직접 수정해 하네스를 삽입했다. 논문도 명시적으로 *"We assume that the bootloader source code is available"*이라고 전제를 밝혔다. 클로즈드 소스 부트로더에는 그대로 적용할 수 없다.

**Intel x86 아키텍처 종속**

kAFL이 Intel VT와 Intel-PT에 의존하기 때문에 다른 아키텍처에는 직접 적용 불가능하다. 논문은 *"파일 시스템이나 네트워크 패킷 같은 고수준 데이터 처리는 아키텍처 간 일관성이 있지만, 디바이스 드라이버 구현은 특정 기기에 종속되어 아키텍처마다 다를 수 있다"*고 한계를 인정했다.

**Exploitability 분석 부재**

발견된 취약점이 실제로 어떤 조건에서 Secure Boot 우회로 이어지는지 심층 분석은 다루지 않았다. 취약점 발견 자체에 집중한 논문이다. 38개 취약점 중 실제로 공격 가능한 것이 몇 개인지는 열린 문제로 남아있다.

---

## 6. 내가 보기에 앞으로 이어서 해볼 만한 방향

보안 입문 단계에서 실제로 할 수 있는 것 위주로 적는다.

### 지금 바로 할 수 있는 것

- [ ] **CVE 하나 직접 읽어보기**
  CVE-2022-2601(GRUB 폰트 파일 힙 오버플로우)부터. [nvd.nist.gov](https://nvd.nist.gov)에서 CVE 번호로 검색하면 요약, 영향 범위, 패치 링크를 볼 수 있다. 취약점이 어떻게 기록되고 공개되는지 흐름을 익히는 것이 목적.

- [ ] **힙 오버플로우 개념 정리하기**
  이 논문의 취약점 대부분이 힙 오버플로우다. [dreamhack.io](https://dreamhack.io) 메모리 취약점 입문 강의(무료)로 개념을 잡고 오면 논문이 두 배로 이해된다.

- [ ] **논문 GitHub 저장소 구조 살펴보기**
  `git clone https://github.com/wjqsec/bootloader`
  실행까지 안 해도 된다. 코드 구조가 논문 섹션(harness / fuzzing engine / crash detection)과 어떻게 대응되는지 파악하는 것만으로도 이해가 깊어진다.

### 개념이 쌓이면

- [ ] **AFL++ 튜토리얼 따라하기**
  이 논문이 쓴 kAFL은 설정이 복잡하지만, AFL++은 입문용 튜토리얼이 잘 정리되어 있다. [Fuzzing101](https://github.com/antonio-morales/Fuzzing101) 프로젝트가 단계별로 잘 돼 있다. 퍼저가 어떻게 동작하는지 손으로 느끼면 이 논문의 방법론이 훨씬 구체적으로 보인다.

- [ ] **다음 읽을 논문: FirmRCA (IEEE S&P 2025)**
  arXiv 무료 공개 ([arxiv.org/abs/2410.18483](https://arxiv.org/abs/2410.18483)). 퍼징으로 발견한 크래시의 근본 원인을 자동으로 분석하는 도구다. 이 논문(취약점 발견) → FirmRCA(발견 후 원인 분석) 순서로 읽으면 퍼징 사이클 전체가 그려진다.

