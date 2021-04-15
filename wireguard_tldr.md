##### Wireguard TLDR:

##### On server:

To be able to connect through your WireGuard server, you\'ll need to
enable packet forwarding. Only on WireGuard server and not necessary for
any clients.

` $ sudo nano /etc/sysctl.conf`\
` `

Then uncomment the following line:

` net.ipv4.ip_forward=1`\
` `

Then apply the new option with the command below, and it should output
the line you uncommented:

` $ sudo sysctl -p`

Generating private and public keys (as root):

` # cd /etc/wireguard`\
` `

Set the permissions for the directory (or make sure to: chmod 600
publickey privatekey wg0.conf):

` # umask 077`

Then with the required permissions set, generate a new key pair with the
command below.

` # wg genkey | tee privatekey | wg pubkey > publickey`\
` `

#### Generate server config:

` # nano /etc/wireguard/wg0.conf`\
` `\
` [Interface]`\
` PrivateKey = `<contents-of-SERVER-privatekey>\
` Address = 10.0.0.1/24`\
` PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE`\
` PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE`\
` ListenPort = 51820`

` [Peer]`\
` PublicKey = `<contents-of-CLIENT-publickey>\
` AllowedIPs = 10.0.0.2/32`

On router forward the listen port as UDP

#### Starting WireGuard and enabling it at boot:

` # wg-quick up wg0`\
` `

Check if it runs:

` # wg show`\
` `

To enable WireGuard to start automatically at system boot, also enable
the systemd service.

` # systemctl enable wg-quick@wg0`\
` `

##### Client configuration:

` $ sudo nano /etc/wireguard/wg0.conf`\
` `\
` [Interface]`\
` Address = 10.0.0.2/32`\
` PrivateKey = `<contents-of-CLIENT-privatekey>\
` DNS = 1.1.1.1`

` [Peer]`\
` PublicKey = `<contents-of-SERVER-publickey>\
` Endpoint = `<server-public-ip>`:51820`\
` #AllowedIPs = 0.0.0.0/0, ::/0  # Use for all traffic`\
` AllowedIPs = 10.0.0.0/24       # Only use for wireguard peers`

Start and enable service just like the server.
