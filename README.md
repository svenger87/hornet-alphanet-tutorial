# hornet-mainnet-tutorial #

## This tutorial applies to Linux distributions with APT package manager ##

#### Prepare essentials

```
apt install build-essential git nano
```
#### Install Golang

See https://golang.org/doc/install for install instructions.
```bash
cd /tmp
wget https://golang.org/dl/go1.16.4.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.16.4.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
```
Optional add the export to /etc/profile or $HOME/.profile


#### Checkout and build Hornet

```bash
cd /opt
git clone -b main --single-branch https://github.com/gohornet/hornet
cd hornet

./scripts/build_hornet_rocksdb_builtin.sh 
```

## Configure Hornet and Systemd Service
```bash
mkdir /opt/hornet-mainnet
cp /opt/hornet/hornet /opt/hornet-mainnet
```

```bash
cp /opt/hornet/config.json /opt/hornet-mainnet
cp /opt/hornet/peering.json /opt/hornet-mainnet
cp /opt/hornet/profiles.json /opt/hornet-mainnet
```

Create a systemd service file. Note that the following code needs to be copy/pasted completely to your terminal and hit Enter.
```bash
cat << EOF > /lib/systemd/system/hornet-mainnet.service
[Unit]
Description=Hornet Mainnet
After=network-online.target

[Service]
WorkingDirectory=/opt/hornet-mainnet
ExecStart=/opt/hornet-mainnet/hornet -c config.json
ExecReload=/bin/kill -HUP $MAINPID
TimeoutSec=infinity
KillMode=process
Restart=on-failure
RestartSec=100
Type=simple
User=root
Group=root
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=hornet-mainnet

[Install]
WantedBy=multi-user.target
EOF
```

Enable systemd servcie and start it.
```bash
systemctl enable hornet-mainnet.service
systemctl start hornet-mainnet && journalctl -u hornet-mainnet -f
```
If everything went fine hornet should start up. Cancel the log output with _CTRL + C_

Allow traffic through firewall (ufw), if configured.
```bash
ufw allow 8081/tcp
ufw allow 15600/tcp
```

### Dashboard

Your Dashboard should be available via your IP address / hostname on port 8081 (eg. http://127.0.0.1:8081)

#### Remote Access

If you want remote access edit `/opt/hornet-mainnet/config.json` and change

`bindAddress : 0.0.0.0:8081`

Generate your password hash and salt with

```bash
/opt/hornet-mainnet/hornet tool pwdhash
``` 

The section should look something like this:

```json
  "dashboard": {
    "bindAddress": "0.0.0.0:8081",
    "dev": false,
    "auth": {
      "sessionTimeout": "72h",
      "username": "admin",
      "passwordHash": "d2be30670649d661b3edb8969bbc525aa86d00f8914ea2423ad09870b5b0dd15",
      "passwordSalt": "0893a61b4c5a109149dc37c73fc3d0bf08ca54f0638b402f10f3c3eef376cb86"
    }
  },
```

#### Add Neighbors

The dashboad shows your Peer ID. 
You need the peer ID to generate your connection string which you share with your neighbors.
You can find neighbors via the community project http://nodesharing.wisewolf.de/ or the IOTA Discord.

Example:
```
/dns/testnet.hornetnode.com/tcp/15600/ 12D3KooWS7nyRgFjzgkethzi6SDdjmuAGooxDmnoLzyex7Lu4hKo
/ip4/80.58.56.41/tcp/15600/ 12D3KooWS7nyRgFjzgkethzi6SDdjmuAGooxDmnoLzyex7Lu4hKo
```
Neighbors are added via the Dashboard. Alternatively they can be added to the peering.json like in pre Chrysalis Hornet. But the format changed. 

Example:
```json
{
  "peers": [
    {
      "alias": "Node1",
      "multiAddress": "/ip4/85.10.52.22/tcp/15601/p2p/12D3KooWPNAjVAtFdbDzbtB3htTFaGDWQePBK2TSGZLx3wwrkA97"
    },
    {
      "alias": "Node2",
      "multiAddress": "/dns/node-1.testnetnode.com/tcp/15601/p2p/12D3KooWPJ2tQ1uCTSj6L6UNLarkbRBLqWvucot6UQsUreTxtvX3"
    }
  ]
}
```

Alias need to be entered in the order of your peer list.

## Updating a node

```bash
systemctl stop hornet-mainnet && cd /opt/hornet && git pull && scripts/build_hornet_rocksdb_builtin.sh && cp hornet /opt/hornet-mainnet && systemctl start hornet-mainnet
```

If the version contains breaking changes:

```bash
systemctl stop hornet-mainnet && cd /opt/hornet-mainnet && rm -rf mainnetdb && rm -rf snapshots && cd /opt/hornet && git pull && scripts/build_hornet_rocksdb_builtin.sh && cp hornet /opt/hornet-mainnet && systemctl start hornet-mainnet
```
## If you got your node running and like to buy me a beer :D

```iota1qz2szrdezvy4ar3dln77jy06xt7u99ug3vpyg0aqj34j6j2t2l632sh38wu```
