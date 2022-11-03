# Nibiru testnet guide by Airdropfind

<p style="font-size:14px" align="right">
<a href="https://t.me/airdropfind" target="_blank">Join Telegram Airdrop Finder<img src="https://user-images.githubusercontent.com/50621007/183283867-56b4d69f-bc6e-4939-b00a-72aa019d1aea.png" width="30"/></a>
</p>

<p align="center">
  <img height="100" height="auto" src="https://raw.githubusercontent.com/bayy420-999/airdropfind/main/NavIcon.png">
</p>

## References

- [Nibiru official documents](https://docs.nibiru.fi/run-nodes/testnet/)

- [kj89 guide (His/Her script really helping)](https://github.com/kj89/testnet_manuals/tree/main/nibiru)


## Available Networks

You can find each Nibiru testnet below

| Network | Chain id | Description | Version | Status |
|---------|----------|-------------|---------|--------|
|Testnet|nibiru-testnet-1|Nibiru's default testnet|v0.15.0|Active|


## Hardware requirements

### Minimum requirements

| Hardware | Minimum spec |
|----------|----------------------|
|CPU|4x CPUs; the faster clock speed the better|
|RAM|8GB RAM|
|Storage|100GB of storage (SSD or NVME)|
|Internet|Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)|

### Recommend requirements

| Hardware | Recommend spec |
|----------|----------------------|
|CPU|8x CPUs; the faster clock speed the better|
|RAM|64GB RAM|
|Storage|1TB of storage (SSD or NVME)|
|Internet|Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)|

## Setup Nibiru node

There are 2 options to setup Nibiru node (Automatic and manual installation) but i recommend you to use automatic installation because manual installation is not necessary. Automatic installation also prevent you from wrong doing

I'll stick to script written by kj89 for automatic installation. Run this commands:

```
wget -O nibiru.sh https://raw.githubusercontent.com/kj89/testnet_manuals/main/nibiru/nibiru.sh && chmod +x nibiru.sh && ./nibiru.sh
```

## Post installation

After installation finished, run this commands to load variables into system

```
source $HOME/.bash_profile
```

And then make sure that your node synchronized by running commands below:

```
nibid status 2>&1 | jq .SyncInfo
```

> Note: if `catching_up` == `false` that mean the node has been fully synchronized otherwise you need to wait until node fully synchronized before continue to next step. You can also speed up your node synchronization by using State-Sync provided by PPNV service

## (OPTIONAL) Using State-Sync provided by PPNV service to speed up node synchronization

You can state sync your node in minutes by running commands below:

```
SNAP_RPC="http://rpc.nibiru.ppnv.space:10657"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
#if there are no errors, then continue
sudo systemctl stop nibid
nibid tendermint unsafe-reset-all --home $HOME/.nibid --keep-addr-book
peers="ff597c3eea5fe832825586cce4ed00cb7798d4b5@rpc.nibiru.ppnv.space:10656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.nibid/config/config.toml
sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.nibid/config/config.toml
sudo systemctl restart nibid
sudo journalctl -u nibid -f --no-hostname -o cat
```

## Update block time parameters

```
CONFIG_TOML="$HOME/.nibid/config/config.toml"
sed -i 's/timeout_propose =.*/timeout_propose = "100ms"/g' $CONFIG_TOML
sed -i 's/timeout_propose_delta =.*/timeout_propose_delta = "500ms"/g' $CONFIG_TOML
sed -i 's/timeout_prevote =.*/timeout_prevote = "100ms"/g' $CONFIG_TOML
sed -i 's/timeout_prevote_delta =.*/timeout_prevote_delta = "500ms"/g' $CONFIG_TOML
sed -i 's/timeout_precommit =.*/timeout_precommit = "100ms"/g' $CONFIG_TOML
sed -i 's/timeout_precommit_delta =.*/timeout_precommit_delta = "500ms"/g' $CONFIG_TOML
sed -i 's/timeout_commit =.*/timeout_commit = "1s"/g' $CONFIG_TOML
sed -i 's/skip_timeout_commit =.*/skip_timeout_commit = false/g' $CONFIG_TOML
```

## Create wallet

To create new wallet, use commands below:

```
nibid keys add $WALLET
```

And then (optional) recover your wallet using this commands:

```
nibid keys add $WALLET --recover
```

## Save wallet info

This commands below will automatically add `wallet address` and `valoper addressr` into system variables, so you don't need to copy-paste wallet/valopar address everytime

```
NIBIRU_WALLET_ADDRESS=$(nibid keys show $WALLET -a)
NIBIRU_VALOPER_ADDRESS=$(nibid keys show $WALLET --bech val -a)
echo 'export NIBIRU_WALLET_ADDRESS='${NIBIRU_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export NIBIRU_VALOPER_ADDRESS='${NIBIRU_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Fund your wallet

You need to fund your wallet as requirements to create validator

```
curl -X POST -d '{"address": "'"$NIBIRU_WALLET_ADDRESS"'", "coins": ["10000000unibi","100000000000unusd"]}' https://faucet.testnet-1.nibiru.fi/
```

## Check wallet balances

You need atleast 1 nibi (10000000 unibi) to create validator

Check your balances using this commands:

```
nibid query bank balances $NIBIRU_WALLET_ADDRESS
```

> If your balances didn't show up, that's probably because your node isn't synced yet. wait until it finish synchronizing before continue to next step

## Create validator

To create validator run commands below:

```
nibid tx staking create-validator \
  --amount 2000000unibi \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(nibid tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $NIBIRU_CHAIN_ID
```


## Security

### Setup SSH Key for authentication

[Article from Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-20-04)

### Basic Firewall security

Check ufw status:

```
sudo ufw Status
```

Set the default to allow outgoing connections, deny all incoming except ssh and 26656. Limit SSH login attempts

```
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh/tcp
sudo ufw limit ssh/tcp
sudo ufw allow ${NIBIRU_PORT}656,${NIBIRU_PORT}660/tcp
sudo ufw enable
```

## (OPTINAL) Calculate synchronization time

This script will measure avarage block that being synchronized per minutes for 5 minutes periode, props to kj89 that made this script

```
wget -O synctime.py https://raw.githubusercontent.com/kj89/testnet_manuals/main/nibiru/tools/synctime.py && python3 ./synctime.py
```

## Usefull commands

### Service Management

Check logs

```
journalctl -fu nibid -o cat
```

Start service

```
sudo systemctl start nibid
```

Stop service

```
sudo systemctl stop nibid
```

Restart service

```
sudo systemctl restart nibid
```

#### Validator management

Edit validator

```
nibid tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$NIBIRU_CHAIN_ID \
  --from=$WALLET
```


Unjail validator

```
nibid tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$NIBIRU_CHAIN_ID \
  --gas=auto
```

Delete Node

> After you run this commands all node related thing will be removed from your server. Do with your own risk!

```
sudo systemctl stop nibid
sudo systemctl disable nibid
sudo rm /etc/systemd/system/nibi* -rf
sudo rm $(which nibid) -rf
sudo rm $HOME/.nibid* -rf
sudo rm $HOME/nibiru -rf
sed -i '/NIBIRU_/d' ~/.bash_profile
```

### Validator info

Check validator key

```
[[ $(nibid q staking validator $NIBIRU_VALOPER_ADDRESS -oj | jq -r .consensus_pubkey.key) = $(nibid status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

Get validator info

```
nibid status 2>&1 | jq .ValidatorInfo
```

Get validator list

```
nibid q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

Get current connected peers list with ids

```
curl -sS http://localhost:${NIBIRU_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

### Node info

Synchronization info

```
nibid status 2>&1 | jq .SyncInfo
```

Node info

```
nibid status 2>&1 | jq .NodeInfo
```

Show node id

```
nibid tendermint show-node-id
```

### Wallet operations

Get wallet list

```
nibid keys list
```

Recover wallet

```
nibid keys add $WALLET --recover
```

Delete wallet

```
nibid keys delete $WALLET
```

Get wallet balances

```
nibid query bank balances $NIBIRU_WALLET_ADDRESS
```

Transfer funds

```
nibid tx bank send $NIBIRU_WALLET_ADDRESS <TO_NIBIRU_WALLET_ADDRESS> 10000000unibi
```

### Voting

```
nibid tx gov vote 1 yes --from $WALLET --chain-id=$NIBIRU_CHAIN_ID
```

### Staking, Delegation and Rewards

Delegate stake

```
nibid tx staking delegate $NIBIRU_VALOPER_ADDRESS 10000000unibi --from=$WALLET --chain-id=$NIBIRU_CHAIN_ID --gas=auto
```

Redelegate from validator to another validator 

```
Redelegate stake from validator to another validator
```

Withdraw all rewards

```
nibid tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$NIBIRU_CHAIN_ID --gas=auto
```

Withdraw rewards with commision

```
nibid tx distribution withdraw-rewards $NIBIRU_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$NIBIRU_CHAIN_ID
```
