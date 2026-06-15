# 📒 PlanB Note — Bitcoin & Lightning 학습 노트

> **비트코인과 라이트닝 네트워크 전반을 공부하며 정리한 옵시디언 기반 학습 노트 저장소입니다.**  
> 개념 이해부터 C++ 소스코드 분석, 실무 기업과제 풀이까지 담겨 있습니다.

---

## 📂 전체 구조

```
_PlanB_note_A/
├── 1_공통수업/          # Bitcoin 개념 공통 수업 (Why / How / Where)
├── 2_개발수업/          # Bitcoin 프로토콜 개발 심화 수업
├── index/              # 옵시디언 Canvas 마인드맵 파일 모음
├── 구멍난 개념 다시 채우기/  # 부족한 개념 보충 노트
└── 기업과제/            # 실무 기업과제 (Ark Protocol VTXO 검증)
```

---

## 1️⃣ 공통수업 — Bitcoin 개념 기초

### 📌 Exam 1-1 · Why Bitcoin — 왜 비트코인인가
> **핵심 질문:** 편리함은 유지하면서도, 탈중앙화된 개인 주권 화폐 시스템을 만들 수 있을까?

| 챕터 | 주제 |
|------|------|
| [1) Store of Value](1_공통수업/Exam1-1_Why%20Bitcoin/1\)%20Store%20of%20Value/1\)%20Store%20of%20Value.md) | 가치 저장 수단으로서의 비트코인 |
| [2) Medium of Exchange](1_공통수업/Exam1-1_Why%20Bitcoin/2\)%20Medium%20of%20Exchange/2\)%20Medium%20of%20Exchange.md) | 교환 수단으로서의 비트코인 |
| [3) Unit of Account](1_공통수업/Exam1-1_Why%20Bitcoin/3\)%20Unit%20of%20Account/3\)%20Unit%20of%20Account.md) | 계산 단위로서의 비트코인 |
| [4) System of Control](1_공통수업/Exam1-1_Why%20Bitcoin/4\)%20System%20of%20Control/4\)%20System%20of%20Control.md) | 통제 시스템으로서의 비트코인 |

---

### 📌 Exam 1-2 · How Bitcoin — 어떻게 작동하는가

| 챕터 | 주제 |
|------|------|
| [1) System of Control](1_공통수업/Exam1-2_How%20Bitcoin/1\)%20System%20of%20Control/1\)%20System%20of%20Control.md) | 통제 시스템의 작동 방식 |
| [2) Voucher of Knowledge](1_공통수업/Exam1-2_How%20Bitcoin/2\)%20Voucher%20of%20Knowledge/2\)%20Voucher%20of%20Knowledge.md) | 지식의 증표 (개인키/공개키) |
| [3) Proof of Work](1_공통수업/Exam1-2_How%20Bitcoin/3\)%20Proof%20of%20Work/3\)%20Proof%20of%20Work.md) | 작업 증명 메커니즘 |
| [4) Chain of Time](1_공통수업/Exam1-2_How%20Bitcoin/4\)%20Chain%20of%20Time/4\)%20Chain%20of%20Time.md) | 시간의 체인 (블록체인) |

---

### 📌 Exam 2-1 · Where Bitcoin — 비트코인은 어디에 있는가

| 챕터 | 주제 |
|------|------|
| [1) Chain of Time](1_공통수업/Exam2-1_Where%20Bitcoin/1\)%20Chain%20of%20Time/1\)%20Chain%20of%20Time.md) | 비트코인 타임체인 |
| [2) Network of Lightning](1_공통수업/Exam2-1_Where%20Bitcoin/2\)%20Network%20of%20Lightning/2\)%20Network%20of%20Lightning.md) | 라이트닝 네트워크 |
| [3) Internet of Money](1_공통수업/Exam2-1_Where%20Bitcoin/3\)%20Internet%20of%20Money/3\)%20Internet%20of%20Money.md) | 돈의 인터넷 |
| [4) Web of Trust](1_공통수업/Exam2-1_Where%20Bitcoin/4\)%20Web%20of%20Trust/4\)%20Web%20of%20Trust.md) | 신뢰의 그물망 |

---

### 📌 Exam 2-2 · Bitcoin Industry Landscape — 비트코인 산업 지형도
> [궁극적인 질문 & 답변](1_공통수업/Exam2-2_Bitcoin%20Industry%20Landscape/0\)%20기타/) — 비트코인 생태계 전반의 산업 구조 파악

---

## 2️⃣ 개발수업 — Bitcoin 프로토콜 심화

### 🔩 Exam 3-1 · Bitcoin Protocol — 비트코인 C++ 소스코드 분석
> **핵심 질문:** 비트코인 소스코드는 어떻게 생겼으며, 새로운 개발자가 망가뜨리지 않고 기여하려면 어떻게 해야 하는가?

| 챕터 | 주제 |
|------|------|
| [1) Opening — Use the Source, Luke](2_개발수업/Exam3-1_Bitcoin%20protocol/1\)%20Opening_Use%20the%20Source%2CLuke/Opening_Use%20the%20Source%2C%20Luke.md) | 비트코인 소스코드 입문 |
| [2) Entry Point & Chain Params](2_개발수업/Exam3-1_Bitcoin%20protocol/2\)%20Entry%20Point%20%26%20Chain%20Params/Entry%20Point%20%26%20Chain%20Params.md) | 진입점과 체인 매개변수 |
| [3) Transactions, Scripts, Serialization](2_개발수업/Exam3-1_Bitcoin%20protocol/3\)%20Transaction%2C%20Scripts%2C%20Serialization/Transactions%2C%20Scripts%2C%20Serialization.md) | 트랜잭션 · 스크립트 · 직렬화 |
| [4) Blocks, Merkle Trees, PoW](2_개발수업/Exam3-1_Bitcoin%20protocol/4\)%20Blocks%2C%20Merkle%20Trees%2C%20PoW/Blocks%2C%20Merkle%20Trees%2C%20PoW.md) | 블록 · 머클 트리 · 작업증명 |
| [5) Validation & The UTXO Set](2_개발수업/Exam3-1_Bitcoin%20protocol/5\)%20Validation%20%26%20The%20UTXO%20Set/Validation%20%26%20The%20UTXO%20Set.md) | 검증 로직 · UTXO 집합 |
| [6) P2P Network & Message Flow](2_개발수업/Exam3-1_Bitcoin%20protocol/6\)%20P2P%20Network%20%26%20Message%20Flow/P2P%20Network%20%26%20Message%20Flow.md) | P2P 네트워크 · 메시지 흐름 |
| [7) Mempool & Block Assembly](2_개발수업/Exam3-1_Bitcoin%20protocol/7\)%20Mempool%20%26%20Block%20Assembly/Mempool%20%26%20Block%20Assembly.md) | 멤풀 · 블록 조립 |
| [8) Architecture Map & Next Steps](2_개발수업/Exam3-1_Bitcoin%20protocol/8\)%20Architecture%20Map%20%26%20Next%20Steps/Architecture%20Map%20%26%20Next%20Steps.md) | 아키텍처 맵 · 다음 단계 |
| [9) The "Top 10" Source Files](2_개발수업/Exam3-1_Bitcoin%20protocol/9\)%20The%20%22Top%2010%22%20Source%20Files/설명/The%20%22Top%2010%22%20Source%20Files.md) | 핵심 소스파일 TOP 10 심층 분석 |

<details>
<summary>🔍 Top 10 소스파일 세부 분석 보기</summary>

**거시적 흐름 (Macro)**
- `protocol.h` — 메시지 규격/형식 정의, 네트워크 식별 코드(매직 바이트)
- `chainparams.h` — 메인넷/테스트넷/시그넷/레그테스트 매개변수 정의
- `net_processing.cpp` — 수신·발신 메시지 처리 총괄, 피어 연결 관리

**중간 흐름 (Meso)**
- `validation.cpp / validation.h` — 트랜잭션·블록 검증 핵심 로직
- `txmempool.h` — 멤풀 데이터 구조
- `consensus/validation` — 합의 규칙
- `policy/policy.h` — 정책 필터

**미시적 요소 (Micro)**
- `primitives/transaction.h` — 트랜잭션 기본 구조
- `primitives/block.h` — 블록 기본 구조
- `script/script.h` — 스크립트 언어 정의
</details>

---

### 🌐 Exam 3-2 · Internet Infrastructure — 인터넷 인프라
> **핵심 질문:** 비트코인 P2P 네트워크가 동작하는 인터넷은 물리적으로 어떻게 구성되어 있는가?

| 챕터 | 주요 내용 |
|------|-----------|
| [1) The Origins](2_개발수업/Exam3-2_Internet%20Infrastructure/1\)The%20Origins/The%20Origins.md) | ARPANET · TCP/IP · 회선교환 vs 패킷교환 방식 |
| [2) Networking Fundamentals](2_개발수업/Exam3-2_Internet%20Infrastructure/2\)%20Networking%20Fundamentals/Networking%20Fundamentals.md) | 허브·스위치·라우터 · 정적/동적 라우팅 · BGP · ASN |
| [3) Deep Dive](2_개발수업/Exam3-2_Internet%20Infrastructure/3\)%20Deep%20Dive/Deep%20Dive.md) | 심층 분석 |
| [4) Trust & Security](2_개발수업/Exam3-2_Internet%20Infrastructure/4\)%20Trust%20%26%20Security/Trust%20%26%20Security.md) | 신뢰 모델과 보안 |
| [5) Implementation](2_개발수업/Exam3-2_Internet%20Infrastructure/5\)%20Implementation/Implementation.md) | 구현 사례 |

<details>
<summary>🔍 네트워킹 핵심 개념 노트 목록</summary>

- [허브(Hub)](2_개발수업/Exam3-2_Internet%20Infrastructure/2\)%20Networking%20Fundamentals/허브(Hub).md) / [스위치(Switch)](2_개발수업/Exam3-2_Internet%20Infrastructure/2\)%20Networking%20Fundamentals/스위치(Switch).md) / [라우터(Router)](2_개발수업/Exam3-2_Internet%20Infrastructure/2\)%20Networking%20Fundamentals/라우터(Router).md)
- [정적 라우팅](2_개발수업/Exam3-2_Internet%20Infrastructure/2\)%20Networking%20Fundamentals/정적(Static)%20라우팅.md) / [동적 라우팅](2_개발수업/Exam3-2_Internet%20Infrastructure/2\)%20Networking%20Fundamentals/동적(Dynamic)%20라우팅.md)
- [BGP 작동 원리](2_개발수업/Exam3-2_Internet%20Infrastructure/2\)%20Networking%20Fundamentals/BGP의%20작동%20원리.md) · [경로 선택(AS-PATH)](2_개발수업/Exam3-2_Internet%20Infrastructure/2\)%20Networking%20Fundamentals/BGP-경로%20선택%20(AS-PATH).md) · [신뢰 기반 모델](2_개발수업/Exam3-2_Internet%20Infrastructure/2\)%20Networking%20Fundamentals/BGP-신뢰%20기반%20모델.md)
- [ASN 체계](2_개발수업/Exam3-2_Internet%20Infrastructure/2\)%20Networking%20Fundamentals/ASN의%20체계.md) / [ASN 발급 및 관리](2_개발수업/Exam3-2_Internet%20Infrastructure/2\)%20Networking%20Fundamentals/ASN-발급%20및%20관리.md)
</details>

---

### ⚡ Exam 3-3 · Lightning Protocol — 라이트닝 프로토콜

| 챕터 | 주요 내용 |
|------|-----------|
| [1) Why Lightning](2_개발수업/Exam3-3_Lightning%20Protocol/1\)%20Why%20Lightning/Why%20Lightning.md) | 확장성(Scalability) 문제와 라이트닝의 탄생 배경 |
| [2) The Basics of Lightning](2_개발수업/Exam3-3_Lightning%20Protocol/2\)%20The%20Basics%20of%20Lightning/The%20Basics%20of%20Lightning.md) | HTLC · 프리이미지 · 어니언 라우팅 · 결제 흐름 |
| [3) The BOLTs](2_개발수업/Exam3-3_Lightning%20Protocol/3\)%20The%20BOLT's/The%20BOLT's.md) | BOLT 3/4/11 · 합의 규칙 · 기술 표준 |
| [4) Liquidity](2_개발수업/Exam3-3_Lightning%20Protocol/4\)%20Liquidity/Liquidity.md) | 인바운드/아웃바운드 유동성 · 서브마린 스왑 |
| [5) Lightning UX](2_개발수업/Exam3-3_Lightning%20Protocol/5\)%20Lightning%20UX/Lightning%20UX.md) | 사용자 경험 설계 |
| [6) Assets on Lightning](2_개발수업/Exam3-3_Lightning%20Protocol/6\)%20Assets%20on%20Lightning/Assets%20on%20Lightning.md) | 라이트닝 위의 자산 |
| [7) L402's](2_개발수업/Exam3-3_Lightning%20Protocol/7\)%20L402's/L402's.md) | 기계 간 결제(M2M) · L402 프로토콜 |

---

### 🔄 Exam 4-1 · Swap Services — 볼츠(Boltz) 스왑 서비스
> **핵심 질문:** 비트코인의 서로 다른 레이어(온체인 / 라이트닝 / 리퀴드)를 신뢰 없이 어떻게 연결하는가?

| 챕터 | 주요 내용 |
|------|-----------|
| [1) Boltz — Connecting Bitcoin Layers](2_개발수업/Exam4-1_Swap%20Services/1\)%20Boltz-Connecting%20Bitcoin%20Layers/Boltz-Connecting%20Bitcoin%20Layers.md) | 볼츠 서비스 개요 |
| [2) What](2_개발수업/Exam4-1_Swap%20Services/2\)%20What/What.md) | 라이트닝 · 리퀴드 · 루트스탁 · 아크 비교 |
| [3) How — HTLCs](2_개발수업/Exam4-1_Swap%20Services/3\)%20How%20-%20HTLCs/How%20-%20HTLCs.md) | 해시 락 · 타임 락 · HTLC 작동 원리 |
| [4) Technical Details](2_개발수업/Exam4-1_Swap%20Services/4\)%20Technical%20Details/Technical%20Details.md) | 비트코인 스크립트 · 탭루트(Taproot) · MuSig2 |
| [5) Why & Advantages](2_개발수업/Exam4-1_Swap%20Services/5\)%20Why%20%26%20Advantages/Why%20%26%20Advantages.md) | 쉬운 통합 · 속도 · 유연성 |
| [6) Wallets & Demo](2_개발수업/Exam4-1_Swap%20Services/6\)%20Wallets%20%26%20Demo/Wallets%20%26%20Demo.md) | 지갑 연동 및 데모 |

<details>
<summary>🔍 레이어별 비교 노트</summary>

- [라이트닝 네트워크](2_개발수업/Exam4-1_Swap%20Services/2\)%20What/라이트닝%20네트워크%20(Lightning%20Network).md) — 즉시 결제, 채널 관리
- [리퀴드(Liquid)](2_개발수업/Exam4-1_Swap%20Services/2\)%20What/리퀴드%20(Liquid).md) — 사이드체인, 연합 신뢰 모델
- [루트스탁(Rootstock)](2_개발수업/Exam4-1_Swap%20Services/2\)%20What/루트스탁%20(Rootstock).md) — EVM 호환 사이드체인
- [아크(Ark)](2_개발수업/Exam4-1_Swap%20Services/2\)%20What/아크%20(Arkade).md) — 신뢰 최소화 오프체인 프로토콜
</details>

---

### 💧 Exam 4-2 · Liquid SDK — 리퀴드 SDK

| 챕터 | 주요 내용 |
|------|-----------|
| [1-1) About Me](2_개발수업/Exam4-2_Liquid%20SDK/1_Overview/1-1_About%20me/1-1_About%20me.md) | 발표자 소개 |
| [1-2) Company, Goals, Products](2_개발수업/Exam4-2_Liquid%20SDK/1_Overview/1-2_Company%2C%20goals%2C%20products/1-2_Company%2C%20goals%2C%20products.md) | 회사 목표 및 제품 |
| [1-3) Lightning as Interoperability Layer](2_개발수업/Exam4-2_Liquid%20SDK/1_Overview/1-3_Lightning%20as%20Interoperability%20Layer/1-3_Lightning%20as%20Interoperability%20Layer.md) | 라이트닝 상호운용성 레이어 |
| [1-4) SDK — Why Bother](2_개발수업/Exam4-2_Liquid%20SDK/1_Overview/1-4_SDK%20-%20Why%20bother/1-4_SDK%20-%20Why%20bother.md) | SDK가 필요한 이유 |
| [2-1) Common Traits](2_개발수업/Exam4-2_Liquid%20SDK/2_Basics%20and%20Features/2-1_Common%20traits/2-1_Common%20traits.md) | 공통 특성 |
| [2-2) Quality-of-Life Features](2_개발수업/Exam4-2_Liquid%20SDK/2_Basics%20and%20Features/2-2_Quality-of-Life%20Features/2-2_Quality-of-Life%20Features.md) | UX 개선 기능 |
| [2-3) Interface Overview](2_개발수업/Exam4-2_Liquid%20SDK/2_Basics%20and%20Features/2-3_Interface%20Overview/2-3_Interface%20Overview.md) | 인터페이스 개요 |
| [2-4) Try It Out](2_개발수업/Exam4-2_Liquid%20SDK/2_Basics%20and%20Features/2-4_Try%20It%20Out/2-4_Try%20It%20Out.md) | 직접 해보기 |

---

### 🎨 Exam 4-3 · RGB Protocol — RGB 프로토콜
> **핵심 질문:** 이더리움·솔라나 같은 알트코인 없이, 비트코인 기반의 자산 발행 대안을 만들 수 있을까?

| 챕터 | 주요 내용 |
|------|-----------|
| [1) Problems with Altcoins](2_개발수업/Exam4-3_RGB/1_Problems%20with%20Altcoins/1_Problems%20with%20Altcoins.md) | 알트코인의 구조적 문제 (확장성 한계, 중앙화) |
| [2) Why the Market Likes Altcoins](2_개발수업/Exam4-3_RGB/2_Why%20the%20market%20likes%20altcoins/2_Why%20the%20market%20likes%20altcoins.md) | DeFi 생태계, 토큰 인플레이션 마케팅 |
| [3) Can We Have a Bitcoin-Based Alternative](2_개발수업/Exam4-3_RGB/3_Can%20we%20have%20a%20bitcoin-based%20alternative%20%26%20What%20we%20do%20not%20want/3_Can%20we%20have%20a%20bitcoin-based%20alternative%20%26%20What%20we%20do%20not%20want.md) | 소수 검증자(PoS) 금지 · 특정 연합 신뢰 금지 |
| [4) RGB Solution — Client Side Validation](2_개발수업/Exam4-3_RGB/4_RGB%20Solution%20-%20Client%20Side%20Validation/4_RGB%20Solution%20-%20Client%20Side%20Validation.md) | 클라이언트 측 검증 · UTXO 연동 |
| [5) Lightning Network Compatibility](2_개발수업/Exam4-3_RGB/5_Lightning%20Network%20Compatibility/5_Lightning%20Network%20Compatibility.md) | 라이트닝 네트워크 호환성 |
| [6) DEX on Lightning](2_개발수업/Exam4-3_RGB/6_DEX%20on%20Lightning/6_DEX%20on%20Lightning.md) | 라이트닝 위의 탈중앙 거래소 |
| [7) Bitfinex's RGB Team & Iris Wallet](2_개발수업/Exam4-3_RGB/7_Bitfinex's%20RGB%20Team%20%26%20Iris%20Wallet%20Android/7_Bitfinex's%20RGB%20Team%20%26%20Iris%20Wallet%20Android.md) | Bitfinex RGB팀 · Iris Wallet |

---

### 💵 Exam 4-4 · e-cash
> [궁극적인 질문](2_개발수업/Exam4-4_e-cash/궁극적인%20질문_e-cash.md) — Chaumian e-cash와 비트코인의 결합

### 🏦 Exam 4-5 · Ark Protocol
> [궁극적인 질문](2_개발수업/Exam4-5_Ark/궁극적인%20질문_Ark.md) — Ark 프로토콜의 핵심 개념

---

## 3️⃣ 기업과제 — Ark Protocol VTXO 검증 구현

> **비트코인 Ark 프로토콜에서 서버(ASP)를 신뢰하지 않고 클라이언트 측에서 VTXO 체인을 직접 검증하는 코드를 처음부터 구현한 실전 과제**

### 📋 과제 개요
| 티어 | 내용 |
|------|------|
| **Tier 1** (필수) | VTXO 체인 재구성·검증 / 모든 서명 검증(MuSig2, Schnorr) / 온체인 앵커링 확인 |
| **Tier 2** (추가) | 탭루트 스크립트 트리 분석·검증 / 타임락 검증 / 해시 프리이미지 조건 검증 |
| **Tier 3** (고급) | 일방적 종료(Unilateral Exit) 데이터 저장 · 실행 함수 구현 |

### 📂 폴더별 내용
| 폴더 | 내용 |
|------|------|
| [개념정리](기업과제/개념정리/) | VTXO · ASP · 탭루트 스크립트 · 서명검증 핵심 개념 |
| [과제 설명](기업과제/과제%20설명/) | 과제 요구사항 상세 분석 |
| [쌩초짜의 실전 머리박치기](기업과제/쌩초짜의%20실전%20머리박치기/) | Node.js 환경 세팅 · 코드 작성 · 에러 해결 과정 |
| [끝나고 나서](기업과제/끝나고%20나서/) | 아크서버 가상트리 구조 · 비상탈출 규칙 · 과제 회고 |
| [보고서 작성하기](기업과제/보고서%20작성하기/) | 최종 보고서 |

<details>
<summary>🔍 핵심 개념 노트 목록</summary>

- [VTXO란](기업과제/개념정리/트랜잭션,%20장부%20그리고%20vtxo.md) — 가상 UTXO 개념
- [전체 흐름 설명](기업과제/개념정리/전체%20흐름%20설명.md) — VTXO 받는 전체 과정
- [VTXO 족보 재구성 및 확인](기업과제/개념정리/VTXO%20족보%20재구성%20및%20확인%20(Reconstruct%20and%20validate%20the%20VTXO%20chain).md)
- [탭루트 스크립트](기업과제/개념정리/탭루트%20스크립트.md)
- [신뢰하지 말고 검증하라](기업과제/개념정리/신뢰하지%20말고%20검증하라.md)
- [서명검증 코드](기업과제/쌩초짜의%20실전%20머리박치기/내가%20짠%20코드_설명/서명검증_코드.md) / [코드 설명](기업과제/쌩초짜의%20실전%20머리박치기/내가%20짠%20코드_설명/서명검증_코드_설명.md)
- [verifyVtxo 함수 완전 해설](기업과제/쌩초짜의%20실전%20머리박치기/내가%20짠%20코드_설명/최종코드설명%20폴더/verifyVtxo%20함수%20완전%20해설.md)
- [아크서버 비상탈출](기업과제/끝나고%20나서/아크서버_비상탈출.md) / [존재증명방법](기업과제/끝나고%20나서/아크서버_비상탈출규칙_존재증명방법.md)
</details>

---

## 📌 보충 개념 노트

### 구멍난 개념 다시 채우기
- [UTXO란](구멍난%20개념%20다시%20채우기/UTXO란.md) — UTXO(Unspent Transaction Output) 개념 정리

---

## 🗺️ 인덱스 / 마인드맵

옵시디언 Canvas로 제작된 주제별 마인드맵 파일 (`index/` 폴더)

| 파일 | 주제 |
|------|------|
| `Map.canvas` | 전체 개요 맵 |
| `Map_WhyBitcoin.canvas` | Why Bitcoin |
| `Map_HowBitocin.canvas` | How Bitcoin |
| `Map_WhereBitcoin.canvas` | Where Bitcoin |
| `Map_Bitcoin Industry Landscape.canvas` | 산업 지형도 |
| `Map_Bitcoin Protocol.canvas` | 비트코인 프로토콜 |
| `Lightning Protocol - Map.canvas` | 라이트닝 프로토콜 |
| `Map_Swap Services.canvas` | 스왑 서비스 |
| `Map_Liquid SDK.canvas` | 리퀴드 SDK |
| `Map-RGB.canvas` | RGB 프로토콜 |
| `Map_Ark.canvas` | Ark 프로토콜 |
| `Map_e-cash.canvas` | e-cash |

---

## 🔑 주요 키워드

`Bitcoin` `Lightning Network` `UTXO` `HTLC` `Taproot` `MuSig2` `Schnorr` `SegWit`  
`Blockchain` `Proof of Work` `Merkle Tree` `P2P Network` `BGP` `TCP/IP`  
`Liquid` `RGB` `Ark` `Rootstock` `e-cash` `Submarine Swap` `Boltz`  
`VTXO` `Client-Side Validation` `Unilateral Exit` `Onion Routing` `BOLT`

---

## 📝 노트 작성 환경

- **노트 도구:** [Obsidian](https://obsidian.md/)
- **버전 관리:** Git / GitHub
- **학습 과정:** PlanB Network Bitcoin 개발자 교육 과정

---

*이 저장소는 학습 목적으로 작성된 개인 노트입니다.*
