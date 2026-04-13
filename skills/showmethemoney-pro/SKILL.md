---
name: showmethemoney-pro
description: execute the paid showmethemoney premium action through the merchant backend. use when the user wants to unlock or run the protected showmethemoney capability, and payment should be handled through stablepay before retrying the action.
---

# ShowMeTheMoney Pro

Execute the protected ShowMeTheMoney premium action only after backend verification and StablePay payment succeed.

## Fixed merchant settings

Use these defaults for this skill:

- skill_name: `ManualDemoSkill2`
- skill_did: `did:solana:2kZGwkLnVdSxjjNueeUQmqBf3tRKMn7y1bbktRZkJWdR`
- default_price_usdc: `1.00`
- currency: `USDC`
- stablepay_gateway_base_url: `http://127.0.0.1:28080`
- merchant_backend_base_url: `http://127.0.0.1:8787`
- premium_action_endpoint: `/execute`

Prefer values returned by the backend or StablePay 402 response when available. Use the defaults above only as fallback.

## Preconditions

Before using this skill:

1. Call `stablepay_runtime_status`.
2. If no local wallet exists, create or bind one.
3. If no backend DID is registered, call `stablepay_register_local_did`.
4. If payment limits are missing, call `stablepay_configure_payment_limits`.

Do not require `stablepay_build_payment_policy` unless another workflow explicitly depends on it.

## Main workflow

When the user asks to use the premium ShowMeTheMoney capability:

1. Resolve the current buyer DID from `stablepay_runtime_status`.
2. Call the merchant backend premium endpoint:
   - `GET http://127.0.0.1:8787/execute?agent_did=<buyer_did>`
3. Treat the merchant backend as the source of truth.
4. If the backend returns `200`, return the protected result to the user.
5. If the backend returns `402 Payment Required`:
   - read `payment_requirement` from the backend response
   - prefer the returned `skill_did`, `skill_name`, `price`, `currency`, and `message`
   - call `stablepay_pay_via_gateway`
6. If payment succeeds, retry the same `/execute` request once.
7. If the retry still does not return `200`, report that the premium action is still locked or verification failed.

## Request contract

Use this request for the premium action:

- method: `GET`
- url: `http://127.0.0.1:8787/execute`
- required query parameter: `agent_did`

Expected responses:

- `200`: premium capability is unlocked and executed
- `402`: payment required, with a StablePay payment requirement payload
- `400`: missing or invalid input
- `502`: backend verification failure
- `500`: merchant backend internal error

## Verification rules

Always rely on backend verification.

Do not assume the user has access just because:

- a local wallet exists
- a DID is registered
- payment was attempted earlier
- a previous local state file exists

Only treat the action as unlocked if the backend returns success after verification.

## Payment rules

When payment is required:

1. Use `stablepay_pay_via_gateway`.
2. Use the requirement returned by the backend when present.
3. Respect the local payment limits already configured in the StablePay plugin.
4. Never claim payment succeeded unless StablePay returns a successful result.

## Scope limits

This skill is only for the protected premium action exposed by:

- `GET /execute?agent_did=<buyer_did>`

Do not use these backend routes as part of the premium skill flow:

- `/developer/revenue`
- `/developer/sales`
- `/agent/balance`
- `/agent/transactions`

Those are debugging or operator-facing routes, not the protected paid action itself.

## Safety

- Never expose private keys, mnemonic material, decrypted local state, API keys, or merchant secrets.
- Never bypass backend verification.
- Never invent payment success or purchase ownership.
- Never return premium output unless the backend confirms access.