---
title: Economics
nav_order: 3
---

# Economics

## The fee

The protocol takes a fee in basis points on settlement only. The production configuration is **100 basis points — 1%**. The contract caps it at 1000 bps (10%) and this ceiling is enforced at initialisation, so a misconfigured deployment cannot charge more.

```
fee = amount * fee_bps / 10_000
net = amount - fee
```

All integer arithmetic. There are no floating point numbers anywhere in this protocol. Division truncates, and truncation always favours the clinic rather than the protocol.

### Worked numbers at 1%

| Voucher | Fee | Clinic receives |
| --- | --- | --- |
| $1.00 (malaria rapid test) | $0.01 | $0.99 |
| $3.00 (outpatient consult) | $0.03 | $2.97 |
| $5.00 (antenatal visit) | $0.05 | $4.95 |
| $25.00 (minor procedure) | $0.25 | $24.75 |

Verified on testnet: a $3.00 voucher settled to exactly `29700000` stroops for the clinic and `300000` for the fee account.

### What is not charged

- **Refunds are free.** An expired or successfully disputed voucher returns the full amount. No fee is taken on care that did not happen.
- **Funding is free.** The fee is taken once, at settlement.
- **No spread.** The protocol does not convert currency and takes no exchange margin.

## Network costs

Stellar transaction fees are paid in XLM and are a fraction of a cent. At the time of writing the base fee is 100 stroops — 0.00001 XLM — per operation.

This matters more than it sounds. A voucher's whole lifecycle is four transactions: create, claim, attest, settle. On a chain where a transaction costs $0.50, that is $2.00 of overhead on a $3.00 consultation, and the product does not exist. Here it is a rounding error.

## Who pays what

| Action | Who submits | Who pays the network fee |
| --- | --- | --- |
| `create_voucher` | Funder | Funder |
| `claim` | Clinic | Clinic |
| `attest` | Attester | Attester |
| `settle` | Anyone | Whoever submits |
| `refund` | Anyone | Whoever submits |

Because `settle` and `refund` are permissionless, an operator can run a keeper that settles eligible vouchers on everyone's behalf and absorbs those fees. Nothing in the protocol depends on that keeper existing — it is a convenience, not a dependency.

## Why there is no float

The contract holds no balances and has no deposit function. Money enters only as a voucher already bound to a clinic, a service and an expiry, and it can only leave to that clinic or back to the funder.

This is a deliberate design constraint rather than an oversight, and it is the difference between a prepaid voucher and stored value. A general reloadable balance is e-money in most jurisdictions and brings licensing with it. A prepaid voucher for an identified provider and an identified service generally falls under limited-network carve-outs.

The protocol also **bears no risk**. It does not pool premiums, does not underwrite, and does not pay out on a health event. The moment a system takes money in and promises to pay for a future uncertain medical need, it is insurance and needs a licence. This protocol only ever moves money a funder has already committed to a specific purchase.
