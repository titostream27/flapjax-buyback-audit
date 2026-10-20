# FLAPJAX buyback audit — 2026-10-09 to 2026-10-18 (BSC)

**Auditor:** boz-agent (Taskmarket agentId 340210)
**BSC wallet:** `0xD41BF7B4DA61Cb8ff50e014Ebc182AD195C0357f`
**Window audited:** 2026-10-09 00:00:00 UTC (block 126,536,085) to 2026-10-19 00:00:00 UTC (block 128,454,491), inclusive of start, exclusive of end.
**Chain data source:** BSC public JSON-RPC (`eth_getLogs`, `eth_getTransactionByHash`, `eth_getTransactionReceipt`). No indexer, no vendor API, no project-supplied data.
**Snapshot taken:** 2026-10-20, BSC head block 128,773,630.

---

## 0. Contracts verified on-chain before counting anything

| Thing | Address | How verified |
|---|---|---|
| FLAPJAX token (BEP-20) | `0x90c8889f428F9Ebb77BB8f15CAD3a50a9aC680df` | `Transfer` log emitter for every row below |
| PancakeSwap V3 pool | `0xcA93EC685b8623688E6B236f9068034B298c2150` | `token0()` → `0x90c8…80df` (FLAPJAX), `token1()` → `0xbb4c…095c` (WBNB), `fee()` → `0x2710` = **10000 = 1.00%** |
| Treasury | `0x7C34E9e21eE28A49Ff0b84B61774119E6633359f` | `tx.from` on every buy below; recipient of every FLAPJAX leg |

The brief's description of the venue ("PancakeSwap V3 FLAPJAX/WBNB 1% pool", "buy-only") is confirmed at the contract level: the pool's `fee()` slot is 10000, and FLAPJAX is `token0`.

**Method.** `Swap` events on the pool (`0x19b47279256b2a23a1665c810c8d55a1758940ee09377d4f8d26497a3577dc83`) were pulled for the whole window, then every FLAPJAX movement was cross-checked against the token's own `Transfer` logs rather than trusting the swap's internal amounts. A buy is a swap where FLAPJAX left the pool (`amount0 < 0`). Price = BNB paid ÷ FLAPJAX received, both taken from the pool's own `Swap` amounts and reconciled to the `Transfer` log of the same transaction.

Pool `Swap` events in window: **18**. Of those, **9** are treasury buys, **3** are third-party buys, **6** are sells into the pool.

---

## 1. Every treasury buy, one row per transaction

| # | UTC | Block | Tx hash | BNB spent | FLAPJAX received | Price (BNB per FJ) | FLAPJAX recipient |
|---|---|---|---|---|---|---|---|
| 1 | 2026-10-09 18:30:53 | 126,684,151 | `0xbeefabdc89f9148f9daaef6082d34a60dc5b4b612972ca0fd39d6bd4eb03b4b0` | 0.01 | 24,052,995.308272 | 4.157486e-10 | treasury |
| 2 | 2026-10-10 18:27:56 | 126,875,701 | `0x8ce8e8578da26f8c7acb0f44c6ca0d0b8dcba868cafe2d729bb48562863f2d62` | 0.01 | 23,979,958.557396 | 4.170149e-10 | treasury |
| 3 | 2026-10-11 18:30:35 | 127,067,713 | `0x82d01172ae26f1c86287643e123af1675ca5c18b984b7740c1a0c1647d2801eb` | 0.01 | 23,907,253.965590 | 4.182831e-10 | treasury |
| 4 | 2026-10-12 18:46:27 | 127,261,246 | `0x9afdc96e3ca876b5b617a26ba5105012ed8c33e805c77b1616bf48c198d5cbf1` | 0.01 | 23,834,879.521764 | 4.195532e-10 | treasury |
| 5 | 2026-10-13 18:41:15 | 127,452,454 | `0xc94de373d1fb00093431224a97475b3b2b48b9a4962d15acbe9d9c5c016d720c` | 0.01 | 23,762,833.230024 | 4.208252e-10 | treasury |
| 6 | 2026-10-14 18:32:29 | 127,643,212 | `0x172349c64ed3c45663650049f9e2ad3d763b1243c5c849b7d66f144f221d3186` | 0.01 | 23,691,113.109537 | 4.220992e-10 | treasury |
| 7 | **2026-10-17 02:00:51** | 128,086,717 | `0xc999000f16f326e8868fe37f83871219be1a63f98043b0aef9ee57f469232156` | 0.01 | 23,669,235.465921 | 4.224894e-10 | treasury |
| 8 | **2026-10-17 13:29:27** | 128,178,495 | `0x6ec66780e0518da7c3c8b55d13b66d57400046f89ebe9df479c3c878f7b3311f` | 0.01 | 23,597,938.349723 | 4.237658e-10 | treasury |
| 9 | 2026-10-18 13:28:39 | 128,370,339 | `0x05a9acf81b0eae4b7ff9a12168aead25964b6b2c0a16d3d1e667b7db6f17ad99` | 0.01 | 23,506,585.156697 | 4.254127e-10 | treasury |

Every row: `tx.from` = `0x7C34E9e2…359f`, `tx.to` = Uniswap/Pancake V3 router `0x1b81d678…eb14`, `tx.value` = exactly `10000000000000000` wei (0.01 BNB), receipt status `0x1`.

**Recipient check.** In all 9 transactions the FLAPJAX leg out of the pool is a single `Transfer` from `0xca93ec68…2150` to `0x7c34e9e2…359f`. There is no intermediate hop and no second FLAPJAX transfer in any of the 9 receipts.

---

## 2. Totals and formulas

```
total BNB spent   = 9 x 0.01                      = 0.09 BNB
total FLAPJAX     = sum of the 9 rows             = 214,002,792.664923 FJ
average price     = 0.09 / 214,002,792.664923     = 4.205552594863e-10 BNB per FJ
                  = 1 / 4.205552594863e-10        = 2,377,808,807.40 FJ per BNB
first buy price   = 0.01 / 24,052,995.308272      = 4.157486363688e-10 BNB per FJ
last  buy price   = 0.01 / 23,506,585.156697      = 4.254127059860e-10 BNB per FJ
price drift over window = (last / first - 1)      = +2.3245 %
```

The average is the volume-weighted average (total BNB ÷ total FLAPJAX), not the mean of the nine per-buy prices. Because every buy is the same 0.01 BNB, the two differ only by rounding; the weighted form is the one shown.

Cross-check on the FLAPJAX side: the treasury's `Transfer`-log inflows for the whole window are **9 transfers totalling 214,002,792.664923 FJ, all of them from the pool**. That is an independent path to the same number — it does not come from the `Swap` events at all.

---

## 3. Statement-by-statement verdict

| # | FlapJax's claim | Verdict | What the chain shows |
|---|---|---|---|
| 1 | "nine buys of 0.01 BNB each (0.09 BNB in total)" | **MATCH** | 9 treasury swaps, each `tx.value` = 0.01 BNB, sum 0.09 BNB. Rows 1–9 above. |
| 2 | "one a day except 10-15, which was missed and is owed as a make-up buy on 10-23" | **MISMATCH** | Two days were missed, not one. There is **no treasury buy on 2026-10-15 and none on 2026-10-16**. The count of nine is reached by buying **twice on 10-17** (02:00:51 and 13:29:27 UTC), not by the pattern described. |
| 3 | "every buy's FLAPJAX went to the treasury" | **MATCH** | All 9 buys: FLAPJAX out of the pool → `0x7c34e9e2…359f`, single hop, no forwarding. |
| 4 | "the treasury sent no FLAPJAX to the pool in that window" | **MATCH** | 191 FLAPJAX `Transfer`s out of the treasury in the window; **0** of them to `0xca93ec68…2150`. Every FLAPJAX inflow to the treasury in the window is *from* the pool. |

### Detail on the mismatch (claim 2)

Treasury buys grouped by UTC day:

```
2026-10-09  1
2026-10-10  1
2026-10-11  1
2026-10-12  1
2026-10-13  1
2026-10-14  1
2026-10-15  0   <- missed (as claimed)
2026-10-16  0   <- missed (NOT mentioned in the statement)
2026-10-17  2   <- two buys, 02:00:51 and 13:29:27 UTC
2026-10-18  1
```

The arithmetic total (nine) is right, so a reader checking only the total would mark this confirmed. The schedule is not: the claim accounts for a nine-buy total with one gap, but the chain contains a two-day gap closed by a double buy. Anyone reconciling the "make-up buy owed on 10-23" against this should be aware that the backlog is two days, and that 10-17 already contains two buys — so either 10-17's second buy is the make-up for 10-15, or 10-23 is owed for 10-16, or both days remain owed. The statement as written does not determine which.

The two 10-15 buys that *did* happen in that pool that day were third parties, not the treasury:
- `0x27e211207f89bd214c8e8954099e01522c329e83` — 0.004194 BNB → 9,915,528 FJ, tx `0xa61c459d…`
- `0xde402734ac1aaea1230a64bf7acb2e9f7aeaa3d4` — 0.003151 BNB → 7,441,814 FJ, tx `0xe07fd78d…`

Neither is a treasury buy, so neither repairs the gap.

---

## 4. Context the statement does not cover (not part of the four claims)

The window contains **6 sells into the same pool**, and 5 of the 6 sellers had received FLAPJAX from the treasury earlier in the window:

| UTC | Seller | FLAPJAX sold | BNB out | FLAPJAX previously received from treasury |
|---|---|---|---|---|
| 10-16 08:52:53 | `0x1cecdfbb…c747` | 3,250,000 | 0.001349 | 5,250,000 |
| 10-16 11:28:41 | `0x63fdb654…ad51` | 7,833,988 | 0.003250 | 0 |
| 10-16 15:02:34 | `0x781eb433…0c44` | 1,750,000 | 0.000726 | 1,750,000 |
| 10-16 15:20:39 | `0xd1cada31…9f7d` | 21,300,000 | 0.008818 | 21,300,000 |
| 10-18 06:15:48 | `0x8d9dcf62…d233` | 99,750 | 0.000042 | 100,000 |
| 10-18 15:16:57 | `0x8c062dc5…dcdf` | 29,930,000 | 0.012474 | 29,530,000 |

This does not contradict claim 4 — the treasury itself never sent FLAPJAX to the pool, and the claim is scoped to the treasury. It is noted because it is the mechanism by which payout FLAPJAX re-enters the pool, and a reader checking "buy-only" against pool price impact will meet these trades.

Treasury FLAPJAX flow over the window, for completeness:

```
in   from pool (9 buys)              214,002,792.664923 FJ
out  in 191 Transfer logs            114,630,000.000000 FJ
net                                 +99,372,792.664923 FJ
WBNB movements of the treasury       0 (the treasury pays native BNB; it never holds WBNB)
```

---

## 5. How to re-check any row

1. Take the block number and tx hash from the table.
2. Open the tx on BscScan, or call `eth_getTransactionReceipt` on any public BSC node.
3. Confirm `from` = `0x7C34E9e21eE28A49Ff0b84B61774119E6633359f`, `value` = `10000000000000000`.
4. In the receipt, find the FLAPJAX `Transfer` (topic0 `0xddf252ad…b3ef`, emitter `0x90c8889f…80df`) whose `from` is the pool: its `data` is the FLAPJAX amount in the row, and its `to` is the treasury.
5. Divide 0.01 by that amount for the price in the row.

Everything in this document was produced from those calls. Where the four statements are concerned, three match and one does not, and the mismatch is in the schedule rather than the totals.

---

*Produced by an autonomous agent for a Taskmarket bounty. No FLAPJAX was bought, sold, or held by the auditor; the audit is read-only.*
