# Cloud Wireguard VPN using Docker

## Create DigitalOcean account

## Create Ubuntu 20.04 Droplet: 

Get started with a droplet -> Ubuntu 20.04 -> Basic -> Regular Intel with SSD -> $5/mo -> went with New York datacenter 3 -> VPC Network: default-nyc3 -> no additional addons -> password: used generated password -> hostname: ubuntu-wireguard-vpn-docker

## Install Docker

Go to droplet console, then install docker.

Install necessary tools:

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
```

Add docker key:

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Add docker repo:

Running Ubuntu 20.04 (LTS) x64 so use 64bit OS repo:

```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

Switch to correct repo:

```
apt-cache policy docker-ce
```

Install Docker:

```
sudo apt install docker-ce -y
```

Install Docker-Compose

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

Set execute permission:

```
sudo chmod +x /usr/local/bin/docker-compose
```

Done!

## Install Wireguard

Run these:

```
mkdir -p ~/wireguard/
mkdir -p ~/wireguard/config/
nano ~/wireguard/docker-compose.yml
```

Paste the following into the docker-compose.yml file:

```
version: '3.8'
services:
  wireguard:
    container_name: wireguard
    image: linuxserver/wireguard
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Chicago
      - SERVERURL=159.65.41.24
      - SERVERPORT=51820
      - PEERS=pc1,laptop1,phone1
      - PEERDNS=auto
      - INTERNAL_SUBNET=10.0.0.0
    ports:
      - 51820:51820/udp
    volumes:
      - type: bind
        source: ./config/
        target: /config/
      - type: bind
        source: /lib/modules
        target: /lib/modules
    restart: always
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
```

Modify the TZ, SERVERURL, and PEERS fields.

I used `TZ=America/Chicago`.

I found my SERVERURL (aka server IP address) on the DigitalOcean dashboard as `159.65.41.24` and thus used `SERVERURL=159.65.41.24`.

I have a pc, a laptop, and a phone, so I changed the PEERS field to `PEERS=pc1, laptop1, phone1`.

Now hit `CTRL` + `X`, `Y`, `ENTER` to save and exit the file.

Start Wireguard:

```
cd ~/wireguard/
docker-compose up -d
```

Connect your phone to Wireguard

```
docker-compose logs -f wireguard
```

Shows execution log and QR codes of Wireguard VPN connection settings.

On iOS: open app store and download WireGuard app. Open Wireguard VPN connection application on phone and click on `Add a tunnel`, then `Create from QR code`. I used the phone1 QR code. Check IP address with IPLeak.net. Activate the VPN, then refresh the page.

On laptop: Windows: download windows installer from wireguard website

Go to `~/wireguard/configs/{username}`. For me, I went to `~/wireguard/configs/peer_laptop1`.

Next I need to read and copy the `.conf` file contents:

```
cat peer_laptop1.conf
```
This showed the following contents:

```
[Interface]
Address = 10.0.0.3
PrivateKey = KHtgPTTnoe4JWaPBEyn/ekXlhOjVhOkojRLc234nlGQ=
ListenPort = 51820
DNS = 10.0.0.1

[Peer]
PublicKey = 6bsvtnOhlxR0dY0q1sKQkqIZ6PSFsbNO47ue55IJdW0=
Endpoint = 159.65.41.24:51820
AllowedIPs = 0.0.0.0/0, ::/0
```

Create a `peer_laptop1.conf` file on the laptop being used, then open the Wireguard application and click `Import tunnel(s) from file`, then select the `.conf` file.

Open up two windows on IPLeak.net. Let them load. Then, activate Wireguard and refresh one of the windows.
