This is a [Charms](https://charms.dev) app.

It is a simple fungible token managed by a reference NFT. The NFT has a state that specifies the remaining total supply of the tokens available to mint. If you control the NFT, you can mint new tokens.

NOTE: you may need to install Wasm WASI P1 support:

```sh
rustup target add wasm32-wasip1
```

Build with:
```sh
cargo update
app_bin=$(charms app build)
```

The resulting Wasm binary will show up at `./target/wasm32-wasip1/release/my-token.wasm`.

Get the verification key for the app with:
```sh
charms app vk $app_bin
```

Test the app with a simple NFT mint example:

```sh
export app_vk=$(charms app vk)

# set to a UTXO you're spending (you can see what you have by running `b listunspent`)
export in_utxo_0="d8fa4cdade7ac3dff64047dc73b58591ebe638579881b200d4fea68fc84521f0:0"
# This command generates a unique identifier (app_id) for the app based on the input UTXO (in_utxo_0). The SHA-256 hash ensures that the identifier is unique and deterministic for the given UTXO.

export app_id=$(echo -n "${in_utxo_0}" | sha256sum | cut -d' ' -f1)

export addr_0=""

prev_txs=

cat ./spells/mint-nft.yaml | envsubst | charms spell check --prev-txs=${prev_txs} --app-bins=${app_bin}
```
