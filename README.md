# hornet-testnet-tutorial

# This tutorial applies to Linux distributions with APT package manager #

#### Prepare essentials ####

```
apt install build-essential git nano
```
#### Install Golang ####

See https://golang.org/doc/install

#### Checkout and build Hornet ####

```
cd /opt
git clone -b develop --single-branch https://github.com/gohornet/hornet
cd hornet

go build
```
# Configure Hornet and Systemd Service
```
mkdir /opt/hornet-testnet
cp /opt/hornet/hornet /opt/hornet-testnet
```

```
cp /opt/hornet/config_chrysalis_testnet.json /opt/hornet-testnet
cp /opt/hornet/peering.json /opt/hornet-testnet
cp /opt/hornet/profiles.json /opt/hornet-testnet
```
Create the service file. Note that the following code needs to be copy/pasted completely to your terminal and hit Enter.
```
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
```
systemctl enable hornet-testnet.service
systemctl start hornet-testnet && journalctl -u hornet-testnet -f
```
If everything went fine hornet should start up. Cancel the log output with CTRL + C

Allow traffic through firewall
```
ufw allow 8081/tcp
ufw allow 15600/tcp
```
Your Dashboard should be available via your IP address / hostname on port 8081 (eg. http://127.0.0.1:8081)
The dashboad shows your Peer ID. You need the peer ID to generate your connection string which you share with your neighbors.

Example:
```
/dns/testnet.hornetnode.com/tcp/15600/p2p/12D3KooWS7nyRgFjzgkethzi6SDdjmuAGooxDmnoLzyex7Lu4hKo
/ip4/80.58.56.41/tcp/15600/p2p/12D3KooWS7nyRgFjzgkethzi6SDdjmuAGooxDmnoLzyex7Lu4hKo
```
Neighbors are added to peering.json like in pre Chrysalis Hornet. But the format changed. 

Example:
```
{
  "p2p": {
    "peers": [
        "/dns/testnet.hornetnode.com/tcp/15600/p2p/12D3KooWS7nyRgFjzgkethzi6SDdjmuAGooxDmnoLzyex7Lu4hKo",
        "/ip4/80.58.56.41/tcp/15600/p2p/12D3KooWS7nyRgFjzgkethzi6SDdjmuAGooxDmnoLzyex7Lu4hKo"
    ],
    "peerAliases": ["testnet.hornetnode.com","80.598.56.41"
    ]
  }
}
```
Alias need to be entered in the order of your peer list.

# Updating a node
```
systemctl stop hornet-testnet && cd /opt/hornet && git pull && scripts/build_hornet.sh && cp hornet /opt/hornet-testnet && systemctl start hornet-testnet
```

If the version contains breaking changes:

```
systemctl stop hornet-testnet && cd /opt/hornet-testnet && rm -rf testnetdb && rm -rf snapshots && cd /opt/hornet && git pull && scripts/build_hornet.sh && cp hornet /opt/hornet-testnet && systemctl start hornet-testnet
```
#### If you got your node running and like to buy me a beer :D ####
CMO9GUCMEPEGPLKNSZJOUTJMWEDCZVIGCNFGHIKRJR9ZCKTTKQK9XDTRRFTMS9JQJAPSRCQDVIJDYBNNCWAQINV9N9
