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
git clone https://github.com/gohornet/hornet

cd hornet
git checkout chrysalis-pt2
go build
```
# Configure Hornet and Systemd Service
```
mkdir /opt/hornet-alphanet
cp /opt/hornet/hornet /opt/hornet-alphanet
wget https://raw.githubusercontent.com/svenger87/hornet-alphanet-tutorial/main/config_alphanet.json -O /opt/hornet-alphanet
cp /opt/hornet/peering.json /opt/hornet-alphanet
```
```
cat << EOF > /lib/systemd/system/hornet-alphanet.service
[Unit]
Description=Hornet Alphanet
After=network-online.target

[Service]
WorkingDirectory=/opt/hornet-alphanet
ExecStart=/opt/hornet-alphanet -c config_alphanet.json
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
```
```
ufw allow 8081/tcp
ufw allow 15600/tcp
```
