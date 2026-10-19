# dev-tools-tron

A single HTML page for day-to-day Tron scanner work: create a Base58 address, build the watch-list payload, and sign a Collect transaction (native TRX or USDT) in the browser.

**Live:** [https://afsh7n.github.io/dev-tools-tron/](https://afsh7n.github.io/dev-tools-tron/)

The private key never leaves the tab. Address generation and signing run locally with `ethers` from a CDN. The only network call is fetching the latest block header from TronGrid for the live fields. The page does not broadcast and cannot speak native gRPC, so Scan and Collect only produce the JSON you paste into Postman or grpcurl.

## What you use it for

1. **Address** — generate a Tron keypair (`T…`), reveal the key, or download and load `wallet.json`.
2. **Scan** — build `BatchUpdateAddresses` for `tron_scanner.AddressService`, operation `ADDRESS_OPERATION_ADD`.
3. **Sign Collect** — sign one item for `tron_scanner.TransactionService/Collect`. Leave `contract` empty for native TRX; fill it for USDT (TRC-20). The red fields are used only in the TRC-20 case.

## Fees: burning TRX

This tool targets the "burn TRX" setup: no staking and no delegated bandwidth or energy. A Tron transaction has no bandwidth/energy fields; the network charges the sender automatically.

- **Native TRX** — a plain `TransferContract`. It consumes bandwidth only (no energy). With no staked bandwidth, the network burns TRX from the sender. Nothing extra to fill in.
- **USDT (TRC-20)** — a `TriggerSmartContract`. It consumes energy. With no staked energy, the network burns TRX from the sender, and `fee_limit` (sun) is the maximum it may burn.

## Live block data

**Fetch latest block** calls `https://api.trongrid.io/wallet/getnowblock` and fills all four chain-validity fields in one click:

- `ref_block_bytes` — low 2 bytes of the block number.
- `ref_block_hash` — bytes 8–16 of the block ID.
- `timestamp` — block timestamp in milliseconds.
- `expiration` — block timestamp + 60 seconds.

Re-fetch right before broadcasting. An old reference block is rejected by the network.

## Sign fields

| Field | Native TRX | USDT (TRC-20) | Role |
| --- | --- | --- | --- |
| `to_address` | Required | Required | Recipient. Base58 or `41…` hex. |
| `amount` | Required | Required | Atomic units. `1 TRX = 1000000` sun; USDT has 6 decimals, so `1 USDT = 1000000`. |
| `contract` | — | Required | Leave empty for native TRX. **Use USDT** fills the official USDT contract `TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t`. |
| `fee_limit` | — | Required | Sun. Maximum TRX burned for energy. Ignored when `contract` is empty. Default `100000000` (100 TRX). |
| `transaction_id` | Required | Required | Scanner item id, not the on-chain `txID`. 1–64 chars (`A–Z`, `a–z`, `0–9`, `_`, `-`). |
| `token_id` | Required | Required | Scanner catalog token id. |
| `ref_block_bytes` | Required | Required | 4 hex chars. Low 2 bytes of the reference block number. |
| `ref_block_hash` | Required | Required | 16 hex chars. 8 bytes of the reference block id. |
| `expiration` / `timestamp` | Required | Required | Milliseconds. Empty means now + 60s and now. |

`ref_block_bytes` and `ref_block_hash` come from a recent block. Without them the signature is well-formed but the network will not accept the transaction.

`signed_tx` is the signed Tron transaction as a JSON string: `txID`, `raw_data`, `raw_data_hex`, and `signature`. That is the payload the scanner parses and later broadcasts.

## Open it

Use the [live page](https://afsh7n.github.io/dev-tools-tron/), or open `index.html` locally. Do not commit `wallet.json`.
