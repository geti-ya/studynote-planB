# Bitcoin Protocol — 강의 종합 정리

> 📚 출처: Exam3-1 강의 노트 (Opening ~ Top 10 Source Files)

---

## 목차

- [핵심 질문](#핵심-질문)
- [Opening — Use the Source, Luke](#opening--use-the-source-luke)
- [Entry Point & Chain Params](#entry-point--chain-params)
- [Transactions, Scripts, Serialization](#transactions-scripts-serialization)
- [Blocks, Merkle Trees, PoW](#blocks-merkle-trees-pow)
- [Validation & The UTXO Set](#validation--the-utxo-set)
- [P2P Network & Message Flow](#p2p-network--message-flow)
- [Mempool & Block Assembly](#mempool--block-assembly)
- [Architecture & Next Steps](#architecture--next-steps)
- [Top 10 Source Files](#top-10-source-files)
- [전체 흐름 요약](#전체-흐름-요약)

---

## 핵심 질문

> 비트코인의 실제 뼈대인 이 복잡하고 오래된 소스 코드(C++)는 도대체 어떻게 생겨먹었으며,  
> 새로운 개발자가 이 시스템을 망가뜨리지 않고 **가장 효과적으로 기여(Contribute)** 하려면 어떻게 첫 단추를 끼워야 하는가?

---

## Opening — Use the Source, Luke

비트코인 프로토콜에 기여하려면 다른 사람의 요약본에 의존하지 말고, 가장 궁극적인 명세서(Specification)인 **C++ 소스 코드를 직접 확인**해야 한다.

코드를 섣불리 수정하기보다 다른 사람의 코드를 **검토(Code Review)하는 것부터 시작**하는 것이 올바른 첫 단추다.

---

## Entry Point & Chain Params

### Entry Point

비트코인 프로그램이 처음 실행되는 메인 루프는 `bitcoind.cpp`다.

### Chain Params — `chainparams.h`

네트워크에 따라 달라지는 합의 매개변수를 선언하는 파일이다. 실제로 동작하는 함수 정의는 별도의 C++ 구현 파일에 있으며, `chainparams.h`는 `Add(x, y)`처럼 선언부만 존재하는 **'계약서(헤더 파일)'** 에 해당한다.

**지원하는 네트워크 종류**

| 네트워크 | 설명 |
|---------|------|
| **메인넷(Mainnet)** | 실제 경제적 가치가 오가는 실제 비트코인 네트워크 |
| **테스트넷(Testnet)** | 개발·테스트용. 실제 비트코인과 별개로 동작 |
| **시그넷(Signet)** | 통제된 환경의 테스트넷. 블록 생성에 서명이 필요 |
| **레그테스트(Regtest)** | 로컬 전용. 블록을 즉시 생성할 수 있어 개발에 적합 |

> 각 네트워크는 서로 다른 매직 바이트, 난이도, 제네시스 블록 등의 매개변수를 가진다.

---

## Transactions, Scripts, Serialization

### Transaction — `primitives/transaction.h`

가치 이전을 기록하는 기본 데이터 구조. 누가 누구에게 얼마를 보내는지를 담는 '법적 문서'에 해당하며, `CTransaction` 클래스로 정의되어 있다.

### Script — `script/script.h`

단순 결제를 넘어 다중 서명(Multi-sig)이나 시간 잠금(Time-lock) 같은 **소비 조건을 설정하는 튜링 불완전(Non-Turing-complete) 언어**다.

- `script.h`에 연산 코드(Op-codes)들이 정의되어 있다
- 스크립트를 실제로 실행하는 함수는 `EvalScript`

### Serialization — `serialize.h`

데이터를 네트워크로 전송하거나 디스크에 저장할 때 **바이트 스트림으로 변환**하는 역할을 담당한다. 비트코인 코어에서 사용하는 자체 직렬화 구현체다.

---

## Blocks, Merkle Trees, PoW

### Block — `primitives/block.h`

채굴자가 해시를 찾아내는 대상인 **80바이트짜리 블록 헤더**의 구조를 정의한다.

| 헤더 필드 | 내용 |
|----------|------|
| Version | 블록 버전 |
| PrevBlockHash | 이전 블록의 해시 (체인 연결) |
| MerkleRoot | 블록 내 모든 트랜잭션의 요약값 |
| Time | 타임스탬프 |
| Bits | 현재 난이도 목표값 |
| Nonce | 채굴자가 변경하는 값 |

### Merkle Tree

블록 안의 수천 개 트랜잭션을 두 번씩 해시(Double-SHA256)하고 쌍을 이루어, 최종적으로 **단 하나의 32바이트 값(머클 루트)**으로 압축하는 암호학적 구조다.

블록 전체를 다운받지 않아도 특정 트랜잭션의 포함 여부를 증명할 수 있다(SPV 검증).

### Proof of Work — `pow.cpp`

시스템의 불변성을 확보하고, 약 10분마다 블록이 생성되도록 **난이도를 자동으로 조절**하는 핵심 로직이 담긴 파일이다.

---

## Validation & The UTXO Set

> 코드베이스 전체에서 가장 중요한 파일 — `validation.cpp`

`validation.cpp`의 주요 함수들이 블록과 트랜잭션이 합의 규칙을 지켰는지, 서명이 유효한지 검사한다. 입력값의 유효성을 확인할 때 **UTXO(Unspent Transaction Output) 세트**를 참조하여 이중 지불(Double Spend)을 방지하며, 이 상태 데이터는 `.ldb` 파일로 디스크에 저장된다.

**처리 흐름**

```
새 블록/트랜잭션 수신
  → CheckBlock()     : 블록 형식 검사
  → CheckInputs()    : 서명 검사 + UTXO 잔액 확인
  → AcceptBlock()    : 최종 승인
  → ConnectBlock()   : 체인에 연결 및 UTXO 세트 갱신
```

---

## P2P Network & Message Flow

전 세계 피어(노드)들과의 연결 관리와 메시지 송수신은 `net_processing.cpp`가 담당한다.

### 통신 규칙 — `protocol.h`

P2P 네트워크에서 사용하는 **메시지의 종류와 형식을 정의**하는 파일이다.

- 각 메시지 앞에는 항상 4바이트짜리 **매직 바이트(Magic Bytes)** 가 붙어 어느 네트워크인지 식별한다
  - 메인넷: `0xF9BEB4D9`
- 받은 메시지의 매직 바이트를 확인해 "이게 진짜 비트코인 메인넷에서 온 메시지인지"를 먼저 판별한다

**주요 메시지 유형**

| 메시지 | 역할 |
|--------|------|
| `version` | 피어 연결 시 내 버전 정보 전달 |
| `verack` | 버전 확인 응답 |
| `addr` | 다른 피어의 IP 주소 공유 |
| `inv` | 보유한 트랜잭션/블록 목록 알림 |
| `getdata` | 특정 데이터 요청 |

### 메시지 처리 엔진 — `net_processing.cpp`

비트코인 코어에서 가장 큰 파일(5,000줄 이상)로, P2P 통신의 핵심 로직이 집중되어 있다.

- **핵심 함수**: `ProcessMessages()` — 피어로부터 들어오는 모든 메시지를 끝없이 순회하는 반복문(Loop)
- 메시지 도착 → 유형 파악 → 대응 로직 수행 → 필요 시 다른 피어에게 방송(Broadcast)
- 피어 연결 관리에는 `CNode`(개별 피어 상태)와 `CConnman`(연결 풀 전체 관리) 클래스를 사용하며, `net.h`에 정의되어 있다

---

## Mempool & Block Assembly

### Mempool — `txmempool.h`

블록에 아직 포함되지 못한 **미확인 거래들을 임시로 보관하는 메모리 풀**이다. (`CTxMemPool` 클래스)

### Block Assembly — `mining.cpp`

`CreateNewBlock` 함수가 수수료율 등을 기준으로 멤풀에서 거래들을 선별해 **새로운 블록 템플릿을 조립**한다. 채굴자는 이 템플릿을 받아 작업 증명(PoW)을 수행한다.

> `policy/policy.h`: 합의 규칙 외에 각 노드가 자체적으로 적용하는 멤풀 수락 기준을 정의하는 파일이다. 현재 'V3 트랜잭션 정책' 등 활발하게 업그레이드가 진행 중이다.

---

## Architecture & Next Steps

### 전체 디렉토리 구조

비트코인 코어는 단일 프로세스(Monolithic)로 동작하지만, 내부적으로는 역할별로 디렉토리가 분리되어 있다.

```
bitcoin/src/
├── consensus/          ← 합의 규칙 핵심 정의
├── primitives/         ← 트랜잭션, 블록 데이터 구조
│   ├── transaction.h
│   └── block.h
├── script/             ← 스크립트 언어
│   └── script.h
├── chainparams.h       ← 네트워크별 합의 매개변수
├── net.h               ← 피어 연결 관리 (CNode, CConnman)
├── net_processing.cpp  ← P2P 메시지 처리 전체 총괄
├── txmempool.h         ← 미확인 거래 관리
├── validation.cpp      ← 유효성 검사 (가장 중요)
└── pow.cpp             ← 작업 증명
```

### 기여자 로드맵

1. `doc/developer-notes.md` 등 공식 문서 정독
2. 코드 수정보다 **코드 리뷰(Review)부터** 시작
3. 퍼징(Fuzzing) 테스트로 엣지 케이스 발굴
4. 이슈 트래커에서 작은 버그부터 도전

---

## Top 10 Source Files

`protocol.h`, `chainparams.h`, `validation.cpp`, `net_processing.cpp` 등 핵심 10개 파일의 위치와 역할 정리.

### 거시적 흐름 (Macro) — P2P 통신 레이어

| 파일 | 역할 |
|------|------|
| `protocol.h` | P2P 메시지 형식 & 매직 바이트 정의 |
| `chainparams.h` | 네트워크별 합의 매개변수 선언 |
| `net_processing.cpp` | 수신·발신 메시지 처리 전체 총괄 |

### 중간 흐름 (Meso) — 대기실 & 심사대

| 파일 | 역할 |
|------|------|
| `txmempool.h` | 미확인 거래 보관 & 관리 |
| `validation.cpp` / `validation.h` | 블록·트랜잭션 합의 규칙 검사 |
| `consensus/validation.h` | 네트워크와 무관한 코어 합의 규칙 (에러 코드 포함) |
| `policy/policy.h` | 합의 규칙보다 엄격한 노드 자체 수락 기준 |

### 미시적 요소 (Micro) — 핵심 데이터 구조

| 파일 | 역할 |
|------|------|
| `primitives/block.h` | 80바이트 블록 헤더 구조 정의 |
| `primitives/transaction.h` | 비트코인 거래를 기록하는 `CTransaction` 클래스 |
| `script/script.h` | 소비 조건을 설정하는 Op-codes 목록 & `EvalScript` |

---

## 전체 흐름 요약

```
[외부 피어]
    │
    ↓ (protocol.h: 매직 바이트 & 메시지 형식 확인)
net_processing.cpp
    │  (chainparams.h: 네트워크 규칙 참조)
    │
    ├─ 새 트랜잭션 → txmempool.h (대기)
    │                    │
    │                    ↓
    └─ 새 블록 ──→ validation.cpp
                      ├─ CheckBlock()
                      ├─ CheckInputs()  ← UTXO 세트 참조
                      └─ AcceptBlock() / ConnectBlock() → 체인 연결

[채굴자]
    └─ mining.cpp → CreateNewBlock()
                      └─ txmempool.h에서 고수익 거래 선별 → 새 블록 생성
```

> **핵심**: 비트코인 코어는 `net_processing.cpp`(통신) + `validation.cpp`(검증) + `chainparams.h`(규칙) 세 축이 맞물려 돌아가는 시스템이다. 이 세 파일의 상호작용을 이해하는 것이 기여의 출발점이다.
