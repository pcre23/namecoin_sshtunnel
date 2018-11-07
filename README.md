# ssh tunnel to connect ncdns to namecoind

To get access to the namecoin blockchain from everywhere i use a virtual private server (vps). This way i have access to my domains from every computer by setting up **ncdns** to connect via ssh. To my opinion, ssh is better hardened than rpc. So the only open port needed is the sshd port 22 on the vps.

## Setting up the ncdns

**/etc/ncdns/ncdns.conf:**

```bash
namecoinrpcaddress="127.0.0.1:8336"
namecoinrpcusername="Anton"
namecoinrpcpassword="mypassword"
```
The namecoin address is pointing to localhost instead of the ip address of the vps.

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

### Create the ssh tunnel script

Add a script in */usr/local/bin/onbootnamecoinrpc.sh*

```bash
#!/bin/bash
autossh -N -f -M 10000 -L 8336:127.0.0.1:8336 -i /home/Anton/.ssh/id_rsa -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -o StrictHostKeyChecking=no Anton@myvps -p 22 &
```

Change the file mode bits to **555** ```sudo chmod 555 /usr/local/bin/onbootnamecoinrpc.sh```

### Create a systemd bootscript

This is needed to auto start the ssh tunnel on reboot.
A starting routine for systemd in */etc/systemd/system/sshtunnel.service* 

```bash
[Unit]
Description=SSH tunnel for namecoin rpc connection
After=network.target

[Service]
Type=oneshot
User=Anton
ExecStart=/usr/local/bin/onbootnamecoinrpc.sh
RemainAfterExit=yes

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