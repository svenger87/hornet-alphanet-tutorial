# hornet-alphanet-tutorial

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
git clone -b chrysalis-pt2 --single-branch https://github.com/gohornet/hornet
cd hornet

go build
```
# Configure Hornet and Systemd Service
```
mkdir /opt/hornet-alphanet
cp /opt/hornet/hornet /opt/hornet-alphanet
```
We need to use a preconfigured Config file because the source file is missing some config values.
Feel free to not download it but compare the changes and edit the file yourself.
https://github.com/svenger87/hornet-alphanet-tutorial/blob/main/config_alphanet.json

```
wget https://raw.githubusercontent.com/svenger87/hornet-alphanet-tutorial/main/config_alphanet.json -O /opt/hornet-alphanet/config_alphanet.json
cp /opt/hornet/peering.json /opt/hornet-alphanet
cp /opt/hornet/profiles.json /opt/hornet-alphanet
```
Create the service file. Note that the following code needs to be copy/pasted completely to your terminal and hit Enter.
```
cat << EOF > /lib/systemd/system/hornet-alphanet.service
[Unit]
Description=Hornet Alphanet
After=network-online.target

[Service]
WorkingDirectory=/opt/hornet-alphanet
ExecStart=/opt/hornet-alphanet/hornet -c config_alphanet.json
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
SyslogIdentifier=hornet-alphanet


[Install]
WantedBy=multi-user.target
EOF
```
```
systemctl enable hornet-alphanet.service
systemctl start hornet-alphanet && journalctl -u hornet-alphanet -f
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
/dns/alphanet.hornetnode.com/tcp/15600/p2p/12D3KooWS7nyRgFjzgkethzi6SDdjmuAGooxDmnoLzyex7Lu4hKo
/ip4/80.58.56.41/tcp/15600/p2p/12D3KooWS7nyRgFjzgkethzi6SDdjmuAGooxDmnoLzyex7Lu4hKo
```
Neighbors are added to peering.json like in pre Chrysalis Hornet. But the format changed. 

Example:
```
{
  "p2p": {
    "peers": [
        "/dns/alphanet.hornetnode.com/tcp/15600/p2p/12D3KooWS7nyRgFjzgkethzi6SDdjmuAGooxDmnoLzyex7Lu4hKo",
        "/ip4/80.58.56.41/tcp/15600/p2p/12D3KooWS7nyRgFjzgkethzi6SDdjmuAGooxDmnoLzyex7Lu4hKo"
    ],
    "peerAliases": ["alphanet.hornetnode.com","80.598.56.41"
    ]
  }
}
```
Alias need to be entered in the order of your peer list.

#### Updating a node ####

Now it´s getting really dirty.
If a new version of hornet is released copy and paste the following one-liner

```
systemctl stop hornet-alphanet && cd /opt/hornet && git pull && go build && cp hornet /opt/hornet-alphanet && systemctl start hornet-alphanet
```

If the version contains breaking changes:

```
systemctl stop hornet-alphanet && cd /opt/hornet && rm -rf alphanetdb && rm -rf snapshots && git pull && go build && cp hornet /opt/hornet-alphanet && systemctl start hornet-alphanet
```
