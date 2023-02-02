# Wireguard Router VM

1. Set static IP by editing `/etc/netplan/00-installer-config.yaml`
<pre>
network:
  ethernets:
    enp0s5:
      link-local: [ipv4]
      dhcp4: true
    enp0s6:
      link-local: [ipv4]
      dhcp4: no
      addresses:
       - 192.168.7.1/24
      nameservers:
        addresses: [127.0.0.1]
  version: 2
</pre>
2. `sudo netplan apply`

## Forwarding Settings
1. Add the followings to `/etc/sysctl.conf`:
<pre>
net.ipv6.conf.all.disable_ipv6=0
net.ipv6.conf.default.disable_ipv6=0
net.ipv6.conf.lo.disable_ipv6=0

net.ipv4.ip_forward=1
</pre>
2. `sudo sysctl -p`
3. `sudo iptables -A FORWARD -i enp0s6 -o wg0 -j ACCEPT`
4. `sudo iptables -A FORWARD -i wg0 -o enp0s6 -m state --state ESTABLISHED,RELATED -j ACCEPT`
5. `sudo iptables -t nat -A POSTROUTING -o wg0 -j MASQUERADE`
6. `sudo apt install iptables-persistent`
7. `sudo systemctl enable netfilter-persistent.service`

## Wireguard Installation
1.  `sudo ln -sf /lib/systemd/system/systemd-resolved.service /etc/systemd/system/dbus-org.freedesktop.resolve1.service`
2. `sudo systemctl restart systemd-resolved.service`
3. `sudo apt install wireguard`

4. Add the followings to `/etc/wireguard/wg0.conf`:
<pre>
[Interface]
PrivateKey = PRIVATE_KEY_FOR_WIREGUARD
Address = ADDRESS_FOR_WIREGUARD
DNS = DNS_FOR_WIREGUARD

[Peer]
PublicKey = PUBLIC_KEY_FOR_WIREGUARD
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = HOST_FOR_WIREGUARD
PersistentKeepalive = 25
</pre>
5. `sudo systemctl enable wg-quick@wg0.service`
6. `sudo systemctl daemon-reload`
7. `sudo systemctl start wg-quick@wg0`

## Dnsmasq Installation
1. `sudo apt install dnsmasq`
2. Add the followings to `/etc/dnsmasq.conf`:
<pre>
# Global settings
domain-needed
bogus-priv
no-resolv
expand-hosts
filterwin2k

# Upstream nameservers
# Wireguard DNS server
server=103.86.96.100

# domain name
domain=ztan.net
local=/ztan.net/

listen-address=127.0.0.1
listen-address=192.168.7.1

# DHCP options
dhcp-range=192.168.7.2,192.168.7.254,12h
dhcp-lease-max=100
dhcp-option=option:router,192.168.7.1
dhcp-option=option:dns-server,192.168.7.1
dhcp-option=option:netmask,255.255.255.0
</pre>
3. Add the followings to `sudo nano /etc/systemd/resolved.conf`:
<pre>
DNSStubListener=no
DNS=127.0.0.1
FallbackDNS=127.0.0.1
</pre>
