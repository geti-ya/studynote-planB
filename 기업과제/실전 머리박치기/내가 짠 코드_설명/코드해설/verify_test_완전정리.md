# verify.test.ts — 테스트 파일 완전 정리

## 코드 원본

```typescript
/**
 * verify.test.ts
 * 실행: pnpm vitest run test/e2e/vtxo-verify/verify.test.ts
 *
 * fixture.json을 로드해서 서버 없이 로컬에서 직접 VTXO 검증
 */

import { describe, it, expect, beforeAll } from "vitest";
import { readFileSync } from "fs";
import { join, dirname } from "path";
import { fileURLToPath } from "url";
import { RestIndexerProvider } from "../../../src";
import { RestArkProvider, IndexerProvider } from "../../../src";
import { verifyVtxo, verifyChainLinking, verifyOnchainAnchoring } from "./verifier";
import { hex } from "@scure/base";

const __dirname = dirname(fileURLToPath(import.meta.url));

// ─── fixture 로드 ─────────────────────────────────────────────────
let fixture: any;
let sweepTapTreeRoot: Uint8Array;
let indexerProvider: RestIndexerProvider;

beforeAll(async () => {
    // fixture 로드
    const fixturePath = join(__dirname, "fixture.json");
    try {
        fixture = JSON.parse(readFileSync(fixturePath, "utf-8"));
        console.log(`\n📦 fixture 로드: ${fixturePath}`);
        console.log(`   생성 시각: ${fixture.createdAt}`);
        console.log(`   시나리오: ${fixture.scenario}`);
    } catch (err) {
        throw new Error(
            "fixture.json 없음! 먼저 setup.ts를 실행하세요:\n" +
            "  npx tsx test/e2e/vtxo-verify/setup.ts"
        );
    }

    const arkProvider = new RestArkProvider("http://localhost:7070");
    const arkInfo = await arkProvider.getInfo();
    sweepTapTreeRoot = hex.decode(arkInfo.signerPubkey);

    indexerProvider = new RestIndexerProvider("http://localhost:7070");
});

// ─── 테스트 ────────────────────────────────────────────────────────
describe("VTXO 클라이언트 사이드 검증 (Tier 1)", () => {

    it("fixture 데이터 구조 확인", () => {
        expect(fixture).toBeDefined();
        expect(fixture.targetVtxo).toBeDefined();
        expect(fixture.vtxoChain).toBeDefined();
        expect(fixture.vtxoChain.chain).toBeInstanceOf(Array);

        const vtxo = fixture.targetVtxo;
        console.log(`\n📋 검증 대상 VTXO:`);
        console.log(`   outpoint : ${vtxo.txid}:${vtxo.vout}`);
        console.log(`   금액     : ${vtxo.value} sats`);
        console.log(`   상태     : preconfirmed=${vtxo.virtualStatus?.state === "preconfirmed"}, spent=${vtxo.isSpent}`);
        console.log(`   만료일   : ${vtxo.virtualStatus?.batchExpiry ?? "(없음)"}`);
        console.log(`   체인 길이: ${fixture.vtxoChain.chain.length}개 tx`);
    });

    it("[Step 1] VTXO 체인 연결 검증 — 서버 안 믿고 직접 확인", () => {
        const { targetVtxo, vtxoChain } = fixture;
        expect(vtxoChain.chain.length).toBeGreaterThan(0);

        const targetTxid = targetVtxo.outpoint?.txid || (targetVtxo as any).txid;
        const result = verifyChainLinking(vtxoChain.chain, targetTxid);

        vtxoChain.chain.forEach((tx: any, i: number) => {
            console.log(`   [${i}] ${tx.type.padEnd(35)} ${tx.txid.slice(0, 16)}...`);
            if (tx.spends?.length) {
                console.log(`       spends: ${tx.spends.map((s: string) => s.slice(0, 12) + "...").join(", ")}`);
            }
        });

        if (!result.ok) {
            console.warn(`⚠️  체인 연결 검증 경고: ${result.error}`);
        }

        if (targetVtxo.virtualStatus?.state !== "preconfirmed") {
            expect(result.ok).toBe(true);
        }
    });

    it("[Step 2] VTXO 소비 상태 검증 — 이미 쓴 VTXO면 무효", () => {
        const { targetVtxo } = fixture;
        expect(targetVtxo.isSpent).toBe(false);
        expect(targetVtxo.virtualStatus.state).not.toBe("swept");
        console.log(`\n✅ VTXO 소비 상태: spent=${targetVtxo.isSpent}, swept=${targetVtxo.isSwept}`);
    });

    it("[Step 3] 온체인 앵커링 검증 — esplora에 직접 물어보기", async () => {
        const { targetVtxo, commitmentTxids } = fixture;
        const commitmentTxid = commitmentTxids[0];
        console.log(`\n🔍 commitment tx 온체인 확인: ${commitmentTxid}`);

        const result = await verifyOnchainAnchoring(commitmentTxid, 0);

        console.log(`   온체인 존재: ${result.ok}`);
        console.log(`   확인 수: ${result.confirmations ?? "N/A"}`);
        if (result.error) console.log(`   오류: ${result.error}`);

        expect(result.ok).toBe(true);
        expect(result.confirmations).toBeGreaterThanOrEqual(1);
    }, 15_000);

    it("[통합] 전체 VTXO 검증 파이프라인", async () => {
        const { targetVtxo, vtxoChain, vtxoTreeData } = fixture;

        const result = await verifyVtxo(
            targetVtxo,
            vtxoChain,
            {
                vtxoTreeNodes: vtxoTreeData?.vtxoTree ?? undefined,
                commitmentTxHex: fixture.commitmentTxHex ?? undefined,
                indexer: indexerProvider,
                minConfirmations: 1,
            }
        );

        console.log("📊 검증 결과:");
        for (const [key, val] of Object.entries(result.checks)) {
            console.log(`   ${val ? "✅" : "❌"} ${key}`);
        }

        if (targetVtxo.virtualStatus?.state === "preconfirmed") {
            expect(result.checks.chainReconstructed).toBe(true);
            expect(result.checks.notDoubleSpent).toBe(true);
        } else {
            expect(result.success).toBe(true);
        }
    }, 30_000);
});
```

---

## 테스트 구조 설명

이 테스트는 **fixture.json을 로드해서 서버 없이 로컬에서 직접 VTXO를 검증**한다.

### 준비 (beforeAll)
- `fixture.json`을 파일에서 읽어온다 (setup.ts로 미리 만들어놔야 함)
- `RestArkProvider`로 서버에서 `signerPubkey`를 가져와 `sweepTapTreeRoot`로 변환
- `RestIndexerProvider` 인스턴스 생성

### 5개 테스트

| 테스트 | 역할 |
|---|---|
| fixture 데이터 구조 확인 | 필수 필드가 있는지 체크, VTXO 정보 출력 |
| [Step 1] 체인 연결 검증 | `verifyChainLinking`으로 족보 연결 확인 |
| [Step 2] 소비 상태 검증 | `isSpent` 와 `swept` 상태 확인 |
| [Step 3] 온체인 앵커링 | `verifyOnchainAnchoring`으로 블록체인에 존재 확인 |
| [통합] 전체 파이프라인 | `verifyVtxo`로 5개 체크 전부 한 번에 실행 |

### preconfirmed 처리
VTXO가 아직 온체인에 없는 `preconfirmed` 상태면 온체인 확인은 완전히 통과하길 기대하지 않는다. 체인/소비 상태 검증만 통과하면 OK.
