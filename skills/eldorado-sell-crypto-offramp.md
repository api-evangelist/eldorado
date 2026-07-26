---
name: Sell crypto for local fiat (offramp)
description: Authenticate a user, confirm KYC, then create a sell quote/order that converts USDT (Arbitrum) into local fiat paid out over a local payment method via the El Dorado API.
api: https://api.eldorado.io/api/
operations:
  - POST /auth/login/send-otp
  - POST /auth/login/verify-otp
  - GET /kyc
  - POST /quote/sell
---

# Sell crypto for local fiat (offramp)

Use the El Dorado API to let a user sell crypto and receive local fiat (e.g. COP via Nequi, ARS via bank transfer).

## Prerequisites
- Partner **ClientID** + **ReferralID** on every call (`X-Client-ID`, `X-Referral-ID`).
- Base URL `https://api.eldorado.io/api/` (or the testnet host in sandbox).

## Steps

1. **Authenticate.** `POST /auth/login/send-otp` then `POST /auth/login/verify-otp`; keep the JWT and send `Authorization: Bearer <token>`.

2. **Confirm KYC L2+.** `GET /kyc`; advance the user if needed (trading requires L2, monthly limits scale L2 $10k → L3 $250k → L4 $1M USDT).

3. **Create the sell quote.** `POST /quote/sell` with `{ addressFrom, token, chainId, fiatCurrencyId, paymentMethodId, amount, fixedAmountSide: "IN" }`. Read `amountOut` (fiat), `minAmountOut`, `exchangeRate`, `fee`, `quoteId`, `expiresAt`.

4. **Create the order** from the quote. For sell orders the user deposits **USDT on Arbitrum** to the deposit address the API returns; to send another token/chain, execute the provided `txData` swap/bridge via li.fi.

5. **Settle.** On completion the response includes `transferInfo` (`txHash`, `transferDate`, `amountOut`) and the fiat is paid out via the chosen payment method.

## Rules
- All trades concentrate liquidity in USDT on Arbitrum; deposit exactly USDT on Arbitrum.
- If an order fails after the USDT deposit, the deposited funds carry to a new order with the same parameters.
- `fixedAmountSide` only supports `IN`. Errors return `{ code, reason }`.
- Sandbox: use the testnet USDT token `0x25f5D414F85b7f4668ADe4E557ef234023208771` on Arbitrum Sepolia (chainId 421614) from the El Dorado faucet.
