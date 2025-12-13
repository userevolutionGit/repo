This is a detailed Markdown guide for setting up, building, and testing a programmable fungible token application using the **Charms** protocol on Bitcoin, where the token's total supply is managed by the state of a reference NFT.

You can copy the source code of this Markdown block directly.

-----

````markdown
# ⚛️ Charms App Tutorial: NFT-Managed Fungible Token

This tutorial guides you through the process of building, inspecting, and testing a decentralized application (dApp) using the **Charms** protocol. This specific app implements a simple **Fungible Token** whose total remaining supply is dynamically controlled by the state of a reference **NFT**.

This is a [Charms](https://charms.dev) application, allowing for rich, programmable assets directly on the Bitcoin UTXO model.

## 🛠️ Setup and Build

### 1. Install Wasm WASI P1 Target

The Charms apps are compiled into WebAssembly (Wasm) bytecode that conforms to the WASI (WebAssembly System Interface) Preview 1 standard. You must install the required Rust target.

```sh
# This command adds the necessary target for cross-compilation to Wasm/WASI.
rustup target add wasm32-wasip1
````

### 2\. Build the Charms Application

Use `cargo` to build the Rust application and then use the `charms app build` command to compile and finalize the Wasm binary.

```sh
# Update dependencies first
cargo update

# Build the app and capture the path to the resulting Wasm binary
# Note: The 'charms app build' command handles the target for you.
app_bin=$(charms app build)
```

The resulting Wasm binary will show up at:

`./target/wasm32-wasip1/release/my-token.wasm` (or similar, depending on the project name).

### 3\. Get the Verification Key (VK)

The Verification Key (VK) is derived from the Wasm binary and is essential for on-chain verification using zkVMs.

```sh
# The $app_bin variable holds the path to the compiled Wasm file
charms app vk $app_bin

# Export the VK for use in subsequent commands (assuming the output is captured)
# In a typical script, you would run:
export app_vk=$(charms app vk $app_bin)
```

## 🧪 Test: Minting the Reference NFT

This section demonstrates how to test the application by simulating the initial transaction that **mints the reference NFT** and simultaneously defines the **initial state** of the fungible token (e.g., setting the initial remaining supply).

### 1\. Define Environment Variables

We set necessary environment variables to simulate a Bitcoin transaction.

| Variable | Value | Description |
| :--- | :--- | :--- |
| `app_vk` | (Output of `charms app vk`) | The Verification Key for the app. |
| `in_utxo_0`| `d8fa4cdade7ac3dff64047dc73b58591ebe638579881b200d4fea68fc84521f0:0`| A single UTXO being spent to fund and create the Charm. |
| `app_id` | (SHA-256 of `in_utxo_0`) | The unique, deterministic identifier for the Charm/App. |
| `addr_0` | `tb1p3w06fgh64axkj3uphn4t258ehweccm367vkdhkvz8qzdagjctm8qaw2xyv`| The Bitcoin Testnet address receiving the resulting Charm. |
| `prev_txs` | (Raw Hex of UTXO's transaction) | The raw transaction hex that created `in_utxo_0`. Required for context. |

```sh
# 1. Export the Application's Verification Key
export app_vk=$(charms app vk $app_bin)

# 2. Set the input UTXO (UTXO being spent)
export in_utxo_0="d8fa4cdade7ac3dff64047dc73b58591ebe638579881b200d4fea68fc84521f0:0"

# 3. Generate the deterministic App ID (Charm ID)
# The App ID is the SHA-256 hash of the UTXO identifier (txid:vout), which ensures the ID is unique to the Charm's creation transaction.
export app_id=$(echo -n "${in_utxo_0}" | sha256sum | cut -d' ' -f1)

# 4. Set a receiving address (Testnet P2TR)
export addr_0="tb1p3w06fgh64axkj3uphn4t258ehweccm367vkdhkvz8qzdagjctm8qaw2xyv"

# 5. Define the raw hex of the transaction that created $in_utxo_0
# NOTE: This is a single string and should not contain line breaks.
export prev_txs="02000000000101a3a4c09a03f771e863517b8169ad6c08784d419e6421015e8c360db5231871eb0200000000fdffffff024331070000000000160014555a971f96c15bd5ef181a140138e3d3c960d6e1204e0000000000002251207c4bb238ab772a2000906f3958ca5f15d3a80d563f17eb4123c5b7c135b128dc0140e3d5a2a8c658ea8a47de425f1d45e429fbd84e68d9f3c7ff9cd36f1968260fa558fe15c39ac2c0096fe076b707625e1ae129e642a53081b177294251b002ddf600000000"
```

brew install gettext

### 2\. Run the Spell Check

The `mint-nft.yaml` file contains the "spell" (the Charm metadata and transaction instructions) for minting the reference NFT. The `envsubst` command injects your environment variables (like `app_id` and `addr_0`) into the YAML template before the `charms spell check` command processes it.

```sh
# Execute the spell check command
# It verifies that the spell (YAML) is valid for the app logic (Wasm) given the inputs (prev_txs).
cat ./spells/mint-nft.yaml | envsubst | charms spell check --prev-txs=${prev_txs} --app-bins=${app_bin}
```

```sh
cat ./spells/mint-nft.yaml | envsubst
```

If successful, this command validates that the Wasm application logic correctly handles the minting of the reference NFT and initializes the total supply state.

```
```

FAUCET : 
https://coinfaucet.eu/en/btc-testnet4/


bitcoin-cli -testnet4  -rpcwallet=testwallet getbalance

To import the private key
bitcoin-cli importprivkey <WIF_private_key> "imported_label" true
