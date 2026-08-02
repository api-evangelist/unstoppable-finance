---
name: Create a fiat-to-stablecoin onramp (Autoramp)
description: Register and verify a destination wallet, quote the conversion, then create a standing Autoramp that converts incoming fiat to stablecoins.
api: openapi/unstoppable-finance-iron-openapi-original.yml
operations: [registerSelfHostedWalletAddress, registerAttestedWalletAddress, registerHostedWallet, searchHostedVasps, getAutorampQuotePreview, createAutoramp, getAutorampById, getAutorampTransactions, createSandboxTransaction, updateSandboxTransactionState]
---

# Create a fiat-to-stablecoin onramp (Autoramp)

An Autoramp is Iron's core primitive: a standing conversion rule between fiat and crypto. Prereq: an onboarded, KYC-approved customer (see the onboarding skill).

1. Register the destination wallet (Travel Rule requires verification for every Autoramp):
   - Self-hosted with signed proof-of-ownership: `registerSelfHostedWalletAddress` (`POST /addresses/crypto/selfhosted`).
   - Self-attested (US / Rest of World): `registerAttestedWalletAddress` (`POST /addresses/crypto/attested`).
   - Institution-hosted: find the VASP with `searchHostedVasps`, then `registerHostedWallet` (`POST /addresses/crypto/hosted`).
   All registration calls take `IDEMPOTENCY-KEY`.
2. Preview pricing with `getAutorampQuotePreview` (`GET /autoramps/quotes-preview`).
3. `createAutoramp` (`POST /autoramps`) with `IDEMPOTENCY-KEY`; body includes `source_currencies` (e.g. Fiat EUR), `destination_currency` (e.g. USDC on Ethereum), `recipient_account`, and `customer_id`.
4. Wait for approval: poll `getAutorampById` or subscribe to the `register_autoramp_status` webhook. In sandbox, approve it via the dashboard or `PUT /sandbox/autoramp/{id}` (`updateSandboxAutorampStatus`, body `"Approved"`).
5. Fund it: in production the customer sends fiat to the provisioned deposit account. In sandbox, simulate the deposit with `createSandboxTransaction` (`POST /sandbox/transaction`, body `{"autoramp_id": ..., "amount": "100"}`) and drive it with `updateSandboxTransactionState`.
6. Track conversions with `getAutorampTransactions` (`GET /autoramp-transactions`) or the `transaction_status` webhook topic.

Rate limits return HTTP 429 - retry with exponential backoff.
