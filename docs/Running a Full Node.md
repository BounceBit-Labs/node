# Running a Full Node



### Hardware requirements

Suggested minimum hardware to run a Bouncebit Testnet Full node:

| CPUs                        | RAM   | Storage           |
|-----------------------------|-------|-------------------|
| 8 physical cores / 16 vCPUs | 32 GB | 500 GB NVMe drive |

### Software requirements

Bouncebit recommends running Testnet Full nodes on Linux. Bouncebit supports the Ubuntu and Debian distributions. 

Make sure the architecture is [x86_64]([x86-64 - Wikipedia](https://en.wikipedia.org/wiki/X86-64))

```
Ubuntu, 22.04 LTS, amd64
```

### Network requirements

Make sure that the specified ports for the following types of nodes are accessible from the public internet.
| Node Type | Port  | Protocol |
| --------- | ----- | -------- |
| Fullnode / Validator  | 26656 | TCP      |


## Configure a Full node

### Preparation

**Create workspace directory**

```
cd $WORKSPACE
mkdir -p bouncebit/bin
mkdir -p bouncebit/config
```

**Create work user**

```
useradd -m -s /bin/bash bouncebit
```


### Download the bbcored binary

You can download the latest binaries from [Release](https://github.com/BounceBit-Labs/node/releases/tag/v0.12.0)

```
cd $WORKSPACE

wget https://github.com/BounceBit-Labs/node/releases/download/v0.12.0/bbcored
chmod +x bbcored

ln -s $WORKSPACE/bbcored /usr/local/bin/bbcored
```

### Init Bouncebit Testnet Network

```
cd $WORKSPACE
bbcored init [moniker] -o --chain-id bouncebit_6000-1 --home $WORKSPACE
```

### Install Configuration Files

Bouncebit Chain configuration files can be downloaded from  [Testnet file repo](https://github.com/BounceBit-Labs/testnet/tree/main/network_files)

And move the downloaded file to the following path.

```
config/app.toml
config/client.toml
config/config.toml
config/genesis.json
```
If you are running as a validator and seeking to save disk space, we recommend simply using the following settings.

**app.toml**
```
pruning = "custom"

# These are applied if and only if the pruning strategy is custom.
pruning-keep-recent = "100"
pruning-interval = "10"
```
**config.toml**
```
indexer = "null"
```
### Running Node via systemd

We recommend using Systemd to manage Bouncebit service and check logs. 

**Setting up systemd unit file**

You can create a Systemd unit file at the following location `/etc/systemd/system/bouncebit.service`:

```
[Unit]
Description=BounceBit Testnet
After=network-online.target

[Service]
Type=simple
User=bouncebit
KillMode=process
Restart=on-failure
RestartSec=3s
ExecStart=bbcored start --log_level info --home <WORKSPACE> --chain-id bouncebit_6000-1 --moniker <YOUR_NODE_NAME>

[Install]
WantedBy=multi-user.target
```

**Start the service**

```
systemctl daemon-reload
systemctl start bouncebit.service
systemctl enable bouncebit.service
```

**View logs**

```
journalctl -o cat -f -u bouncebit
```
### StateSync
With fast sync a node is downloading all of the data of an application from genesis and verifying it. With state sync your node will download data related to the head or near the head of the chain and verify the data. This leads to drastically shorter times for joining a network.


Fetch the latest block height.

```bash
curl -s http://18.140.239.235:26657/block  | jq -r '.result.block.header.height'

959266
```

Fetch the trust hash of the latest block height.

```bash
curl -s "http://18.140.239.235:26657/block?height=959266" | jq -r '.result.block_id.hash'

85CD36BAB77E5338966B6033BB8D2C3701385D424F02650319F8A62A32D3E9A7
```

Enable state-sync, add RPC servers
```bash

[statesync]

enable = true
rpc_servers = "18.140.239.235:26657,18.143.160.245:26657"
trust_height = 959266
trust_hash = "85CD36BAB77E5338966B6033BB8D2C3701385D424F02650319F8A62A32D3E9A7"
```

## Monitoring

Monitor your Full node using the instructions at Logging, Tracing, Metrics, and Observability.

The default metrics port is `26660`. To change the port, edit your config/config.toml file.

```
prometheus = true

# Address to listen for Prometheus collector(s) connections
prometheus_listen_addr = ":26660"
```


