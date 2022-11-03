# Tutorial tesnet Nibiru dari Airdropfind

<p style="font-size:14px" align="right">
<a href="https://t.me/airdropfind" target="_blank">Join Telegram Airdrop Finder<img src="https://user-images.githubusercontent.com/50621007/183283867-56b4d69f-bc6e-4939-b00a-72aa019d1aea.png" width="30"/></a>
</p>

<p align="center">
  <img height="auto" width="auto" src="https://raw.githubusercontent.com/bayy420-999/airdropfind/main/NavIcon.png">
</p>

## Referensi

- [Dokumen resmi Nibiru](https://docs.nibiru.fi/run-nodes/testnet/)

- [Tutorial dari kj89 (Script yang dia buat sangat membantu)](https://github.com/kj89/testnet_manuals/tree/main/nibiru)


## Jaringan tersedia

Anda dapat mencari tiap Nibiru testnet dalam tabel dibawah

| Network | Chain id | Description | Version | Status |
|---------|----------|-------------|---------|--------|
|Testnet|nibiru-testnet-1|Nibiru's default testnet|v0.15.0|Active|


## Persyaratan perangkat keras

### Persyaratan minimal

| Komponen | Persyaratan |
|----------|----------------------|
|CPU|4x CPUs; the faster clock speed the better|
|RAM|8GB RAM|
|Penyimpanan|100GB penyimpanan (SSD atau NVME)|
|Internet|Koneksi internet permanen (Saat testnet traffic hanya sedikit; 10Mbps sudah lebih dari cukup - untuk produksi(mainnet) 100Mbps dibutuhkan)|

### Recommend requirements

| Hardware | Minimum requirements |
|----------|----------------------|
|CPU|8x CPUs; the faster clock speed the better|
|RAM|64GB RAM|
|Penyimpanan|1TB penyimpanan (SSD atau NVME)|
|Internet|Koneksi internet permanen (Saat testnet traffic hanya sedikit; 10Mbps sudah lebih dari cukup - untuk produksi(mainnet) 100Mbps dibutuhkan)|

## Instalasi node Nibiru

Ada 2 opsi untuk menginstal node Nibiru (Instalasi otomatis dan manual) tetapi saya menyarankan untuk menggunakan instalasi otomatis saja karena instalasi manual tidak dibutuhkan. Instalasi otomatis juga mencegah anda dari kesalahan saat melakukan instalasi

Saya akan menggunakan skrip yang ditulis oleh kj89 untuk menginstal node secara otomatis. Jalankan perintah dibawah

```
wget -O nibiru.sh https://raw.githubusercontent.com/kj89/testnet_manuals/main/nibiru/nibiru.sh && chmod +x nibiru.sh && ./nibiru.sh
```

## Pasca instalasi

Setelah instalasi usai, jalankan perintah dibawah untuk memuat variabel ke dalam sistem

```
source $HOME/.bash_profile
```

Lalu pastikan node anda sudah tersinkronisasi dengan menjalankan perintah dibawah

```
nibid status 2>&1 | jq .SyncInfo
```

> Catatan: Jika `catching_up` == `false` itu artinya node anda telah tersinkronisasi, jika tidak maka anda perlu menunggu sampai proses sinkronisasi selesai sebelum melanjutkan ke langkah berikutnya. Anda juga dapat mempercepat proses sinkronisasi dengan menggunakan State-Sync yang disediakan oleh layanan PPNV.

## (OPSIONAL) Menggunakan State-Sync yang disediakan oleh layanan PPNV untuk mempercepat proses sinkronisasi node

Anda dapat melakukan State-Sync dalam beberapa menit dengan menjalankan perintah dibawah

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

## Memperbarui parameter blok waktu

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

## Membuat dompet

Untuk membuat dompet, gunakan perintah dibawah:

```
nibid keys add $WALLET
```

Lalu (opsional) pulihkan dompet anda menggunakan perintah dibawah

```
nibid keys add $WALLET --recover
```

## Simpan informasi dompet

Perintah dibawah akan secara otomatis menambahkan `wallet address` dan `valoper address` kedalam variabel sistem, jadi anda tidak perlu untuk menyalin wallet/valoper address setiap saat

```
NIBIRU_WALLET_ADDRESS=$(nibid keys show $WALLET -a)
NIBIRU_VALOPER_ADDRESS=$(nibid keys show $WALLET --bech val -a)
echo 'export NIBIRU_WALLET_ADDRESS='${NIBIRU_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export NIBIRU_VALOPER_ADDRESS='${NIBIRU_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Danai dompet anda

Anda perlu mendanai dompet anda sebagai syarat untuk membuat validator

```
curl -X POST -d '{"address": "'"$NIBIRU_WALLET_ADDRESS"'", "coins": ["10000000unibi","100000000000unusd"]}' https://faucet.testnet-1.nibiru.fi/
```

## Cek saldo dompet

Anda memerlukan 1 nibi (10000000 unibi) untuk membuat validator

Cek saldo anda menggunakan perintah dibawah:

```
nibid query bank balances $NIBIRU_WALLET_ADDRESS
```

> Jika saldo anda tidak muncul, itu kemungkinan terjadi karena node anda belum sepenuhnya tersinkronisasi. Tunggu sampai proses sinkronisasi selesai sebelum melanjutkan ke langkah berikutnya

## Membuat validator

Untuk membuat validator jalankan perintah dibawah

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


## Keamanan

### Setel kunci SSH untuk autentikasi

[Artikel dari Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-20-04)

### Keamanan dasar Firewall

Cek status ufw

```
sudo ufw Status
```

Setel default untuk mengizinkan koneksi keluar, tolak semua koneksi kecuali ssh dan 26656. Batasi percobaan masuk SSH

```
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh/tcp
sudo ufw limit ssh/tcp
sudo ufw allow ${NIBIRU_PORT}656,${NIBIRU_PORT}660/tcp
sudo ufw enable
```

## (OPSIONAL) Hitung waktu sinkronisasi

Skrip ini akan mengkur rata-rata blok yang di sinkronisasi per menit selama periode 5 menit. Respek kepada kj89 yang telah membuat skrip ini

```
wget -O synctime.py https://raw.githubusercontent.com/kj89/testnet_manuals/main/nibiru/tools/synctime.py && python3 ./synctime.py
```

## Perintah bermanfaat

### Menejemen layanan

Cek log

```
journalctl -fu nibid -o cat
```

Memulai layanan

```
sudo systemctl start nibid
```

Hentikan layanan

```
sudo systemctl stop nibid
```

Mulai ulang layanan

```
sudo systemctl restart nibid
```

#### Menejemen validator

Sunting validator

```
nibid tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$NIBIRU_CHAIN_ID \
  --from=$WALLET
```


Membebaskan validator

```
nibid tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$NIBIRU_CHAIN_ID \
  --gas=auto
```

Hapus node

> Setelah kamu menjalankan perintah ini, semua hal yang berkaitan dengan node akan dihapus dari server, tanggung resikonya sendiri!

```
sudo systemctl stop nibid
sudo systemctl disable nibid
sudo rm /etc/systemd/system/nibi* -rf
sudo rm $(which nibid) -rf
sudo rm $HOME/.nibid* -rf
sudo rm $HOME/nibiru -rf
sed -i '/NIBIRU_/d' ~/.bash_profile
```

### Informasi validator

Cek kunci validator

```
[[ $(nibid q staking validator $NIBIRU_VALOPER_ADDRESS -oj | jq -r .consensus_pubkey.key) = $(nibid status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

Dapatkan info validator

```
nibid status 2>&1 | jq .ValidatorInfo
```

Dapatkan list validator

```
nibid q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

Dapatkan list koneksi peers dengan id

```
curl -sS http://localhost:${NIBIRU_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

### Informasi node

Informasi sinkronisasi

```
nibid status 2>&1 | jq .SyncInfo
```

Informasi node

```
nibid status 2>&1 | jq .NodeInfo
```

Tampilkan id node

```
nibid tendermint show-node-id
```

### Operasi dompet

Dapatkan list dompet

```
nibid keys list
```

Pulihkan dompet

```
nibid keys add $WALLET --recover
```

Hapus dompet

```
nibid keys delete $WALLET
```

Dapatkan saldo dompet

```
nibid query bank balances $NIBIRU_WALLET_ADDRESS
```

Transfer saldo

```
nibid tx bank send $NIBIRU_WALLET_ADDRESS <TO_NIBIRU_WALLET_ADDRESS> 10000000unibi
```

### Voting 

```
nibid tx gov vote 1 yes --from $WALLET --chain-id=$NIBIRU_CHAIN_ID
```

### Staking, Delegasi and Upah

Delegasi stake

```
nibid tx staking delegate $NIBIRU_VALOPER_ADDRESS 10000000unibi --from=$WALLET --chain-id=$NIBIRU_CHAIN_ID --gas=auto
```

Mendelegasikan ulang dari validator ke validator lain

```
Redelegate stake from validator to another validator
```

Tarik semua upah

```
nibid tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$NIBIRU_CHAIN_ID --gas=auto
```

Tarik upah dengan komisi

```
nibid tx distribution withdraw-rewards $NIBIRU_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$NIBIRU_CHAIN_ID
```
