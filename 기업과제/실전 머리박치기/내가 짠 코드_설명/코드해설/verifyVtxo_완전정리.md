# verifyVtxo — 메인 검증 함수 완전 정리

## 코드 원본

```typescript
export async function verifyVtxo(
    vtxo: ExtendedVirtualCoin,
    vtxoChain: VtxoChain,
    options?: {
        vtxoTreeNodes?: TxTreeNode[];
        commitmentTxHex?: string;
        sweepTapTreeRoot?: Uint8Array;
        minConfirmations?: number;
    }
): Promise<VerificationResult> {

    const errors: string[] = [];
    const checks = {
        chainReconstructed: false,
        chainLinked: false,
        signaturesValid: false,
        onchainConfirmed: false,
        notDoubleSpent: false,
    };

    const commitmentTxid = (vtxo as any).virtualStatus?.commitmentTxIds?.[0] ?? null;

    // Check 1: 체인 재구성
    if (vtxoChain.chain.length > 0) {
        checks.chainReconstructed = true;
    } else {
        errors.push("VTXO 체인 데이터가 비어있음");
    }

    // Check 2: 체인 연결 검증
    const vtxoTxid = vtxo.txid;
    const linkResult = verifyChainLinking(vtxoChain.chain, vtxoTxid);
    if (linkResult.ok) {
        checks.chainLinked = true;
    } else {
        errors.push(`체인 연결 오류: ${linkResult.error}`);
    }

    // Check 3: 서명 검증 (tree 데이터가 있을 때만)
    if (
        options?.vtxoTreeNodes &&
        options?.commitmentTxHex &&
        options?.sweepTapTreeRoot
    ) {
        const sigResult = verifyVtxoTreeSignatures(
            options.vtxoTreeNodes,
            options.commitmentTxHex,
            options.sweepTapTreeRoot
        );
        if (sigResult.ok) {
            checks.signaturesValid = true;
        } else {
            errors.push(`서명 검증 오류: ${sigResult.error}`);
        }
    } else {
        if (vtxo.virtualStatus?.state === "preconfirmed") {
            checks.signaturesValid = true; // preconfirmed는 ASP 서명으로 보장
        } else {
            errors.push("서명 검증 불가: vtxoTreeNodes 또는 commitmentTxHex 없음");
        }
    }

    // Check 4 & 5: 온체인 확인 + double-spend 체크
    if (commitmentTxid) {
        const onchainResult = await verifyOnchainAnchoring(
            commitmentTxid,
            0,
            undefined,
            options?.minConfirmations ?? 1
        );

        if (onchainResult.ok) {
            checks.onchainConfirmed = true;
            checks.notDoubleSpent = !vtxo.isSpent;
            if (vtxo.isSpent) {
                errors.push(`VTXO가 이미 소비됨: spentBy=${vtxo.spentBy}`);
            }
            if (vtxo.virtualStatus?.state === "swept") {
                errors.push("VTXO가 만료되어 sweep 되었습니다");
            }
        } else {
            errors.push(`온체인 확인 실패: ${onchainResult.error}`);
        }

        return {
            success: Object.values(checks).every(Boolean) && errors.length === 0,
            checks,
            errors,
            details: {
                chainLength: vtxoChain.chain.length,
                commitmentTxid,
                confirmations: (await verifyOnchainAnchoring(commitmentTxid)).confirmations,
                vtxoOutpoint: `${vtxo.txid}:${vtxo.vout}`,
                isPreconfirmed: vtxo.virtualStatus?.state === "preconfirmed",
            },
        };
    } else {
        // preconfirmed: 아직 commitment tx 없음
        checks.onchainConfirmed = vtxo.virtualStatus?.state === "preconfirmed";
        checks.notDoubleSpent = !vtxo.isSpent;

        if (vtxo.virtualStatus?.state !== "preconfirmed") {
            errors.push("commitment txid 없고 preconfirmed도 아님 — 유효하지 않은 VTXO");
        }

        return {
            success: Object.values(checks).every(Boolean) && errors.length === 0,
            checks,
            errors,
            details: {
                chainLength: vtxoChain.chain.length,
                commitmentTxid: null,
                confirmations: null,
                vtxoOutpoint: `${vtxo.txid}:${vtxo.vout}`,
            },
        };
    }
}
```

---

## 함수 설명

VTXO를 검증하는 메인 함수. `async` 함수 (온체인 조회가 필요하기 때문에 비동기).

### 검증 항목 5가지

| 항목 | 설명 |
|---|---|
| `chainReconstructed` | 체인 데이터가 존재하는가 |
| `chainLinked` | 체인이 올바르게 연결되어 있는가 |
| `signaturesValid` | 서명이 유효한가 |
| `onchainConfirmed` | 온체인에서 확인됐는가 |
| `notDoubleSpent` | 이중 지불이 없는가 |

모두 처음엔 `false`로 시작하고, 검증을 통과하면 `true`로 바뀐다.

### Check 3 — 서명 검증 두 가지 경로

| 상황 | 동작 |
|---|---|
| 트리 데이터 있음 | 직접 서명 검증 수행 |
| `preconfirmed` 상태 | ASP가 이미 서명 보장 → 통과 |
| 그 외 | 에러 |

### Check 4 & 5 — 온체인 확인 + 이중지불

**경우 A: commitmentTxid가 있을 때 (정상 확정된 VTXO)**
- 비트코인 블록체인에서 실제로 해당 트랜잭션이 확인됐는지 조회
- VTXO가 이미 사용됐는지(이중지불) 확인

**경우 B: commitmentTxid가 없을 때 (preconfirmed 상태)**
- `preconfirmed`면 허용, 아니면 유효하지 않은 VTXO로 처리

### 전체 흐름

```
verifyVtxo 호출
    │
    ├─ [Check 1] 체인 데이터 있나?
    ├─ [Check 2] 체인이 올바르게 연결됐나?
    ├─ [Check 3] 서명 유효한가? (or preconfirmed인가?)
    │
    ├─ commitmentTxid 있음? ──Yes──▶ 온체인 실제 조회
    │                                 └─ [Check 4] 온체인 확인
    │                                 └─ [Check 5] 이중지불 없음?
    │
    └─ commitmentTxid 없음? ──▶ preconfirmed인지 확인
                                 └─ preconfirmed면 조건부 통과
```

5개 체크가 모두 통과해야 `success: true`가 된다.
