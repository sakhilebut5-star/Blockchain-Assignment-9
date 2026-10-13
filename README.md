# mZAR Remittance Prototype (BLCH9X2 Capstone)

Simplified remittance rails: ZAR -> simulated stablecoin (mZAR, 1:1 reserve) -> signed transfer in a
proof-of-work block -> redemption to ZAR, with chain verification, Merkle inclusion proofs, a small
HTTP payment API and an explorer. Educational prototype - not production software.

## Requirements
Python 3.10+ and one dependency: `pip install cryptography`  (HTTP server uses only the standard library)

## Run the reproducible demo (steps i-vi of the brief)
    python demo.py > demo_output.txt      # starts API on 127.0.0.1:8765, runs the full script, exits
    python bench.py                       # PoW difficulty study -> pow_bench.csv

Outputs: demo_output.txt, chain_export.json, explorer.html. Wallet addresses are reproducible
(seeded keys); txids, nonces and timings differ per run because timestamps are part of each transaction.

## Run the API interactively
    python mzar_api.py                    # then open http://127.0.0.1:8765/explorer
    curl -X POST localhost:8765/wallets -d '{"name":"Sender"}'
    curl localhost:8765/verify

## Files
- mzar_core.py  wallets, transactions, Merkle tree, PoW blocks, Chain validation, Bank/reserve ledger
- mzar_api.py   payment API + explorer (JSON/HTML/text)
- demo.py       demo script incl. rejected transfers and tamper detection
- bench.py      difficulty study

## API
POST /wallets, /onramp, /payments, /transactions, /mine, /offramp; GET /balance/<name>, /tx/<txid>, /chain, /verify, /explorer
