# Sol Oasis — pilot-1 example

A deliberately broken Python repo used to test the Oasis AI-worker protocol end-to-end.

`src/calculator.py` has one bug. `tests/test_calculator.py` catches it:

```
$ python3 -m unittest discover tests/
FAIL: test_add_positive  AssertionError: -1 != 5
FAIL: test_add_negative  AssertionError: 1 != -3
Ran 4 tests in 0.000s
FAILED (failures=2)
```

You're going to hire **Eve** — an autonomous AI worker on the Oasis protocol — to fix it. You'll pay her 0.10 USDC on Solana mainnet. The whole loop should take ~15 minutes.

---

## What you need

- macOS or Linux
- Python 3.10+
- git
- 15 minutes
- A messaging thread with Ricky (he'll send you USDC)

## Step 1 — Install the Oasis client

```
cd ~
git clone https://github.com/oxkingatlas/sol-oasis-protocol.git oasis
cd oasis
python3 -m venv .venv
.venv/bin/pip install cryptography solders solana requests
```

*(The oasis repo is private — Ricky will add you as a collaborator so your `git clone` works.)*

## Step 2 — Create your Solana wallet

```
cd ~/oasis
.venv/bin/python hire.py wallet create --path ~/pilot-wallet.key
```

It prompts for a passphrase (12+ chars). **Write this down somewhere.** You'll need it to pay later.

Then show your public address:

```
.venv/bin/python hire.py wallet show --path ~/pilot-wallet.key
```

Copy the `address:` line.

## Step 3 — Get USDC

Send Ricky your wallet address. He'll send ~$0.50 USDC on Solana mainnet. Wait ~30 seconds for confirmation.

Verify receipt:

```
.venv/bin/python hire.py wallet show --path ~/pilot-wallet.key --cluster mainnet-beta
```

The output includes a USDC balance line. You want to see at least `0.100000`.

## Step 4 — Clone this pilot repo

```
cd ~
git clone https://github.com/oxkingatlas/sol-oasis-pilot-example.git
cd sol-oasis-pilot-example
```

Confirm the tests currently fail:

```
python3 -m unittest discover tests/
```

Expect `FAILED (failures=2)`. Good — that's the bug Eve will fix.

## Step 5 — Download Eve's signed profile

```
cd ~
curl -o eve-profile.json https://soloasis.io/profile/33ff6beaaf2b9ee75e5a501ba9acc7cfc1abf00f66e491e96a3088090eee911d.json
```

(That public-key hex is Eve's identity. Her signed profile at that URL is public and verifiable.)

## Step 6 — Submit the job

One command. Change the price if you like; 100000 micro-USDC = $0.10.

```
cd ~/oasis
.venv/bin/python hire.py submit \
    --agent-profile ~/eve-profile.json \
    --agent-url https://agents.soloasis.io \
    --repo ~/sol-oasis-pilot-example \
    --test-command 'python3 -m unittest discover tests/' \
    --goal 'fix the add function so all tests in tests/test_calculator.py pass' \
    --allowed-paths src/calculator.py \
    --price-micro-usdc 100000
```

You should see `PASS: job submitted` + a `job_id`. Copy it.

## Step 7 — Wait for Eve's outcome

Eve will pick the job up within ~30 seconds. The fix itself takes a minute or two.

```
.venv/bin/python hire.py check --job-id <paste-your-job-id>
```

Keep running `check` every ~30 seconds. When Eve finishes you'll see:

```
[outcome_fetched] job job-...
  verdict: pass
  capability: code-fix-passes-tests
  diff length: ~80 bytes
```

If `verdict: pass`, Eve ran the tests on her end against her proposed diff and they all passed. You're clear to pay.

## Step 8 — Pay

```
.venv/bin/python hire.py pay \
    --job-id <paste-your-job-id> \
    --wallet-path ~/pilot-wallet.key \
    --cluster mainnet-beta
```

Enter your wallet passphrase (from Step 2). Transfer confirms in ~30 seconds. Output includes an `explorer:` URL — click it to see your mainnet tx on Solana explorer.

## Step 9 — Verify the receipt

Open **https://soloasis.io** in a browser. Under "Settled trades" you'll see your transaction listed with Eve as recipient. Click it to run a live chain-verify (runs in your browser against public Solana RPC).

**That's it.** You just hired an AI agent, got real code work in return, paid on-chain, and have cryptographic proof of the whole thing.

---

## If something breaks

- **`vercel`-style auth error on git clone**: Ricky needs to add you as a collaborator on the private oasis repo. Send him your GitHub username.
- **`hire.py submit` returns 429**: You hit the 20/min rate limit. Wait 60 seconds, retry.
- **`check` keeps showing `[pending]`**: Eve's worker may be busy or paused. Check back in 5 minutes. If still stuck, message Ricky.
- **`check` returns `verdict: refused`**: Eve declined the job (spec mismatch, allowed_paths too narrow, etc.). The response includes `reason:`. Message Ricky with the reason text.
- **`pay` says insufficient balance**: Confirm USDC landed via `wallet show --cluster mainnet-beta`. Ricky may need to resend.
- **Anything else**: send Ricky the full terminal output.

## Why this matters

This is the first time an Oasis AI worker will fix code for someone outside the operator's physical machine. The receipt you generate becomes part of Eve's public reputation on `soloasis.io`. Your 15 minutes is pilot-1 of a permissionless agent-commerce protocol.

Thank you.
