---
name: Buy crypto with local fiat (onramp)
description: Authenticate a user, ensure KYC, then create a buy quote that converts local fiat (e.g. ARS, COP) into USDT on Arbitrum via the El Dorado API.
api: https://api.eldorado.io/api/
operations:
  - POST /auth/login/send-otp
  - POST /auth/login/verify-otp
  - GET /kyc
  - POST /kyc
  - GET /currencies
  - GET /payment-methods
  - POST /quote/buy
---

# Buy crypto with local fiat (onramp)

Use the El Dorado API to let a Latin American user buy crypto (settled as USDT on Arbitrum) using a local fiat payment method.

## Prerequisites
- A partner **ClientID** and **ReferralID** (request from api@eldorado.io). Send both on every call:
  `X-Client-ID: <clientId>` and `X-Referral-ID: <referralId>`.
- Base URL: `https://api.eldorado.io/api/` (production) or `https://api-testnet.eldorado.io/api/` (sandbox).

## Steps

1. **Authenticate the user.**
   - `POST /auth/login/send-otp` with `{ email, language }` and the ClientID/ReferralID headers.
   - `POST /auth/login/verify-otp` with `{ otp, otpIdentifier, username }`. Save the returned JWT `token`.
   - Send `Authorization: Bearer <token>` on all user-facing calls after this. In sandbox the email OTP is always `12345678`.

2. **Confirm KYC (L2 minimum to trade).**
   - `GET /kyc` to read `kycLevel`/`kycStatus`.
   - If below L2, advance the user: `POST /kyc` with the required level data (L1 needs country, city, phone), then verify phone via `POST /kyc/phone` (sandbox code `1234`), then complete L2 document verification through the returned external URL.

3. **Fetch market data.**
   - `GET /currencies` for supported fiat/crypto IDs and `GET /payment-methods` for local rails (e.g. `bank_tx_ar`, `app_nequi_co`).

4. **Create the buy quote.**
   - `POST /quote/buy` with `{ addressTo, token, chainId, fiatCurrencyId, paymentMethodId, amount, fixedAmountSide: "IN" }`.
   - Read back `amountOut`, `minAmountOut`, `exchangeRate`, `fee`, `quoteId`, `expiresAt`.

5. **Create the order** from the quote before it expires (quotes carry an `expiresAt`). All buy orders settle in USDT on Arbitrum; other tokens/chains are bridged via li.fi.

## Rules
- `fixedAmountSide` only supports `IN` today.
- Handle token expiry by re-authenticating; handle `Too many requests` with throttling.
- Use `externalReferenceId` (letters/numbers/hyphens) to correlate the quote/order with webhook notifications.
- Errors return `{ code, reason }` — branch on `code` (see errors/eldorado-problem-types.yml).
