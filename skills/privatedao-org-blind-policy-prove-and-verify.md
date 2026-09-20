---
name: Prove and verify a Blind Policy decision
description: Generate a Groth16 proof that a private policy was satisfied, verify the public proof package, optionally
  anchor the receipt on Solana.
api: openapi/privatedao-org-blind-policy-openapi.yml
operations:
- getBlindPolicyStatus
- getBlindPolicySample
- proveBlindPolicy
- verifyBlindPolicy
- storeBlindPolicyOnchainReceipt
generated: '2026-09-19'
method: generated
grounding: operationIds grep-verified against the named spec (or assigned by the named overlay for the Blind Policy
  API)
---
# Prove and verify a Blind Policy decision

Base URL: https://api.privatedao.org/api/v1. The provider's OpenAPI declares NO operationIds; the ids used here are assigned by overlays/privatedao-org-blind-policy-overlay.yaml and map 1:1 to the five paths.

## Steps
1. `getBlindPolicyStatus` - GET /proof-workflows/blind-policy/status. Read `proofSystem` (Groth16), `circuit`, `publicSignals` and `proofIssueRules`.
2. `getBlindPolicySample` - GET /proof-workflows/blind-policy/sample for a safe fixture (`privateInputs` + `publicPolicyShape`). Use it to test the flow before sending customer data (sandbox/privatedao-org-sandbox.yml).
3. `proveBlindPolicy` - POST /proof-workflows/blind-policy/prove with `BlindPolicyPrivateInputs` (organizationId, subjectId, membershipVerified, records[], riskScore, liabilitiesUsd). A proof is issued ONLY if membership is verified, records are present, the policy is satisfied and Groth16 prove+verify succeed; otherwise expect 422.
4. `verifyBlindPolicy` - POST /proof-workflows/blind-policy/verify with the returned `BlindPolicyProofPackage`. It recomputes the package hash, checks expiry and returns match or mismatch.
5. `storeBlindPolicyOnchainReceipt` - POST /proof-workflows/blind-policy/onchain-receipt to write the receipt hashes to a Solana Anchor PDA (or a Memo transaction when the program is unavailable). The response labels the storage mode.

## Rules
- Private inputs go ONLY to `prove`; never send them to `verify` or `onchain-receipt`.
- Proof packages embed proofId, nonce, issuedAt, expiresAt, circuitVersion and policyVersion; an expired or altered package verifies as mismatch. This is replay protection for the verifier, not idempotency for `prove`.
- Anchoring is irreversible; there is no delete.
- Errors on this host are `{"ok": false, "error": string}`.
