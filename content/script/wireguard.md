# Wireguard 

## install 

``` bash 

sudo apt update
sudo apt install wireguard -y
# forward packages 
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p

```

## Generate keys 

```bash 

#wg genkey | tee /etc/wireguard/private.key | wg pubkey | tee /etc/wireguard/public.key
#wg genkey | tee /etc/wireguard/mobile-private.key | wg pubkey > /etc/wireguard/mobile-public.key

#create keys for server
cd /etc/wireguard/
umask 077; wg genkey | tee privatekey | wg pubkey > publickey
#cat privatekey
#cat publickey
chmod 600 /etc/wireguard/privatekey

#create client keys
umask 077; wg genkey | tee privatekeyClient | wg pubkey > publickeyClient
#cat privatekey
#cat publickey
chmod 600 /etc/wireguard/privatekeyClient

```
<!---
### Für IPv6 
'''bash
printf echo "$(date +%s%N)""$(cat /var/lib/dbus/machine-id)" | sha1sum | cut -c 31 - 
'''
-->
## Copy config 

```
/etc/wireguard/wg0.conf
[Interface]
PrivateKey = base64_encoded_private_key_goes_here
Address = 10.8.0.1/24, fd24:609a:6c18::1/64
ListenPort = 51820
SaveConfig = true
```

or with cli magic: 

```bash 
echo "[Interface]
PrivateKey = ""$(tail /etc/wireguard/privatekey)""
Address = 127.31.0.1/24
ListenPort = 51820
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE; ip6tables -A FORWARD -i %i -j ACCEPT; ip6tables -t nat -A POSTROUTING -o eth0 -j MASQUERADE;
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE; ip6tables -D FORWARD -i %i -j ACCEPT; ip6tables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
SaveConfig = true


#[Peer]
#PublicKey = ""$(tail /etc/wireguard/mobile-public.key)""
#AllowedIPs = 10.8.0.2/24
" >> /etc/wireguard/wg0.conf
echo "[Interface] 
PrivateKey = ""$(tail /etc/wireguard/privatekeyClient)""
Address = 172.31.0.2/32 
DNS = 1.1.1.1

[Peer] 
PublicKey = ""$(tail /etc/wireguard/publickey)""
Endpoint = ""$(curl -s checkip.dyndns.org|sed -e 's/.*Current IP Address: //' -e 's/<.*$//')"":51820 
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25" >> /etc/wireguard/client.conf

```

## Add Client 

```bash 
wg-quick up wg0
wg set wg0 peer ""$(tail /etc/wireguard/publickeyClient)"" allowed-ips 172.31.0.2/32
wg-quick down wg0
wg-quick up wg0
```
<!---
## activate ipv4 

'''bash
sudo nano /etc/sysctl.conf
'''

and uncomment 

'''
net.ipv4.ip_forward=1
'''

and than load the new values with: 

'''bash 

sudo sysctl -p
'''

## try to ignore step 5 

## starting the wireguard server 

sudo systemctl enable wg-quick@wg0.service
sudo systemctl start wg-quick@wg0.service


## create mobile auth file 

'''bash
echo "[Interface]
PrivateKey = ""$(tail /etc/wireguard/mobile-private.key)""
Address = 10.8.0.1/24
ListenPort = 51820
DNS = 1.1.1.1

[Peer]
PublicKey = ""$(tail /etc/wireguard/public.key)""
Endpoint = ""$(curl -s checkip.dyndns.org|sed -e 's/.*Current IP Address: //' -e 's/<.*$//'
)"":51820
AllowedIPs = 0.0.0.0/0" >> /etc/wireguard/mobile.conf
'''
-->
## get create qr code 
``` bash 
sudo apt install qrencode -< 
qrencode -t ansiutf8 < /etc/wireguard/client.conf

```
