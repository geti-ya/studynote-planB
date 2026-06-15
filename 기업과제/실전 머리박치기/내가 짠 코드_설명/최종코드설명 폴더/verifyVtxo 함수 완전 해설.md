### 🌍 배경: 이 코드는 무엇을 하는가?

이 코드는 **ARK 프로토콜** (비트코인 레이어2)에서 사용하는 **VTXO(Virtual UTXO, 가상 코인)** 를 검증하는 함수입니다.

비트코인의 UTXO(미사용 트랜잭션 출력)처럼, ARK에서는 VTXO라는 가상의 코인 단위를 씁니다. 이 코인이 **진짜 유효한지**, **이미 쓰인 건 아닌지**, **서명은 올바른지** 등을 직접 검증합니다.

---

### 📦 함수 입력값 (파라미터)

typescript

```typescript
verifyVtxo(
    vtxo: ExtendedVirtualCoin,       // 검증할 VTXO (가상 코인)
    vtxoChain: VtxoChain,            // 이 코인의 거래 족보(체인)
    options?: { ... }                // 선택적 옵션들
)
```

|파라미터|의미|
|---|---|
|`vtxo`|내가 가진 가상 코인 1개|
|`vtxoChain`|이 코인이 거쳐온 트랜잭션 족보|
|`vtxoTreeNodes`|트리 구조의 트랜잭션 노드들|
|`commitmentTxHex`|온체인에 올라간 앵커 트랜잭션|
|`sweepTapTreeRoot`|Taproot 스크립트 루트|
|`minConfirmations`|최소 블록 확인 수|
|`indexer`|블록체인 데이터 제공자|

---

### 🔍 5가지 검증 체크리스트

함수 시작부분에 검증 항목들이 정의됩니다:

typescript

```typescript
const checks = {
    chainReconstructed: false,   // ① 족보가 존재하는가?
    chainLinked: false,          // ② 족보가 끊기지 않고 연결되는가?
    signaturesValid: false,      // ③ 서명이 유효한가?
    onchainConfirmed: false,     // ④ 온체인에 기록됐는가?
    notDoubleSpent: false,       // ⑤ 이중지불이 없는가?
};
```

이 5개가 모두 `true`여야 코인이 유효하다고 판단합니다.

---

### ① 체인 재구성 확인 (chainReconstructed)

typescript

```typescript
if (vtxoChain.chain.length > 0) {
    checks.chainReconstructed = true;
} else {
    errors.push("VTXO 체인 데이터가 비어있음");
}
```

**핵심:** 족보 데이터가 아예 없으면 검증 자체가 불가능합니다. 가장 기본적인 확인입니다.

---

### ② 체인 연결 검증 (chainLinked)

typescript

```typescript
const linkResult = verifyChainLinking(vtxoChain.chain, vtxoTxid);
```

**핵심:** 족보(chain)의 각 트랜잭션이 내 코인(`vtxoTxid`)까지 **끊기지 않고 연결**되는지 확인합니다.

> 예: A → B → C → 내코인 이렇게 끊김 없이 이어져야 합니다.

---

### ③ 서명 검증 (signaturesValid) ⭐ 가장 복잡

서명 검증은 **두 가지 경로**로 나뉩니다:

#### 경로 A: 트리 데이터가 있을 때 (간단)

typescript

```typescript
if (options?.vtxoTreeNodes && options?.commitmentTxHex && options?.sweepTapTreeRoot) {
    const sigResult = verifyVtxoTreeSignatures(...);
}
```

트리 데이터가 다 갖춰져 있으면 바로 검증합니다.

---

#### 경로 B: `preconfirmed` 상태일 때 (복잡) 🔥

아직 온체인에 올라가지 않은 VTXO는 indexer(블록체인 데이터 서버)에서 트랜잭션을 직접 가져와서 검증합니다.

**3단계로 진행됩니다:**

**[1단계] 트랜잭션 데이터 수집 및 파싱**

typescript

```typescript
const virtualTxs = await options.indexer.getVirtualTxs(txids);
```

가져온 데이터는 두 가지 형식 중 하나입니다:

typescript

```typescript
if (txData.startsWith("cHNid")) {
    // Base64로 인코딩된 PSBT 형식
    tx = Transaction.fromPSBT(base64.decode(padded));
} else {
    // 일반 Hex 형식
    tx = Transaction.fromRaw(txBytes);
}
```

> `cHNid`는 `psbt`를 Base64로 인코딩하면 항상 나오는 접두사입니다. 일종의 파일 형식 구분자예요.

**[2단계] 부모 트랜잭션 출력값 주입 (Zero Trust 방식)**

typescript

```typescript
const parentOutput = parentTx.getOutput(input.index);
tx.updateInput(i, {
    witnessUtxo: {
        amount: parentOutput.amount,
        script: parentOutput.script
    }
});
```

서버가 주는 금액/스크립트를 그냥 믿지 않고, **직접 부모 트랜잭션을 찾아서 실제 값을 주입**합니다.

**[3단계] Tapscript 서명 검증**

typescript

```typescript
verifyTapscriptSignatures(tx, i, []);
```

주입한 진짜 값으로 서명이 맞는지 검증합니다.

---

### ④⑤ 온체인 확인 + 이중지불 검증

#### 이중지불 탐지 (서버 믿지 않음!)

typescript

```typescript
for (const node of vtxoChain.chain) {
    const spendsArray = Array.isArray(node.spends) ? node.spends : [node.spends];
    if (spendsArray.some((spend: string) => spend.split(":")[0] === vtxo.txid)) {
        isSpentOffchain = true;
        foundSpenderTxid = node.txid;
    }
}
```

서버의 `isSpent` 필드를 믿지 않고, **직접 족보(DAG)를 뒤져서** 내 코인을 소비한 트랜잭션이 있는지 찾습니다.

#### 온체인 앵커 확인

이후 `commitmentTxid`가 있는지 여부에 따라 두 갈래로 나뉩니다:

```
commitmentTxid 있음 → 온체인에 실제 기록됐는지 확인
commitmentTxid 없음 → preconfirmed 상태인지 확인
```

---

### 📤 최종 반환값

typescript

```typescript
return {
    success: Object.values(checks).every(Boolean) && errors.length === 0,
    checks,      // 5가지 체크 결과
    errors,      // 발견된 오류 목록
    details: {
        chainLength,       // 족보 길이
        commitmentTxid,    // 온체인 앵커 txid
        confirmations,     // 블록 확인 수
        vtxoOutpoint,      // 코인 위치 (txid:vout)
    }
};
```

`success`는 **5개 체크가 모두 true이고 에러가 없을 때만** true가 됩니다.

---

### 🗺️ 전체 흐름 요약

```
verifyVtxo 호출
    │
    ├─ [Check 1] vtxoChain.chain.length > 0?
    │            NO → errors: "VTXO 체인 데이터가 비어있음"
    │
    ├─ [Check 2] verifyChainLinking(chain, vtxoTxid)
    │            FAIL → errors: "체인 연결 끊김"
    │
    ├─ [Check 3] vtxoTreeNodes + commitmentTxHex + sweepTapTreeRoot 모두 있나?
    │       YES → verifyVtxoTreeSignatures() 직접 호출
    │       NO  → preconfirmed 상태면 통과 / 아니면 errors
    │
    ├─ commitmentTxid 있음?
    │       │
    │      YES                              NO (preconfirmed)
    │       │                                │
    │  [Check 4] verifyOnchainAnchoring()   [Check 4] state === "preconfirmed"?
    │  [Check 5] vtxo.isSpent === false     [Check 5] vtxo.isSpent === false
    │       │                                │
    │       └────────────────────────────────┘
    │
    └─ return { success, checks, errors, details }

success = checks 5개 모두 true && errors 없음
```

---

### 🔑 핵심 설계 철학: "서버를 믿지 마라"

이 코드 전체를 관통하는 가장 중요한 원칙이 있습니다.

> **"서버가 주는 상태값(isSpent, state 등)을 절대 그냥 믿지 않는다."**

|항목|서버 신뢰 방식 (위험)|이 코드의 방식 (안전)|
|---|---|---|
|이중지불 여부|`vtxo.isSpent` 그대로 사용|DAG 족보를 직접 탐색|
|트랜잭션 금액/스크립트|서버가 준 값 사용|부모 트랜잭션에서 직접 추출|
|온체인 확인|서버 상태 믿음|`verifyOnchainAnchoring()` 직접 호출|

이 원칙을 **Zero Trust(제로 트러스트)** 방식이라고 코드 주석에서 표현하고 있습니다. 비트코인처럼 "검증하지 말고 직접 확인하라 (Don't Trust, Verify)"는 정신을 구현한 것입니다.

---
