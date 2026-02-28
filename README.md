# Solana Token Creation Guide (Devnet)

A step-by-step journal of creating an SPL token on Solana devnet using the CLI.

---

## 1. Check Solana CLI Version

Verify that the Solana CLI is installed and check its version.

```bash
solana --version
```

Output:
```
solana-cli 2.2.20 (src:dabc99a5; feat:3073396398, client:Agave)
```

---

## 2. Generate a Wallet Keypair (Vanity Address)

Generate a keypair whose public key starts with `bos`. This will be our **wallet / payer** account — it holds SOL and pays for all transaction fees.

```bash
solana-keygen grind --starts-with bos:1
```

Output:
```
Searching with 10 threads for:
	1 pubkey that starts with 'bos' and ends with ''
Wrote keypair to bosGGN7wpSqCpgTfJtX5tV7xMyJJvk9xxSA8XAfq7cb.json
```

> 💡 `grind` searches for a vanity address. The `:1` means "find 1 match". The resulting keypair is saved to a JSON file.

---

## 3. Set the Wallet as Default Keypair

Tell the Solana CLI to use our new wallet keypair for all commands.

```bash
solana config set --keypair bosGGN7wpSqCpgTfJtX5tV7xMyJJvk9xxSA8XAfq7cb.json
```

Output:
```
Config File: /Users/rfiser/.config/solana/cli/config.yml
RPC URL: https://api.devnet.solana.com
WebSocket URL: wss://api.devnet.solana.com/ (computed)
Keypair Path: bosGGN7wpSqCpgTfJtX5tV7xMyJJvk9xxSA8XAfq7cb.json
Commitment: confirmed
```

---

## 4. Set the Network to Devnet

Point the CLI to Solana's devnet (test network with free SOL).

```bash
solana config set --url devnet
```

---

## 5. Verify Configuration

Double-check all settings are correct.

```bash
solana config get
```

Output:
```
Config File: /Users/rfiser/.config/solana/cli/config.yml
RPC URL: https://api.devnet.solana.com
WebSocket URL: wss://api.devnet.solana.com/ (computed)
Keypair Path: bosGGN7wpSqCpgTfJtX5tV7xMyJJvk9xxSA8XAfq7cb.json
Commitment: confirmed
```

---

## 6. Generate a Token Mint Keypair (Vanity Address)

Generate a separate keypair for the **token mint**. This address will uniquely identify our token on-chain. The `mnt` prefix is a nice visual reminder that this is a mint.

```bash
solana-keygen grind --starts-with mnt:1
```

Output:
```
Searching with 10 threads for:
	1 pubkey that starts with 'mnt' and ends with ''
Wrote keypair to mntNCCr1QMDJ1Z9YvydYwSfdvKVpRQ5RJXpF4sPRyXe.json
```

> 💡 **Wallet vs Mint**: The wallet (`bos...`) is *you* — it holds SOL and pays fees. The mint (`mnt...`) is *the token* — it defines the token's identity and properties.

---

## 7. Create the Token (with Metadata Extension)

Create a new SPL token using the **Token-2022** program (`TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb`) with the metadata extension enabled.

```bash
spl-token create-token \
  --program-id TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb \
  --enable-metadata \
  mntNCCr1QMDJ1Z9YvydYwSfdvKVpRQ5RJXpF4sPRyXe.json
```

Output:
```
Creating token mntNCCr1QMDJ1Z9YvydYwSfdvKVpRQ5RJXpF4sPRyXe under program TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb

Address:  mntNCCr1QMDJ1Z9YvydYwSfdvKVpRQ5RJXpF4sPRyXe
Decimals:  9
```

> 💡 **Token-2022** is the newer token program that supports extensions like on-chain metadata, transfer fees, etc. The `--enable-metadata` flag allocates space for metadata directly inside the mint account.

---

## 8. Initialize Token Metadata

Set the token's name, symbol, and metadata URI. The URI points to a JSON file with additional information (image, description, etc.).

```bash
spl-token initialize-metadata \
  mntNCCr1QMDJ1Z9YvydYwSfdvKVpRQ5RJXpF4sPRyXe \
  "Radu's Token" \
  "UDAR" \
  https://raw.githubusercontent.com/radufiser/token-command-line/refs/heads/main/metadata.json
```

> ⚠️ **Common mistake**: Running `update-metadata` before `initialize-metadata` will fail with `Error: Program(InvalidAccountData)`. You must **initialize** first, then you can **update** later.

---

## 9. Create a Token Account (ATA)

On Solana, SPL tokens **cannot** live directly in your wallet. Each token type needs its own dedicated **Token Account** (also called an Associated Token Account). Think of it as a "bucket" linked to your wallet that holds a specific token.

```bash
spl-token create-account mntNCCr1QMDJ1Z9YvydYwSfdvKVpRQ5RJXpF4sPRyXe
```

Output:
```
Creating account D2WrWLoq7tijJvAWfDAiZGkxWjYQeCu7iVqubHicES8A
```

> 💡 **Why is this needed?** Solana's architecture requires explicit account creation for parallelism and performance. Each token account is a separate on-chain account that:
> - Is **owned by** your wallet (`bos...7cb`)
> - Is **associated with** a specific mint (`mnt...yXe`)
> - Holds the token balance for that mint
> 
> This is fundamentally different from Ethereum, where ERC-20 contracts track balances internally.

---

## 10. Mint Tokens

Finally, mint 1001 tokens to the token account we just created.

```bash
spl-token mint mntNCCr1QMDJ1Z9YvydYwSfdvKVpRQ5RJXpF4sPRyXe 1001
```

Output:
```
Minting 1001 tokens
  Token: mntNCCr1QMDJ1Z9YvydYwSfdvKVpRQ5RJXpF4sPRyXe
  Recipient: D2WrWLoq7tijJvAWfDAiZGkxWjYQeCu7iVqubHicES8A
```

> 💡 The CLI automatically routes minted tokens to the Associated Token Account (`D2W...S8A`) that we created in the previous step.

---

## Key Addresses Reference

| Address | Role | Description |
|---------|------|-------------|
| `bosGGN7wpSqCpgTfJtX5tV7xMyJJvk9xxSA8XAfq7cb` | **Wallet / Payer** | Holds SOL, pays fees, acts as authority |
| `mntNCCr1QMDJ1Z9YvydYwSfdvKVpRQ5RJXpF4sPRyXe` | **Token Mint** | The token's identity on-chain (UDAR) |
| `D2WrWLoq7tijJvAWfDAiZGkxWjYQeCu7iVqubHicES8A` | **Token Account (ATA)** | Holds UDAR token balance for the wallet |

---

## Account Structure Diagram

```
Wallet: bosGGN7wpSqCpgTfJtX5tV7xMyJJvk9xxSA8XAfq7cb
├── Holds: SOL (native balance)
│
└── Token Account: D2WrWLoq7tijJvAWfDAiZGkxWjYQeCu7iVqubHicES8A
        ├── Linked to mint: mntNCCr1QMDJ1Z9YvydYwSfdvKVpRQ5RJXpF4sPRyXe
        └── Balance: 1001 UDAR tokens
```

---

*This document will be updated as we progress with our understanding of Solana tokens.*


---

## 11. Verify On-Chain with Solana Explorer

After completing all the steps above, you can verify everything on the **Solana Explorer** — a block explorer that lets you inspect accounts, tokens, and transactions on-chain.

### View the Token Mint

Inspect the token mint account to see its metadata (name, symbol, URI), supply, decimals, and authorities:

```
https://explorer.solana.com/address/mntNCCr1QMDJ1Z9YvydYwSfdvKVpRQ5RJXpF4sPRyXe?cluster=devnet
```

Here you can verify:
- **Token Name**: Radu's Token
- **Symbol**: UDAR
- **Total Supply**: 1001 tokens
- **Decimals**: 9
- **Program**: Token-2022 (`TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb`)
- **Mint Authority**: `bosGGN7wpSqCpgTfJtX5tV7xMyJJvk9xxSA8XAfq7cb` (your wallet)
- **Metadata URI**: Points to `metadata.json` in this repo

### View Wallet Token Holdings

Inspect your wallet to see all token accounts and their balances:

```
https://explorer.solana.com/address/bosGGN7wpSqCpgTfJtX5tV7xMyJJvk9xxSA8XAfq7cb/tokens?cluster=devnet
```

Here you can verify:
- Your wallet holds **1001 UDAR** tokens
- The tokens are in the Associated Token Account (`D2WrWLoq7tijJvAWfDAiZGkxWjYQeCu7iVqubHicES8A`)
- Your SOL balance and recent transactions

> 💡 **Tip**: Notice the `?cluster=devnet` parameter in the URLs. Without it, the explorer defaults to mainnet and won't find your accounts. Always make sure you're viewing the correct cluster!
