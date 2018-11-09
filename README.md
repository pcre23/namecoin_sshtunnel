# ssh tunnel to connect ncdns to namecoind

To get access to the namecoin blockchain from everywhere i use a virtual private server (vps). This way i have access to my domains from every computer by setting up **ncdns** to connect via ssh. To my opinion, ssh is better hardened than rpc. So the only open port needed is the sshd port 22 on the vps.

```bash
+---------------------+                          +------------------+
|        myvps        | <======================> |      My PC       |
|                     |   (rpc)ssh tunnel(rpc)   |                  |
|  running namecoind  | <======================> |  running ncdns   |
+---------------------+                          +------------------+
```


## Setting up the ncdns

**/etc/ncdns/ncdns.conf:**

```bash
namecoinrpcaddress="127.0.0.1:8336"
namecoinrpcusername="Anton"
namecoinrpcpassword="mypassword"
```
The namecoin address is pointing to localhost instead of the ip address of the vps.

### Create a systemd bootscript to start the ncdns

A starting routine for systemd in */etc/systemd/system/ncdns.service* 

```bash
[Unit]
Description=ncdns namecoin domain name service *.bit toplevel domain.
ConditionPathExists=/home/Anton/go/bin
After=network.target

[Service]
Type=simple
User=Anton
Group=Anton
WorkingDirectory=/home/Anton/go/bin
ExecStart=/home/Anton/go/bin/ncdns

# prepare for syslog
PermissionsStartOnly=true
ExecStartPre=/bin/mkdir -p /var/log/ncdns
ExecStartPre=/bin/chown syslog:adm /var/log/ncdns
ExecStartPre=/bin/chmod 755 /var/log/ncdns
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=ncdns

[Install]
WantedBy=multi-user.target
```
Change the file mode bits to **644** ```sudo chmod 644 /etc/systemd/system/ncdns.service```

### Enable on systemd

```bash
systemctl daemon-reload
systemctl enable ncdns.service
systemctl start ncdns.service
systemctl status ncdns.service
```

## Setting up the ssh tunnel

### Install autossh
```bash
sudo apt-get install openssh-client autossh
```

### Generating ssh keys

```bash
ssh-keygen
ssh-copy-id Anton@myvps
```

### Create a systemd bootscript to start the ssh tunnel

This is needed to auto start the ssh tunnel on reboot.
A starting routine for systemd in */etc/systemd/system/sshtunnel.service* 

```bash
[Unit]
Description=SSH tunnel for namecoin rpc connection
After=network.target

[Service]
Environment="AUTOSSH_GATETIME=0"
ExecStart=/usr/bin/autossh -M 0 -o "ServerAliveInterval 30" -o "ServerAliveCountMax 3" -o "ExitOnForwardFailure=yes" -o "StrictHostKeyChecking=no" -NL 8336:127.0.0.1:8336 -i /home/Anton/.ssh/id_rsa Anton@myvps -p 22

[Install]
WantedBy=multi-user.target
```

Change the file mode bits to **644** ```sudo chmod 644 /etc/systemd/system/sshtunnel.service```

### Enable on systemd

```bash
systemctl daemon-reload
systemctl enable sshtunnel.service
systemctl start sshtunnel.service
systemctl status sshtunnel.service
```

https://markdownshare.com/view/48683fb4-9cd0-4bc9-9a9a-94b05d172ef9