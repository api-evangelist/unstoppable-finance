---
name: Offramp stablecoins to a bank account
description: Register a customer bank account, create an offramp Autoramp, and monitor payout status through webhooks.
api: openapi/unstoppable-finance-iron-openapi-original.yml
operations: [registerFiatAddress, getFiatAddresses, getFiatAddressesByStatus, createAutoramp, getAutorampTransactions, listWebhooks, pingWebhook, updateSandboxFiatAddressVerificationStatus]
---

# Offramp stablecoins to a bank account

1. `registerFiatAddress` (`POST /addresses/fiat`) with `IDEMPOTENCY-KEY` - register the customer's bank account (SEPA/ACH; PIX destinations use `registerPixAddress`).
2. Wait until the address reaches `Registered`: poll `getFiatAddresses`/`getFiatAddressesByStatus` or subscribe to the `register_fiat_address_status` webhook. In sandbox, set it directly with `updateSandboxFiatAddressVerificationStatus` (`PUT /sandbox/fiat-verification/{id}`, body `"Registered"`).
3. `createAutoramp` (`POST /autoramps`, `IDEMPOTENCY-KEY`) with a Crypto source currency and the registered fiat account as `recipient_account`.
4. The customer sends stablecoins to the provisioned deposit address; conversions appear in `getAutorampTransactions`.
5. Webhooks (Standard Webhooks spec, HMAC-SHA256 `webhook-signature` over `webhook-timestamp` + raw body): manage with `listWebhooks`, test with `pingWebhook`. Verify signatures with constant-time comparison.

Compliance: Travel Rule applies to all hosted-wallet transfers and to self-hosted transfers over EUR 1,000. Customer spending limits are per-direction on a 52-week rolling window - read live values from `getCustomerSpendingLimit` (`threshold_eur`, `used_eur`, `remaining_eur`).
