# Relay interchain-queries using the new GO v2 Relayer

## 1. Download and build binaries
```
cd $HOME
git clone https://github.com/Stride-Labs/interchain-queries.git
cd interchain-queries
go build
sudo mv interchain-queries /usr/local/bin/icq
```

## 2. Make home dir for icq and create configurations file
```
cd $HOME && mkdir .icq
sudo tee $HOME/.icq/config.yaml > /dev/null <<EOF
chains:
  gaia-testnet:
    key: openbitlab
    chain-id: GAIA
    rpc-addr: http://stride.stake-take.com:46657
    grpc-addr: http://stride.stake-take.com:9490
    account-prefix: cosmos
    keyring-backend: test
    gas-adjustment: 1.2
    gas-prices: 0.001uatom
    key-directory: /root/.icq/keys
    debug: false
    timeout: 20s
    block-timeout: ""
    output-format: json
    sign-mode: direct
  stride-testnet:
    key: openbitlab
    chain-id: STRIDE-TESTNET-4
    rpc-addr: http://stride.stake-take.com:26657
    grpc-addr: http://stride.stake-take.com:9090
    account-prefix: stride
    keyring-backend: test
    gas-adjustment: 1.2
    gas-prices: 0.001ustrd
    key-directory: /root/.icq/keys
    debug: false
    timeout: 20s
    block-timeout: ""
    output-format: json
    sign-mode: direct
cl: {}
EOF
```

## 3. Import wallets
> NOTE: Please use the same wallet you have used for relayer task, as it is the only way to prove that icq runs on your behalf!
```
icq keys restore --chain stride-testnet wallet
icq keys restore --chain gaia-testnet wallet
```

## 4. Create icq service
```
sudo tee /etc/systemd/system/icqd.service > /dev/null <<EOF
[Unit]
Description=Interchain Query Service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which icq) run --debug
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## 5. Start icq service
```
sudo systemctl daemon-reload
sudo systemctl enable icqd
sudo systemctl restart icqd
```

## 6. Check icq logs
```
journalctl -u icqd -f -o cat
```

You will have to wait some time until you see some logs (5-15min) than you can check you transaction in the explorer

![image](https://user-images.githubusercontent.com/35404870/187937759-b2bd97a7-d680-4081-afee-545ab7680f96.png)
