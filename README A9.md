# mZAR Remittance Prototype (BLCH9X2 Capstone, Assignment 9)

A simplified remittance platform built on our own blockchain. A sender pays rand into a simulated bank,
receives the same number of **mZAR** stablecoins (1 ZAR : 1 mZAR reserve rule), sends them to a recipient with a
signed transaction mined into a proof-of-work block, and the recipient redeems them for rand against a recorded
receipt. Every step leaves a record that anyone can verify (full-chain audit, Merkle inclusion proofs, a
light-client check and a browser explorer), and the whole flow is exposed through a small Flask payment API.

This is an educational prototype, not production software. The bank is simulated, there is a single operator and
no real money is involved. Limitations are listed at the end of this file, in the notebook and in the report.

**Team:** Sakhile Buthelezi, Palesa Letho, Maphoka Mahlomola, Samson Mashabane.

## What is in this repository

| File | Purpose |
|---|---|
| `A9_code_improved_v2.ipynb` | The whole prototype and the demo, in one Jupyter notebook |
| `requirements.txt` | Python packages the notebook needs |
| `README.md` | This file |
| `evidence.json`, `pow_timing.png` | Created in the working directory when the notebook runs (demo evidence and the proof-of-work timing chart) |

There are no separate `.py` modules. The wallets, ledger, blockchain, bank, payment service and API all live in
the notebook, in the numbered sections below.

## Requirements

- Python 3.10 or newer
- The packages in `requirements.txt`: Flask, PyNaCl (Ed25519 signatures), requests, pandas, matplotlib and JupyterLab

## How to run (reproducible demo)

1. Create and activate a virtual environment.

   ```bash
   python -m venv .venv
   source .venv/bin/activate          # Windows PowerShell: .venv\Scripts\Activate.ps1
   ```

2. Install the packages.

   ```bash
   pip install -r requirements.txt
   ```

3. Open the notebook and run everything from a clean state.

   ```bash
   jupyter lab A9_code_improved_v2.ipynb
   ```

   Then choose **Kernel > Restart and Run All**. A full run usually takes a minute or so, mostly the proof-of-work
   timing study.

4. The notebook starts the API on `http://127.0.0.1:5055` by itself and runs steps i to vi of the brief in order.
   The explorer is at `http://127.0.0.1:5055/explorer` while the server is running.

5. At the end the notebook writes `evidence.json` and `pow_timing.png` and **stops the server**. To keep the API
   and explorer available afterwards, run the notebook up to the last section and leave out the final cell.

If port 5055 is already in use, shut down other notebook kernels and run again.

## Reproducibility

The demo uses a fixed clock and wallet keys derived from the wallet names, and Ed25519 signatures are
deterministic. Transaction IDs, nonces and block hashes are therefore **identical on every run**. Only the mining
timings change from machine to machine.

| Item | Value |
|---|---|
| Reserve address | `8911be18f32f00dcdf7e949f701e116de484be67a8504e1f0896c23769548e1f` |
| On-ramp tx (block 1, nonce 7,663) | `9f4b12c94b30b0db142baa799a0b79c76b1034504ebbd777ae256e98e6618491` |
| Transfer tx (block 2, nonce 59,849) | `49ecdae540d323efd9e724a2925903a167f10d5ff49a171b52cfd708193a57d7` |
| Redeem tx (block 3, nonce 1,849) | `5270433f2cc0072f76c63d8e817b88364779b0c08891936b93fe63fd44e0e67a` |
| Receipt `RCPT-0001` hash | `aaa3d5576df545eedee0f247a12296c9eb701dae325c3ab7e9e0120075be615b` |

If your values differ, the usual cause is the `MZAR_RESERVE_SEED` environment variable (below).

**Private reserve key.** By default the reserve key is derived from the name "Reserve", so anyone can recompute it.
That is fine for a demo. To use a private key, set a secret before starting Jupyter:

```bash
export MZAR_RESERVE_SEED="choose-a-long-secret"      # Windows PowerShell: $env:MZAR_RESERVE_SEED="..."
```

All hashes then differ from the table above and from the report.

## The demo, step by step

| Brief step | Notebook | What happens |
|---|---|---|
| i. Create wallets | Section 9, step i | Wallets for Sipho (sender) and Lindiwe (recipient); a reserve wallet is created at start-up. Sipho has R5,000.00 in the simulated bank. |
| ii. On-ramp | Step ii | Sipho deposits R1,000.00 and receives 1,000.00 mZAR. The conversion event is recorded and the mint is mined into block 1. |
| iii. Transfer | Step iii | Sipho sends 750.00 mZAR to Lindiwe. The signed transfer is mined into block 2. |
| iv. Off-ramp | Step iv | Lindiwe redeems 750.00 mZAR. Settling before the block exists is refused. After mining (block 3) the reserve checks the Merkle proof, pays R750.00 and issues receipt `RCPT-0001`. |
| v. Verification and explorer | Step v, Section 10 | Full-chain audit, an independent check of the transfer, a light-client check from headers and a Merkle proof, and the text and HTML explorers. |
| vi. Failed transfers | Step vi | Eleven invalid attempts are all rejected with HTTP 400 and change nothing. |

End state: 250.00 mZAR in circulation, R250.00 in the reserve, Lindiwe holds R750.00, and the reserve reports a
backing ratio of 1.0 with the identity R = S + M + U holding.

The eleven rejected attempts are: a forged amount (bad signature), an overdraft, a replay, an unauthorised mint,
a second payout of the same redemption, a fractional-cent amount, a non-numeric amount (`NaN`), a wrong field type,
a reserve-signed mint with no deposit behind it, a wallet named `RESERVE`, and a redeem from an address with no
registered bank account.

**Extra tests (Section 11):** tampering with an old block is detected by the audit, a Merkle proof in a block with
four transactions, a proof-of-work timing study for difficulties 1 to 5 (the demo uses difficulty 4), a payment that
arrives while a block is being mined (none is lost), and a non-custodial redemption where the user keeps their own key.

## Notebook layout

| Section | Content |
|---|---|
| 1 | Money helpers: whole cents, strict amount parsing, hashing, canonical JSON |
| 2 | Wallets: Ed25519 keys (PyNaCl), addresses, signatures |
| 3 | Transactions: MINT, TRANSFER, REDEEM |
| 4 | Blocks: proof of work, Merkle root and proof |
| 5 | The ledger rules (`Ledger.check`) and the `Blockchain` |
| 6 | Bank, reserve, on-ramp and off-ramp (`PaymentService`) |
| 7 to 8 | The Flask payment API, and starting it |
| 9 | Demo script, steps i to vi |
| 10 | Explorer (text and HTML) |
| 11 | Extra tests |
| 12 to 13 | Saved evidence, shutdown and limitations |

## Payment API

The server listens on `127.0.0.1:5055` only. Amounts in requests are in rand; amounts in responses are in cents.
Requests must be JSON with the header `Content-Type: application/json`.

| Method and path | Purpose |
|---|---|
| `POST /wallets/new` | Create a wallet; returns the address only |
| `POST /wallets/register` | Link a client-held public key to a bank account (non-custodial) |
| `POST /onramp` | Pay rand in; a mint is queued (the rand is taken first) |
| `POST /payments/send` | Build, sign and submit a transfer for a demo wallet |
| `POST /transactions/new` | Submit a transaction signed elsewhere (refuses every MINT) |
| `POST /mine` | Mine the pending pool into a block |
| `POST /offramp` | Submit a redeem request |
| `POST /offramp/settle` | After confirmation, pay rand and issue a receipt |
| `GET /transactions/<id>` | Look up a transaction with its Merkle proof |
| `GET /chain`, `GET /chain/verify` | The full chain, and a full audit |
| `GET /balances`, `GET /reserve` | Balances, and the reserve checks (backing ratio, identity, mints match deposits) |
| `GET /explorer` | Human-readable HTML view (add `?tx=<id>` to highlight a payment) |
| `GET /nodes`, `POST /nodes/register` | Peer list (no consensus in this prototype) |

Example calls while the notebook server is running:

```bash
curl http://127.0.0.1:5055/chain/verify
curl http://127.0.0.1:5055/reserve
curl http://127.0.0.1:5055/balances

curl -X POST http://127.0.0.1:5055/payments/send \
     -H "Content-Type: application/json" \
     -d '{"from": "Sipho", "to": "Lindiwe", "amount": 10}'
curl -X POST http://127.0.0.1:5055/mine
```

The simulated bank balances are set up inside the notebook (`bank.open(...)`), not through the API, so a wallet
created over HTTP starts with R0.00 and cannot on-ramp until the notebook gives it a bank balance.

## Limitations

- One operator and one machine: no consensus, no incentives, no networking between nodes.
- The bank and the reserve are simulated, and the reserve backing is checked off-chain by the reserve agent
  (the chain cannot see the bank), so a real system would need independent attestation.
- Demo wallets are custodial and the custodial endpoints (`/onramp`, `/payments/send`, `/offramp`) have no
  authentication. The default reserve key is guessable; use `MZAR_RESERVE_SEED` outside a demo.
- Everything is held in memory and is lost when the kernel stops.
- No fees, FX, KYC/AML or privacy, and settlement across a real bank and a chain would need crash recovery.
