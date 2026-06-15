# VTXO 족보 검증 및 Zero Trust 원칙

## 족보 재구성 — DAG를 어떻게 추적하는가

서버가 준 가상 코인(VTXO)의 가장 끝단(잎)부터 시작해서, 진짜 비트코인 덩어리(배치 아웃풋, 뿌리)까지 거슬러 올라가는 전체 거래 족보(DAG)를 재구성해야 한다.

족보의 구조:
- **끝단 (Leaf):** 내가 방금 받은 최신 VTXO
- **중간 다리 (Arkoor):** 오프체인에서 유저들끼리 코인을 주고받은 중간 거래 내역들
- **뿌리 (Root):** 실제 비트코인 블록체인에 단단히 박혀 있는 배치 아웃풋(Commitment transaction)

### 검증 로직 (반복문으로 구현)

`while` 이나 `for` 루프를 돌면서 아래 검사를 수행:

1. **시작:** 서버가 준 VTXO(잎) 데이터에서 `inputs`(이 돈이 어디서 왔는지) 정보를 확인
2. **부모 찾기:** 그 `inputs`가 가리키는 바로 윗세대 부모 트랜잭션을 가져옴
3. **연결 고리 확인 (핵심!):** 내 VTXO의 입력값이 진짜로 부모 트랜잭션의 결과물(outputs)을 정확히 가리키고 있는지 검사 (TXID와 Index 일치 여부)
4. **체크포인트 검사:** 족보를 거슬러 올라가다가 체크포인트 트랜잭션이 있다면, 이 구조가 sweep delay 설정과 모순 없이 DAG에 통합되어 있는지 검증
5. **반복:** 뿌리(Batch Output)에 도달할 때까지 계속 반복. 한 군데라도 끊겨있거나 숫자가 안 맞으면 `false`를 반환

---

## Zero Trust — 신뢰하지 말고 검증하라

ASP가 "이 VTXO는 valid해요" 주장한다고 해서 그냥 믿으면 안 된다.

### 나쁜 접근
```typescript
const vtxo = await asp.getVTXO(id);
// ASP 말 그대로 믿음
return vtxo;
```

### 좋은 접근
```typescript
const vtxo = await asp.getVTXO(id);

// 1. 정말 valid한가? 직접 확인
if (!verifySignatures(vtxo)) {
  throw new Error("서명 무효!");
}

// 2. 정말 체인이 연결되나? 직접 확인
if (!verifyChain(vtxo)) {
  throw new Error("체인 끊김!");
}

// 3. 정말 온체인에 있나? 직접 확인
if (!await verifyOnchain(vtxo)) {
  throw new Error("블록체인에 없음!");
}

return vtxo; // 이제 믿을 수 있음
```
