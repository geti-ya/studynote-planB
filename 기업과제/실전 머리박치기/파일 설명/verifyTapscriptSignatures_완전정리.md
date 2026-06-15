# verifyTapscriptSignatures — 완전 정리

## 코드 원본

```typescript
export function verifyTapscriptSignatures(
    tx: Transaction,
    inputIndex: number,
    requiredSigners: string[],
    excludePubkeys: string[] = [],
    allowedSighashTypes: number[] = [SigHash.DEFAULT]
): void {
    const input = tx.getInput(inputIndex);

    // 모든 입력의 UTXO 정보 수집 (preimageWitnessV1에 필요)
    const prevoutScripts: Uint8Array[] = [];
    const prevoutAmounts: bigint[] = [];

    for (let i = 0; i < tx.inputsLength; i++) {
        const inp = tx.getInput(i);
        if (!inp.witnessUtxo) {
            throw new Error(`Input ${i} is missing witnessUtxo`);
        }
        prevoutScripts.push(inp.witnessUtxo.script);
        prevoutAmounts.push(inp.witnessUtxo.amount);
    }

    // tapScriptSig 존재 확인
    if (!input.tapScriptSig || input.tapScriptSig.length === 0) {
        throw new Error(`Input ${inputIndex} is missing tapScriptSig`);
    }

    // 각 서명 검증
    for (const [tapScriptSigData, signature] of input.tapScriptSig) {
        const pubKey = tapScriptSigData.pubKey;
        const pubKeyHex = hex.encode(pubKey);

        // 제외 목록이면 건너뜀
        if (excludePubkeys.includes(pubKeyHex)) continue;

        // 서명 해시 타입 추출 (Schnorr: 64바이트, 65바이트면 마지막이 sighash 타입)
        const sighashType = signature.length === 65 ? signature[64] : SigHash.DEFAULT;
        const sig = signature.subarray(0, 64);

        if (!allowedSighashTypes.includes(sighashType)) {
            throw new Error(`Unallowed sighash type for input ${inputIndex}, pubkey ${pubKeyHex}.`);
        }

        // 서명에 해당하는 tapscript 리프 찾기
        const leafHash = tapScriptSigData.leafHash;
        const leafHashHex = hex.encode(leafHash);
        let matchingScript: Uint8Array | undefined;
        let matchingVersion: number | undefined;

        for (const [_, scriptWithVersion] of input.tapLeafScript) {
            const script = scriptWithVersion.subarray(0, -1);
            const version = scriptWithVersion[scriptWithVersion.length - 1];
            const computedLeafHash = tapLeafHash(script, version);
            if (hex.encode(computedLeafHash) === leafHashHex) {
                matchingScript = script;
                matchingVersion = version;
                break;
            }
        }

        if (!matchingScript || matchingVersion === undefined) {
            throw new Error(`No tapLeafScript found matching leafHash ${hex.encode(leafHash)}`);
        }

        // 서명 메시지 재구성 후 Schnorr 검증
        const message = tx.preimageWitnessV1(
            inputIndex,
            prevoutScripts,
            sighashType,
            prevoutAmounts,
            undefined,
            matchingScript,
            matchingVersion
        );

        const isValid = schnorr.verify(sig, message, pubKey);
        if (!isValid) {
            throw new Error(`Invalid signature for input ${inputIndex}, pubkey ${pubKeyHex}`);
        }
    }

    // 필수 서명자 전원 서명 확인
    const signedPubkeys = input.tapScriptSig.map(([data]) => hex.encode(data.pubKey));
    const requiredNotExcluded = requiredSigners.filter(pk => !excludePubkeys.includes(pk));
    const missingSigners = requiredNotExcluded.filter(pk => !signedPubkeys.includes(pk));

    if (missingSigners.length > 0) {
        throw new Error(`Missing signatures from: ${missingSigners.map(pk => pk.slice(0, 16)).join(", ")}...`);
    }
}
```

---

## 함수 설명

**비트코인 트랜잭션의 Tapscript 서명을 검증**하는 함수. 반환값이 `void`이므로 통과하면 조용히 끝나고, 실패하면 에러를 던진다.

### 파라미터

| 파라미터 | 의미 |
|---|---|
| `tx` | 검증할 트랜잭션 |
| `inputIndex` | 검증할 입력(input) 번호 |
| `requiredSigners` | 반드시 서명해야 하는 공개키 목록 |
| `excludePubkeys` | 건너뛸 공개키 (아직 서명 안 한 참여자 등) |
| `allowedSighashTypes` | 허용된 서명 해시 타입 |

### 전체 흐름

```
모든 입력의 UTXO 정보 수집
      ↓
tapScriptSig 존재 확인
      ↓
각 서명에 대해:
  ├─ excludePubkeys이면 → 건너뜀
  ├─ sighash 타입 허용됐는지 → 아니면 에러
  ├─ leafHash로 tapscript 리프 찾기 → 없으면 에러
  └─ Schnorr 서명 수학적 검증 → 틀리면 에러
      ↓
필수 서명자 전원 서명했는지 확인 → 빠진 사람 있으면 에러
      ↓
✅ 모두 통과하면 정상 종료 (void)
```

### 핵심: tapscript 리프 찾기

Tapscript는 머클 트리 구조로 여러 스크립트를 담는다. 서명이 어떤 리프에 대한 것인지 `leafHash`로 찾아낸 뒤, 그 스크립트로 메시지를 재구성해 Schnorr 서명을 수학적으로 검증한다.

```
        루트
       /    \
    리프A   리프B   ← leafHash로 해당 리프 찾기
```
