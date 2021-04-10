# hornet-testnet-tutorial #

## This tutorial applies to Linux distributions with APT package manager ##

#### Prepare essentials

```
apt install build-essential git nano
```
#### Install Golang

See https://golang.org/doc/install for install instructions.

#### Checkout and build Hornet

```bash
cd /opt
git clone -b develop --single-branch https://github.com/gohornet/hornet
cd hornet

./scriots/build_hornet.sh 
```

## Configure Hornet and Systemd Service
```bash
mkdir /opt/hornet-testnet
cp /opt/hornet/hornet /opt/hornet-testnet
```

```bash
cp /opt/hornet/config_chrysalis_testnet.json /opt/hornet-testnet
cp /opt/hornet/peering.json /opt/hornet-testnet
cp /opt/hornet/profiles.json /opt/hornet-testnet
```

Create a systemd service file. Note that the following code needs to be copy/pasted completely to your terminal and hit Enter.
```bash
cat << EOF > /lib/systemd/system/hornet-testnet.service
[Unit]
Description=Hornet Testnet
After=network-online.target

[Service]
WorkingDirectory=/opt/hornet-testnet
ExecStart=/opt/hornet-testnet/hornet -c config_chrysalis_testnet.json
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
SyslogIdentifier=hornet-testnet

[Install]
WantedBy=multi-user.target
EOF
```

Enable systemd servcie and start it.
```bash
systemctl enable hornet-testnet.service
systemctl start hornet-testnet && journalctl -u hornet-testnet -f
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

If you want remote access edit `/opt/hornet-testnet/config_chrysalis_testnet.json` and change

`bindAddress : 0.0.0.0:8081`

Generate your password hash and salt with

```bash
/opt/hornet-testnet/hornet tool pwdhash
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
systemctl stop hornet-testnet && cd /opt/hornet && git pull && scripts/build_hornet.sh && cp hornet /opt/hornet-testnet && systemctl start hornet-testnet
```

If the version contains breaking changes:

```bash
systemctl stop hornet-testnet && cd /opt/hornet-testnet && rm -rf testnetdb && rm -rf snapshots && cd /opt/hornet && git pull && scripts/build_hornet.sh && cp hornet /opt/hornet-testnet && systemctl start hornet-testnet
```
## If you got your node running and like to buy me a beer :D

```CMO9GUCMEPEGPLKNSZJOUTJMWEDCZVIGCNFGHIKRJR9ZCKTTKQK9XDTRRFTMS9JQJAPSRCQDVIJDYBNNCWAQINV9N9```
