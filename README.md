# Preconfig 
   ```bash
brew install bitcoin
```
 ```bash
bitcoin-cli --version 
```

Bitcoin Core RPC client version v30.0.0

When you run brew install bitcoin, a bitcoin.conf file is not created by default. You typically have to create it yourself.
# Original Docs

[Charms](https://docs.charms.dev/guides/charms-apps/get-started/)

~/Library/Application Support/Bitcoin/bitcoin.conf
 ```bash
mkdir -p ~/Library/Application\ Support/Bitcoin
nano ~/Library/Application\ Support/Bitcoin/bitcoin.conf
```
This guide assumes a bitcoin node running with the following configuration

server=1
testnet4=1
txindex=1
addresstype=bech32m
changetype=bech32m
```
## Create alias 
   ```bash
alias b=bitcoin-cli
```
# Start the server
 ```bash
brew services start bitcoin
```
# Test if running 
 ```bash
bitcoin-cli getblockchaininfo
```

# Stop the server
 ```bash
bitcoin-cli stop
```


# Creating, Building, and Verifying a New App

Follow these steps to create, build, and verify a new app using the `charms` tool:

1. **Create a new app**:
   ```bash
   charms app new my-token
   ```

2. **Navigate to the app directory**:
   ```bash
   cd ./my-token
   ```

3. **Unset the `CARGO_TARGET_DIR` environment variable**:
   ```bash
   unset CARGO_TARGET_DIR
   ```

4. **Update dependencies**:
   ```bash
   cargo update
   ```

5. **Build the app and generate its verification key**:
   ```bash
   app_bin=$(charms app build)
   charms app vk "$app_bin"
   ```

   - The `charms app build` command compiles the app and outputs the path to the binary.
   - The `charms app vk` command generates and prints the verification key for the binary.

This process ensures your app is properly set up, built, and verified.

# Example: `ins` and `outs` Configuration

Below is an example configuration for `ins` and `outs` using `charms`:

```yaml
ins:
  - utxo_id: ${in_utxo_1}
    charms:
      $00:
        ticker: MY-TOKEN
        remaining: 100000

outs:
  - address: ${addr_1}
    charms:
      $01: 69420
  - address: ${addr_2}
    charms:
      $00:
        ticker: MY-TOKEN
        remaining: 30580
```

- **`ins`**: Defines the input UTXO and associated charms. In this example:
  - `utxo_id` references the input UTXO.
  - `charms` specifies the token (`ticker`) and its remaining amount.

- **`outs`**: Defines the output addresses and associated charms. In this example:
  - Each `address` specifies a recipient.
  - `charms` defines the token or value being sent to each address.

This configuration demonstrates how to manage token transfers using `charms`.

Usage: charms <COMMAND>

Commands:
  server       Charms API Server
  spell        Work with spells
  tx           Work with underlying blockchain transactions
  app          Manage apps
  wallet       Wallet commands
  completions  Generate shell completion scripts
  help         Print this message or the help of the given subcommand(s)

Options:
  -h, --help     Print help
  -V, --version  Print version