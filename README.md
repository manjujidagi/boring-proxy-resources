# boring-proxy-resources
All links and usueful resources for boring proxy

# SSH into a local computer via boring-proxy tunnel

https://forum.indiebits.io/t/how-to-forward-ssh-port-example-just-in-time-feature-request/56/4?u=anders

Say your boringproxy server is running at example.com and you want to be able to do ssh -p 8022 example.com in order to ssh into a specific client.

Here’s the steps:

Create a tunnel with the following values:

Tunnel Port: 8022
Client Name: whatever client you want to ssh into
TLS Termination: Server
Client Port: 22 or whatever the client machine is running sshd on
Allow External: TCP true

That should do it. You’ll need to make sure port 8022 is open on the server. You can use whatever port you want, just make sure it’s open and matches what you use when you try to connect ssh.

Note: The key was setting TLS Termination to Server, because that opens up the code path for the client to support raw TCP connections. I need to change this to be a setting on the tunnel itself, so it will work with server or client termination. Or maybe add an option to disable TLS altogether for SSH tunnels since the domain and TLS will never be used.

EDIT: For anyone looking at this later, make sure you also have GatewayPorts clientspecified in your sshd_config file on the boringproxy server machine.


# Running boring-proxy as a service

https://github.com/boringproxy/boringproxy/issues/108

Hi,

A quick howto on starting the boringproxy server as a service. Probably 101 stuff but took me a bit of fiddling to get right.
I guess boringproxy users was a easy /boring experience so hope this helps. This got it working for me but I am no expert so probably improvements can be made.

On a vps running debian 11. Assumes a password set for root (su passwd) so su works.

#created a user for boring proxy:
#add user bop
adduser bop
#for better way see comment below about useradd -M -r -s /bin/false -c "boringproxy system user" bop

#become user and cd to home dir
su bop
cd

#install boring proxy in home dir /home/bop/boringproxy-linux-x86_64 server -for example
#get BP
curl -LO https://github.com/boringproxy/boringproxy/releases/download/v0.6.0/boringproxy-linux-x86_64
#Make executable
chmod +x boringproxy-linux-x86_64
#Allow binding to ports 80 and 443
su
/usr/sbin/setcap cap_net_bind_service=+ep boringproxy-linux-x86_64
exit

#Create an .ssh dir for boring proxy:

mkdir ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

#Configure boring proxy to run as a service as bop:

su
cd /etc/systemd/system/
touch boringproxy.service
vi boringproxy.service

#in the newly created file add:

```
[Unit]
Description=Boring Proxy service
After=network.target
StartLimitIntervalSec=0
[Service]
Type=simple
Restart=always
RestartSec=1
User=bop
WorkingDirectory=/home/bop
ExecStart=/home/bop/boringproxy-linux-x86_64 server -admin-domain bop.domainname.com

[Install]
WantedBy=multi-user.target
```

#Note: without WorkingDirectory boring proxy can't load the config

#test buy starting and stopping
systemctl start boringproxy
#access https://bop.domainname.com
systemctl stop boringproxy

#to start automatically:
systemctl enable boringproxy
#reboot and test - done

#To debug follow your log:
tail -f /var/log/daemon.log

#token will be in:
cat /home/bop/boringproxy_db.json
