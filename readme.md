# Wiki

Install Near core and Near-Cli

Before completing this guide, your must first create a wallet:

Shardnet: [wallet.shardnet.near.org](https://wallet.shardnet.near.org)

Mainnet: [wallet.near.org](https://wallet.near.org)

# Near-CLI and Near Core installing and Build

1. First we will upgrade the OS and dependencies

```bash
sudo apt update && sudo apt upgrade -y
```

1. Install developer tools, Node.js, and npm

`curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -`  

`sudo apt install build-essential nodejs`

`PATH="$PATH"`

## Install Near Cli

`sudo npm install -g near-cli`

Set the environment for your Near-Cli (shardnet or mainnet)

`export NEAR_ENV=shardnet`

You can make the env  persistent adding it to `~/.profile` or `~/.bashrc` (depending on the OS). 

## Installing the Node

Install Developers tools.

`sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python protobuf-compiler libssl-dev pkg-config clang llvm cargo`

Make sure python3 pip is install, if not proceed to install it

`sudo apt install python3-pip`

Install Rust and Cargo

`curl --proto '=https' --tlsv1.2 -sSf [https://sh.rustup.rs](https://sh.rustup.rs/) | sh`

`source $HOME/.cargo/env`

Clone the near core repository:

```bash
git clone https://github.com/near/nearcore
cd nearcore
git fetch
```

For Stake Wars you would need to check out a certain commit: [https://github.com/near/stakewars-iii/blob/main/commit.md](https://github.com/near/stakewars-iii/blob/main/commit.md) (ignore when in mainnet)

`git checkout <commit>`

Build binaries (ignore shardnet flag for mainnet)

`cargo build -p neard --release --features shardnet`

Create working directory

`./target/release/neard --home ~/.near init --chain-id shardnet --download-genesis`

Go to `~/.near` directory and change config.json - for shardnet you can copy the config.json found here [https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json](https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json) 

These are the parameters you usually have to modify:

`"tracked_shards": [0]`

`"archive": false`

`"boot_nodes": "<INSERT BOOTNODES>"`

Start the node

`./target/release/neard --home ~/.near run`

![Captura de Pantalla 2022-07-27 a la(s) 12.01.58.png](Captura_de_Pantalla_2022-07-27_a_la(s)_12.01.58.png)

## Create Validator

Link your wallet to your created account:

`near login`

![Captura de Pantalla 2022-07-27 a la(s) 12.11.21.png](Captura_de_Pantalla_2022-07-27_a_la(s)_12.11.21.png)

You must then go to the link provided in your console and grant access to your account.

![Captura de Pantalla 2022-07-27 a la(s) 12.13.14.png](Captura_de_Pantalla_2022-07-27_a_la(s)_12.13.14.png)

After this, you will need to create the validator key.

`near generate-key <pool_id>`

<pool_id> == xx.factory.shardnet.near WHERE xx is you pool name, for example senseinode.factory.sharednet.near

Move the newly created file to your `~/.near` directory.

`cp ~/.near-credentials/shardnet/YOUR_WALLET.json ~/.near/validator_key.json`

File must follow this pattern, make sure to change `private_key` for `secret_key` if necessary:

`{
"account_id": "xx.factory.shardnet.near",
"public_key": "ed25519:HeaBJ3xLgvZacQWmEctTeUqyfSU4SDEnEwckWxd92W2G",
"secret_key": "ed25519:****"
}`

Stop and start your validator node again, you should see the “validator” flag in the logs:

![Captura de Pantalla 2022-07-27 a la(s) 12.25.46.png](Captura_de_Pantalla_2022-07-27_a_la(s)_12.25.46.png)

# Register Staking Pool

The following commands will use senseinode.shardnet.near account and senseinode.factory.shardnet.near as the staking pool, you must modify these parameters and the public keys to create your own staking pools. 

### Create Staking Pool

```jsx
near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "senseinode", "owner_id": "senseinode.shardnet.near", "stake_public_key": "ed25519:EfTpUmWR369JqZVf7UnrFsyQVXQ3fhFvNqbhMiDZEGU2", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="senseinode.shardnet.near" --amount=30 --gas=300000000000000
```

### Stake

```jsx
near call senseinode.factory.shardnet.near deposit_and_stake --amount 200 --accountId senseinode.shardnet.near --gas=300000000000000
```

### Ping

```jsx
near call senseinode.factory.shardnet.near ping '{}' --accountId senseinode.shardnet.near --gas=300000000000000
```

### balances

```jsx
near view senseinode.factory.shardnet.near get_account_total_balance '{"account_id": "senseinode.shardnet.near"}'
```

### view staked balances

```jsx
near view senseinode.factory.shardnet.near get_account_staked_balance '{"account_id": "senseinode.shardnet.near"}'
```

### pause

```jsx
near call senseinode.factory.shardnet.near pause_staking '{}' --accountId senseinode.shardnet.near
```

### resume

```jsx
near call senseinode.factory.shardnet.near resume_staking '{}' --accountId senseinode.shardnet.near
```

### check validator info

```jsx
curl -s -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' [http://localhost:3030/](http://localhost:3030/)
```

### Monitor your validator

The following script is the base for a monitor tool that checks your validator status. Make sure to change the staking_pool variable to your own pool name.

```python
import requests as r
import json

url = 'http://localhost:3030'

data = json.dumps({"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": "latest"})

headers = {'content-type': 'application/json'}

res = r.post(url, headers = headers, data = data).json()

#input your staking pool
staking_pool = 'senseinode.factory.shardnet.near'

# check current validators
for i in res['result']['current_validators']:
    if i['account_id'] == staking_pool:
        print(i)
    else:
        continue

# check reason for kickout
for i in res['result']['prev_epoch_kickout']:
    if i['account_id'] == staking_pool:
        print(i)
    else:
        continue
```

Result example:

`{'account_id': 'senseinode.factory.shardnet.near', 'is_slashed': False, 'num_expected_blocks': 15, 'num_expected_chunks': 37, 'num_produced_blocks': 14, 'num_produced_chunks': 34, 'public_key': 'ed25519:EfTpUmWR369JqZVf7UnrFsyQVXQ3fhFvNqbhMiDZEGU2', 'shards': [0], 'stake': '2309034326111880772000000000'}`

For a staking pool in the current validator set you must make sure that `num_expected_blocks` and `num_expected_chunks` match the `num_produced_blocks` ad `num_produced_chunks`

# Other Tools

Ping script that needs to be run every epoch to update delegartors balance: [https://gist.github.com/PixelNoob/d9ad19ffb22d246e25955c1ac3065076](https://gist.github.com/PixelNoob/d9ad19ffb22d246e25955c1ac3065076)
