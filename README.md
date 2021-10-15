# Juno Smart Contract Deployment
 In this guide we'll build and deploy erc20 smart contract in Juno testen lucina
 
 # Instalation 
Rust and Junod [installation page.](https://docs.junochain.com/smart-contracts/installation)

At first we need to install [rustup](https://rustup.rs/).

```
$ rustup default stable
$ cargo version
$ rustup update stable

$ rustup target list --installed
$ rustup target add wasm32-unknown-unknown
```

# Builsing client Juno for testnet
We'll use go 1.16.3 for junod binary compilation 

```
$ clone juno repo 
$ git clone https://github.com/CosmosContracts/Juno.git && cd Juno

> build juno executable

$ make install
$ which junod
```

# Next step we need to install CosmWasm

```
$ git clone https://github.com/CosmWasm/cosmwasm-examples
$ cd cosmwasm-examples
$ git fetch
$ git checkout v0.10.0 
$ cd contracts/erc20
```

# Compile now

```
$ rustup default stable
$ cargo wasm
```

We need to limit our gas usage. So we do

```
$ sudo docker run --rm -v "$(pwd)":/code \
    --mount type=volume,source="$(basename "$(pwd)")_cache",target=/code/target \
    --mount type=volume,source=registry_cache,target=/usr/local/cargo/registry \
    cosmwasm/rust-optimizer:0.11.4
```

This will create cw_erc20.wasm in 'artifacts' catalog.

# Deploying
Need to mention that you need to know the address of active rpc node for transactions to go through. While writing this guide i was using https://rpc.juno.giansalex.dev:443/

```
$ cd artifacts
$ junod tx wasm store cw_erc20.wasm  --from <your-key> --chain-id=<chain-id> --gas auto --node  https://rpc.juno.giansalex.dev:443/
```
In the output you need to find contract ID it looks like this `{"key":"code_id","value":"6"} ` 

# Instalation.

After you uploaded contract you need to install it. As example we'll use "Aqua coin"

#  JSON generation with arguments 

To generate JSON you need to use node REPL or jq.
To install node `$ sudo apt install nodejs`
`$ node` and press Enter

Here you will need to adjust data for one you want

```js
> const initHash = {
  name: "Aqua Coin",
  symbol: "aqua",
  decimals: 83,
  initial_balances: [
    { address: " juno1ucdafm0s4uka6lnmaa7dl7tt489l2wfv3s3wfl", amount: "12345678000"},
  ]
};
< undefined
> JSON.stringify(initHash);
< '{"name":"Aqua Coin","symbol":"aqua","decimals":83,"initial_balances":[{"address":" juno1ucdafm0s4uka6lnmaa7dl7tt489l2wfv3s3wfl","amount":"12345678000"}]}'
```

# Next step is to create contract example
Please note  that the --amount is used to initialise the new account associated with the contract. 83 is the value of $CODE_ID.

```js
junod tx wasm instantiate 83 \
    '{"name":"Aqua Coin","symbol":"AQUA","decimals":83,"initial_balances":[{"address":" juno1ucdafm0s4uka6lnmaa7dl7tt489l2wfv3s3wfl ","amount":"12345678000"}]}' \
    --amount 50000ujuno  --label "Aquacoin erc20" --from Maxon --chain-id lucina --gas auto -y --node https://rpc.juno.giansalex.dev:443/
```
Check the output and find contract address e.g. juno1a2b ....

For ease to use we can create an argument for contract address

` CONTRACT_ADDR=[address of the contract]`  and check `echo $CONTRACT_ADDR`

Now you can query it

`junod query wasm contract $CONTRACT_ADDR`

Now you can check that the contract has assigned the right amount to the self-delegate address:

`junod query wasm contract-state smart <contract-address> '{"balance":{"address":"<validator-self-delegate-address>"}}'`

How to query and execute commands 

`junod tx wasm execute [contract_addr_bech32] [json_encoded_send_args] --amount [coins,optional] [flags]`

You can omit --amount if not needed for execute calls. In this case, your command will look something like:

`junod tx wasm execute <contract-addr> '{"transfer":{"amount":"200","owner":"<validator-self-delegate-address>","recipient":"<recipient-address>"}}' --from <your-key> --chain-id <chain-id>`

In the folder contracts/erc20 within cosmwasm-examples, for example, you can see the schemas:

````tree schema
schema
├── allowance_response.json
├── balance_response.json
├── constants.json
├── execute_msg.json
├── instantiate_msg.json
└── query_msg.json
````
Hope you enjoyed the process and it went smooth.Enjoy your meme coin!

