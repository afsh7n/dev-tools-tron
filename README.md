<h1 align="center">TRX Dev Tools</h1>

<p align="center">
  A local-first Tron workbench for scanner testing.<br />
  Build, hash, sign and inspect transactions without a node, a backend, or a browser wallet.
</p>

<p align="center">
  <a href="https://afsh7n.github.io/dev-tools-tron/"><img src="https://img.shields.io/badge/live-demo-58a6ff?style=flat-square" alt="Live demo" /></a>
  <img src="https://img.shields.io/badge/ethers-v6-8b5cf6?style=flat-square" alt="ethers v6" />
  <img src="https://img.shields.io/badge/protobuf%20%2B%20sha256-2ea86f?style=flat-square" alt="protobuf + SHA-256" />
  <img src="https://img.shields.io/badge/keys-stay_local-3ecf8e?style=flat-square" alt="Keys stay local" />
  <img src="https://img.shields.io/badge/build-none-e6c07b?style=flat-square" alt="No build step" />
</p>

<p align="center">
  <a href="https://afsh7n.github.io/dev-tools-tron/">Live app</a> ·
  <a href="#collect-flow">Collect flow</a> ·
  <a href="#inspect">Inspect</a> ·
  <a href="#live-block-data">Live block data</a> ·
  <a href="#fields">Fields</a> ·
  <a href="#quick-start">Quick start</a>
</p>

---

**TRX Dev Tools** is a single `index.html` for day-to-day Tron scanner work: create a throwaway address, build the watch-list payload, and turn a Collect request into a verified signed transaction — native TRX or any TRC-20 token such as USDT. It also decodes raw transactions so you can see exactly what a txID and signature cover.

The private key never leaves the tab. Address generation, protobuf encoding, hashing and signing run locally with `ethers` v6 from a CDN. The only network call is a live block-header fetch from TronGrid. The page does not broadcast and cannot speak native gRPC, so Scan and Collect produce the JSON bodies you paste into Postman or grpcurl.

## What it does

| Tab | Job |
| --- | --- |
| **Wallet** | Generate a Tron keypair (`T…`), reveal or copy the key, download `wallet.json`, or load one. |
| **Scan** | Build the `BatchUpdateAddresses` body for `tron_scanner.AddressService` with operation `ADDRESS_OPERATION_ADD`. |
| **Collect & sign** | Four explicit steps: build the raw_data object, encode protobuf and compute the txID, sign it, export the Collect item. Each step exposes its output and a copy button. |
| **Inspect** | Paste raw_data_hex, a tx JSON, or a whole Collect item. Get the recomputed txID, a decoded view of every field, and signature verification. |

## Collect flow

```mermaid
flowchart LR
    A["1 · Build<br/>form → raw_data"] --> B["2 · Hash<br/>sha256(protobuf)"]
    B --> C["3 · Sign<br/>secp256k1 ECDSA"]
    C --> D["4 · Export<br/>signed_tx + request"]
end
```

- **Build** assembles the unsigned raw_data object — `TransferContract` for native TRX, `TriggerSmartContract` with `transfer(address,uint256)` calldata for TRC-20. Requires a wallet, because `owner_address` lives inside raw_data.
- **Hash** encodes raw_data to protobuf bytes and computes `txID = SHA-256(raw_data_hex)`. Both the bytes and the 32-byte txID are shown and copyable.
- **Sign** runs secp256k1 ECDSA over the txID digest and attaches the 65-byte `r ‖ s ‖ v` signature. Success is reported only after the recovered signer matches the loaded wallet and the owner address.
- **Export** produces the final Collect item and the full `{ "items": [ item ] }` CollectRequest for `tron_scanner.TransactionService/Collect`. Both can be copied or downloaded.

Changing any Collect input clears the derived txID, signature and exports, so a stale payload cannot be sent by accident. Reset the flow at any time without losing the form values.

## How Tron differs from Ethereum

| | Ethereum | Tron |
| --- | --- | --- |
| Encoding | RLP | Protocol Buffers |
| Hash | `keccak256(RLP(tx))` | `SHA-256(protobuf(raw_data))` |
| What is signed | Transaction hash | The txID digest |
| Signature placement | Folded into the signed RLP | Outside raw_data, so the txID stays valid after signing |
| Address | `keccak256(pubkey)` → `0x…` | Same curve; `0x41 + 20 bytes` → Base58Check `T…` |

## Inspect

The Inspect tab accepts any layer:

- the **raw_data_hex** produced in step 2,
- an **unsigned or signed tx JSON**,
- or a whole **Collect item / CollectRequest** — `signed_tx` is unwrapped automatically.

Decoded output includes the contract type, ref block, timestamps, owner, recipient, amount, calldata and fee_limit. TRC-20 `transfer` calldata gets a readable recipient/amount callout. When a signature is present, the signer is recovered and checked against `owner_address`, and the declared `txID` is compared with the locally recomputed one — tampering is flagged. **Use current tx** pulls the current Collect payload directly from the other tab.

## Fees: burning TRX

This tool targets the "burn TRX" setup: no staking and no delegated bandwidth or energy. A Tron transaction has no bandwidth/energy fields; the network charges the sender automatically.

- **Native TRX** — a plain `TransferContract`. It consumes bandwidth only. With no staked bandwidth, the network burns TRX from the sender. Nothing extra to fill in.
- **TRC-20 (USDT)** — a `TriggerSmartContract`. It consumes energy. With no staked energy, the network burns TRX from the sender, and `fee_limit` (sun) is the maximum it may burn.

## Live block data

**Fetch latest block** calls `https://api.trongrid.io/wallet/getnowblock` and fills all four chain-validity fields in one click:

- `ref_block_bytes` — low 2 bytes of the block number (4 hex).
- `ref_block_hash` — bytes 8–16 of the block ID (16 hex).
- `timestamp` — block timestamp in milliseconds.
- `expiration` — block timestamp + 60 seconds.

Re-fetch right before broadcasting. An old reference block is rejected by the network, and without these fields the signature is well-formed but the transaction will not be accepted.

## Fields

| Field | Native TRX | TRC-20 | Role |
| --- | --- | --- | --- |
| `to_address` | Required | Required | Recipient. Base58 `T…` or `41…` hex. |
| `amount` | Required | Required | Atomic units. `1 TRX = 1000000` sun; USDT has 6 decimals, so `1 USDT = 1000000`. |
| `contract` | — | Required | Leave empty for native TRX. **Use USDT** fills `TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t`. |
| `fee_limit` | — | Required | Sun. Maximum TRX burned for energy. Default `100000000` (100 TRX). |
| `transaction_id` | Required | Required | Scanner item id, not the on-chain `txID`. |
| `token_id` | Required | Required | Scanner catalog token id. |
| `ref_block_bytes` / `ref_block_hash` | Required | Required | From a recent block — use the fetch button. |
| `expiration` / `timestamp` | Required | Required | Milliseconds. Filled by the block fetch. |

Final Collect item:

```json
{
  "transaction_id": "tx-1",
  "from_address": "T…",
  "to_address": "T…",
  "token_id": "1",
  "amount": "1000000",
  "signed_tx": "{ \"txID\": \"…\", \"raw_data\": { … }, \"raw_data_hex\": \"…\", \"signature\": [ \"…\" ] }"
}
```

`signed_tx` is the signed Tron transaction as a JSON string — the payload the scanner parses and later broadcasts.

## Quick start

Use the [live page](https://afsh7n.github.io/dev-tools-tron/), or run it locally:

```bash
git clone https://github.com/afsh7n/dev-tools-tron.git
cd dev-tools-tron
python3 -m http.server 8765
# open http://localhost:8765
```

Opening `index.html` directly also works. An internet connection is needed the first time to load ethers from the CDN, and for the live block fetch.

## Safety

- Use throwaway development keys only. Never use a production wallet.
- The private key lives in memory for the lifetime of the tab and is never uploaded, logged or stored.
- Downloading `wallet.json` writes the key to disk in plain text. Keep it out of git.
- Every signature is locally verified before it is presented as successful. Check the decoded fields and txID before signing anything.

## Project layout

```text
dev-tools-tron/
├── index.html   # the entire app: UI, styling, protobuf encoder/decoder and signing logic
└── README.md
```

Built with vanilla HTML/CSS/JS and [ethers v6](https://docs.ethers.org/v6/) — including a hand-rolled protobuf encoder/decoder and Base58Check. No framework, no bundler, no backend, no TronWeb dependency.
