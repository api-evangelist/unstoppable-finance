---
name: Onboard a customer and complete KYC
description: Create an Iron customer, start identification (KYC), and confirm the customer is ready to transact.
api: openapi/unstoppable-finance-iron-openapi-original.yml
operations: [createCustomer, createCustomerIdentificationV2, getAllCustomerIdentifications, getCustomerById, getCustomerAbilities, getCustomerRequirements, updateSandboxIdentificationStatus]
---

# Onboard a customer and complete KYC

Base URLs: sandbox `https://api.sandbox.iron.xyz/api`, production `https://api.iron.xyz/api`.
Every request needs the `X-API-Key` header (Partner Dashboard -> Developer -> API Keys).
Write calls marked in the reference require an `IDEMPOTENCY-KEY` header with a fresh UUID; retries with the same key return the stored result.

1. `createCustomer` (`POST /customers`) - create the customer record. Send `IDEMPOTENCY-KEY`.
2. `createCustomerIdentificationV2` (`POST /customers/{id}/identifications/v2`) - start KYC. Use `type: Link` for the hosted flow; set `with_edd: true` to trigger Enhanced Due Diligence upfront and unlock Tier 2 (unlimited) limits.
3. Track progress with `getAllCustomerIdentifications` (`GET /customers/{id}/identifications`) or subscribe to the `identification_status` webhook topic.
4. In sandbox, approve the pending identification yourself with `updateSandboxIdentificationStatus` (`POST /sandbox/identification/{id}`, body `{"approved": true}`) - in production Iron's compliance team reviews it.
5. Confirm readiness: `getCustomerById`, `getCustomerAbilities` (what the customer may do), and `getCustomerRequirements` (outstanding requirements).

Errors: 401 means a missing/revoked `X-API-Key`; 409 means a conflicting create (check your idempotency key reuse). Spending limits are dynamic per customer - read them via `getCustomerSpendingLimit`, never hard-code thresholds.
