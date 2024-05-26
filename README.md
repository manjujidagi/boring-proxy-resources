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
