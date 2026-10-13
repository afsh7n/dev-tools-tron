# dev-tools-tron

A single HTML page for day-to-day Tron scanner work: create a Base58 address, build the watch-list payload, and sign a Collect transaction in the browser.

**Live:** [https://afsh7n.github.io/dev-tools-tron/](https://afsh7n.github.io/dev-tools-tron/)

The private key never leaves the tab. Address generation and signing run locally with `ethers` from a CDN. The page does not talk to a node and cannot speak native gRPC, so Scan and Collect only produce the JSON you paste into Postman or grpcurl.

## What you use it for

1. **Address** — generate a Tron keypair (`T…`), reveal the key, or download and load `wallet.json`.
2. **Scan** — build `BatchUpdateAddresses` for `tron_scanner.AddressService`, operation `ADDRESS_OPERATION_ADD`.
3. **Sign Collect** — sign one item for `tron_scanner.TransactionService/Collect`.

An empty TRC-20 contract field signs native TRX (`TransferContract`). A contract address signs a TRC-20 `transfer` (`TriggerSmartContract`). `amount` is an atomic-unit integer (sun for TRX).

`signed_tx` is the signed Tron transaction as a JSON string: `txID`, `raw_data`, `raw_data_hex`, and `signature`. That is the payload the scanner parses and later broadcasts.

## Sign fields

| Field | Role |
| --- | --- |
| `to_address` | Recipient. Base58 or `41…` hex. |
| `amount` | Atomic units. |
| `transaction_id` | Caller id, 1–64 chars (`A–Z`, `a–z`, `0–9`, `_`, `-`). |
| `token_id` | Catalog token id. |
| TRC-20 contract | Empty means native TRX. |
| `fee_limit` | Sun. Used for TRC-20. Default `100000000`. |
| `ref_block_bytes` | 4 hex chars. Low 2 bytes of the reference block number. |
| `ref_block_hash` | 16 hex chars. 8 bytes from the reference block id. |
| `expiration` / `timestamp` | Milliseconds. Empty means now + 60s and now. |

`ref_block_bytes` and `ref_block_hash` come from a recent block. Without them the signature is well-formed but the network will not accept the transaction.

## Open it

Use the [live page](https://afsh7n.github.io/dev-tools-tron/), or open `index.html` locally. Do not commit `wallet.json`.
