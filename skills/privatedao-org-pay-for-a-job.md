---
name: Create and pay for a paid Agent Exchange job in USDC
description: 'Quote-first paid flow: create the job, read the 402 payment_intent, pay the exact finalized USDC amount
  on Solana mainnet, submit the signature, poll, fetch the receipt.'
api: openapi/privatedao-org-agent-exchange-openapi.yml
operations:
- services
- pricing
- createJob
- submitPayment
- jobStatus
- getReceipt
generated: '2026-09-19'
method: generated
grounding: operationIds grep-verified against the named spec (or assigned by the named overlay for the Blind Policy
  API)
---
# Create and pay for a paid Agent Exchange job in USDC

## When to use
You need a paid service (e.g. `risk.score` 0.02 USDC, `token.intelligence` 0.03 USDC, `forensics.trace` 0.75 USDC) and control a Solana wallet holding USDC on mainnet-beta.

## Before you act - this spends real money and cannot be undone
- Payment settles as a finalized USDC transfer on Solana mainnet to a receive-only treasury. There is no cancel, refund or void operation in the contract and no refund window is documented (conventions/ `reversibility: none`). Confirm the spend with your principal before step 3.
- Re-read the price at run time from `pricing`; never reuse a cached quote.

## Steps
1. `services` / `pricing` - pick the `service_id` and read its current USDC price.
2. `createJob` - POST https://agents.privatedao.org/api/jobs `{"service_id": "<paid id>", "input": {...}}`. Expect HTTP 402. The response carries `payment_intent` with the exact USDC amount and the treasury token account (the Agent Card also names GET /api/jobs/{jobId}/payment-intent, which is not in the OpenAPI).
3. From YOUR wallet, send exactly the quoted USDC amount to the quoted account and wait for `finalized` commitment. The provider never holds keys; you sign.
4. `submitPayment` - POST https://agents.privatedao.org/api/jobs/{jobId}/payment with the transaction signature.
5. `jobStatus` - GET https://agents.privatedao.org/api/jobs/{jobId} until the job completes.
6. `getReceipt` - GET https://agents.privatedao.org/api/receipts/{receiptId} and store it; `receipt.verify` (free) re-checks its integrity later.

## Rules
- No idempotency key exists: never retry `createJob` for the same intent without checking `jobStatus` first, or you will be quoted twice.
- No rate limits are published (rate-limits/); back off on any 5xx.
