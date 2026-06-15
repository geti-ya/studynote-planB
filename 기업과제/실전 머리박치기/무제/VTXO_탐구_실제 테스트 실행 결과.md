# VTXO 탐구 — 실제 테스트 실행 결과

처음으로 서버에서 VTXO를 받아서 내부 데이터를 직접 찍어본 테스트 결과.

## 실행 명령

```bash
pnpm test test/e2e/my-verify.test.ts
```

## 테스트 출력

```
✅ 지갑(클라이언트) 생성 완료!
👉 내 지갑 주소: tark1qpt0syx7j0jspe69kldtljet0x9jz6ns4xw70m0w0xl30yfhn0mzmj4t5z665vg8ejagh65v5zhvdxvee0ucun0mqlc06hy6m8eu9g8z28pwfr
⏳ 서버에 VTXO 입금 요청 중...
✅ VTXO 입금 완료!

✓ test/e2e/my-verify.test.ts (1 test) 6086ms
   ✓ VTXO 훔쳐보기 작전 (1)
     ✓ 서버에서 VTXO 받아서 로그 찍기  6085ms

 Test Files  1 passed (1)
      Tests  1 passed (1)
   Start at  22:23:18
   Duration  7.17s
```

## 확보한 VTXO 데이터 구조

```json
{
  "txid": "4b86f1b9590c11a080a05055146e0656d6efc6fddb87021d4e103e2d3c20c419",
  "vout": 0,
  "value": 5000,
  "status": {
    "confirmed": false,
    "isLeaf": false
  },
  "virtualStatus": {
    "state": "preconfirmed",
    "commitmentTxIds": [
      "15f20180ef54e693da414326463a7b251c1e3398a0d0ec44a4ec89e7adaea674"
    ],
    "batchExpiry": 1776433229000
  },
  "spentBy": "",
  "settledBy": "",
  "arkTxId": "",
  "createdAt": "2026-04-17T13:23:25.000Z",
  "isUnrolled": false,
  "isSpent": false,
  "script": "5120caaba0b5aa3107ccba8bea8ca0aec69999cbf98e4dfb07f0fd5c9ad9f3c2a0e2",
  "forfeitTapLeafScript": [...],
  "intentTapLeafScript": [...],
  "tapTree": {...}
}
```

## 핵심 필드 해석

| 필드 | 값 | 의미 |
|---|---|---|
| `txid` | `4b86f1...` | 이 VTXO의 고유 식별자 |
| `vout` | `0` | 트랜잭션의 0번째 출력 |
| `value` | `5000` | 5000 sats |
| `virtualStatus.state` | `"preconfirmed"` | 아직 온체인 미확정, ASP가 서명으로 보증 중 |
| `virtualStatus.commitmentTxIds` | `["15f201..."]` | 연결된 커밋먼트 트랜잭션 ID |
| `virtualStatus.batchExpiry` | `1776433229000` | 이 VTXO가 만료되는 시각 (Unix ms) |
| `isSpent` | `false` | 아직 사용되지 않음 |
| `forfeitTapLeafScript` | (바이트 배열) | 포기(forfeit) 경로의 탭스크립트 |
| `intentTapLeafScript` | (바이트 배열) | 의도(intent) 경로의 탭스크립트 |
| `tapTree` | (바이트 배열) | 전체 탭루트 트리 직렬화 데이터 |

`preconfirmed` 상태란: 온체인에는 아직 없지만 ASP가 서명해서 "이건 진짜야"라고 보증한 상태. `commitmentTxIds`가 있으므로 연결된 커밋먼트 트랜잭션으로 온체인 확인이 가능하다.
