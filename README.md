This document consolidates and explains the steps for setting up a Bitcoin Core development environment on macOS (using Homebrew) and outlines the process for creating, building, and verifying an application using the **Charms** protocol.

This is a comprehensive guide for full-stack Bitcoin development, managing the node, wallets, and the application logic.

-----

````markdown
# 💻 Bitcoin & Charms Development Environment Setup (macOS)

This guide walks through setting up Bitcoin Core for Testnet4 development and introduces the key commands for building Charms applications, which enable programmable assets on Bitcoin.

## 🚀 Part 1: Bitcoin Core Setup

This section focuses on installing Bitcoin Core, configuring it for **Testnet4** (a newer Bitcoin test network), and managing the node via `bitcoin-cli`.

### 1. Preconfiguration (Install & Verify)

Install Bitcoin Core using the Homebrew package manager and verify the installed client version.

```bash
# Install Bitcoin Core via Homebrew
brew install bitcoin

# Verify the installed client version
bitcoin-cli --version 
# Expected Output: Bitcoin Core RPC client version v30.0.0 (or newer)
````

### 2\. Configure for Testnet4

By default, Homebrew does not create the necessary `bitcoin.conf` file. You must create it in the correct directory and configure it for Testnet4.

```bash
# 1. Create the necessary directory structure for Bitcoin Core config (macOS default path)
mkdir -p ~/Library/Application\ Support/Bitcoin

# 2. Open the configuration file in the nano editor
nano ~/Library/Application\ Support/Bitcoin/bitcoin.conf
```

**Configuration Content:**
Add the following lines to the `bitcoin.conf` file to enable server mode, Testnet4, transaction indexing, and the modern bech32m address types.

```
server=1              # Enables the RPC server for bitcoin-cli access
testnet4=1            # Runs the node on the Testnet4 chain
txindex=1             # Indexes all transactions (required for Charms and UTXO lookup)
addresstype=bech32m   # Default address type for new addresses (Taproot)
changetype=bech32m    # Default address type for change outputs
```

### 3\. Start & Test the Bitcoin Server

Start the `bitcoind` daemon, which is the backend process that downloads the Testnet4 blockchain and listens for RPC commands.

```bash
# Start the Bitcoin Core service using Homebrew (runs bitcoind in the background)
brew services start bitcoin

# Optional: Restart the service if you made changes to bitcoin.conf
# brew services restart bitcoin

# Test if the server is running and synchronized
# This command fetches the blockchain status.
bitcoin-cli getblockchaininfo
```

### 4\. Setup Command Alias

Create a simple alias to shorten the common `bitcoin-cli` command to `b`.

```bash
alias b=bitcoin-cli

# Example Usage:
# b getblockchaininfo
```

### 5\. Essential `bitcoin-cli` Commands

| Purpose | Command | Notes |
| :--- | :--- | :--- |
| **Stop Server** | `bitcoin-cli stop` | Safely shuts down the `bitcoind` daemon. |
| **Decode Transaction**| `bitcoin-cli decoderawtransaction "TX_HEX"`| Converts a raw hexadecimal transaction string into a human-readable JSON object. |
| **List Unspent** | `bitcoin-cli listunspent` | Lists all Unspent Transaction Outputs (UTXOs) available to spend in the default wallet. |
| **Get Address Info** | `bitcoin-cli getaddressinfo "ADDRESS"` | Provides details about a specific address (e.g., script type, derivation path). |
| **Check Balance (Secure)**| `bitcoin-cli getbalance "*" 6` | Shows the balance for all accounts that have **6 or more confirmations** (the secure standard). |

## 💰 Part 2: Wallet Management & Funding

### 1\. Wallet Creation & Selection

You must create and/or select a wallet for all transactional RPC commands.

```bash
# Create a new, descript-based wallet named "testwallet"
bitcoin-cli createwallet testwallet

# Create a legacy wallet (for general practice)
bitcoin-cli createwallet "MyNewHotWallet"

# List all wallets currently loaded in the daemon
bitcoin-cli listwallets
```

### 2\. Get Testnet Address (for Funding)

The `-rpcwallet` flag is mandatory when multiple wallets are loaded.

```bash
# Get a new Taproot address from the "testwallet"
bitcoin-cli -rpcwallet=testwallet getnewaddress

# Get a new address with a label from the "MyNewHotWallet"
bitcoin-cli -rpcwallet=MyNewHotWallet getnewaddress "MyFirstReceiveAddress"
```

### 3\. Get Testnet Funds

Testnet BTC (tBTC) has no real value. Use the following faucets to acquire funds for your addresses:

  * **Mempool.space Faucet:** `https://mempool.space/testnet4/faucet`
  * **Testnet4.dev Faucet:** `https://faucet.testnet4.dev`

### 4\. Check Wallet Balance

```bash
bitcoin-cli -rpcwallet=testwallet getbalance
```

## 🧱 Part 3: Charms App Development Workflow

This section outlines the initial commands for creating and inspecting a programmable asset (Charm) using the `charms` CLI tool.

### 1\. App Creation and Setup

These steps prepare your development environment and build the core application logic.

| Command | Purpose |
| :--- | :--- |
| `charms app new my-token` | **Creates a new Rust project** structure for a Charms app named `my-token`. |
| `cd ./my-token` | **Navigates** into the newly created app directory. |
| `unset CARGO_TARGET_DIR` | Ensures the build is not directed to a non-standard or temporary location, preventing build errors. |
| `cargo update` | Updates all Rust dependencies. |
| `app_bin=$(charms app build)` | **Compiles the Rust code** into a Wasm binary and stores the binary's path in the `app_bin` variable. |
| `charms app vk "$app_bin"` | **Generates the Verification Key (VK)** for the Wasm binary. The VK is the on-chain representation of your application's logic. |

### 2\. `charms` Command Structure

The main commands available in the `charms` CLI:

| Command | Description |
| :--- | :--- |
| `server` | Manages the Charms API Server. |
| `spell` | **Core Tool:** Used to create, validate, and manage **spells** (the declarative transaction instructions). |
| `tx` | Tools for working with underlying raw blockchain transactions. |
| `app` | **Core Tool:** Used for creating, building, and verifying application logic (Wasm). |
| `wallet` | Wallet-related commands (may integrate with Bitcoin Core or external signers). |

### 3\. Example `ins` and `outs` Configuration (Spell)

This YAML structure is used within a Charms spell to define how assets change hands in a transaction. This example shows an NFT being spent and fungible tokens being created and sent.

```yaml
ins:
  - utxo_id: ${in_utxo_1}
    charms:
      # Charm $00 is the reference NFT that holds the remaining supply state
      $00:
        ticker: MY-TOKEN
        remaining: 100000 # The NFT's current state (remaining mintable tokens)

outs:
  # Output 1: Sends a quantity of the new fungible token to address 1
  - address: ${addr_1}
    charms:
      $01: 69420 # Sends 69,420 MY-TOKEN

  # Output 2: Sends the NFT (Charm $00) to address 2 with an updated state
  - address: ${addr_2}
    charms:
      $00:
        ticker: MY-TOKEN
        remaining: 30580 # New state: 100000 (initial) - 69420 (minted) = 30580
```

  - **`ins`**: Defines the source UTXOs and the assets currently held within them (the state *before* the transaction).
  - **`outs`**: Defines the destination addresses and the assets that will be created or updated (the state *after* the transaction). This structure allows the application logic to verify the transition (e.g., ensuring `remaining` is correctly reduced by the minted amount).

<!-- end list -->

```
```