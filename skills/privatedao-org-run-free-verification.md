---
name: Run a free verification job on the PrivateDAO Agent Exchange
description: Discover the service catalog, submit the free verify.basic job, and fetch the receipt.
api: openapi/privatedao-org-agent-exchange-openapi.yml
operations:
- services
- createJob
- getReceipt
generated: '2026-09-19'
method: generated
grounding: operationIds grep-verified against the named spec (or assigned by the named overlay for the Blind Policy
  API)
---
# Run a free verification job on the PrivateDAO Agent Exchange

## When to use
You need a canonical digest and receipt for a piece of JSON evidence (for example a Solana mint) and do not want to spend USDC.

## Steps
1. `services` - GET https://agents.privatedao.org/api/services. Confirm `verify.basic` is listed with `access: free`. Do not hard-code prices; they come from this call or `pricing`.
2. `createJob` - POST https://agents.privatedao.org/api/jobs with the provider's documented body shape:
   `{"service_id": "verify.basic", "input": {"record": {"mint": "<SOLANA_MINT>"}}}`
   The OpenAPI publishes no request schema; the body above is the example the Agent Card's `workflow.freeTest` gives. A 400 `{"error":"request_failed","message":"unknown service"}` means the `service_id` is wrong.
3. Read `result` and `receipt` from the 200 response. Free jobs complete inline.
4. `getReceipt` - GET https://agents.privatedao.org/api/receipts/{receiptId} to re-fetch the receipt independently; `receipt.verify` (also free) checks a receipt against an expected hash.

## Rules
- No authentication is required; do not attach keys.
- There is no idempotency key. A retried `createJob` creates a second job; keep your own request log (conventions/privatedao-org-conventions.yml).
- Errors are `{"error": string, "message"?: string}`; 404 `not_found` is returned for unknown jobId/receiptId (errors/privatedao-org-problem-types.yml).
