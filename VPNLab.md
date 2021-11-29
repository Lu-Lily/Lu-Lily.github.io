# Cloud Wireguard VPN using Docker

## Create DigitalOcean account

First, go to https://www.digitalocean.com/ to sign up for an account.

## Create Ubuntu 20.04 Droplet: 

Once the account has been created, click on  `Get started with a droplet`. Next, select the following options:\
Ubuntu 20.04\
Basic\
Regular Intel with SSD\
$5/mo\
New York datacenter 3 (or any other datacenter)\
VPC Network: default-nyc3\
no additional addons\
password: I used generated password\
hostname: I chose to use `ubuntu-wireguard-vpn-docker`

## Install Docker

Go to the droplet console on DigitalOcean's dashboard, then install docker via the following steps:

Install the necessary tools:

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
```

Add docker key:

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Add docker repo:

My droplet is running Ubuntu 20.04 (LTS) x64 so I use the 64bit OS repo:

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

### Begin by creating the wireguard directory and the `docker-compose.yml` file:

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

Modify the TZ, SERVERURL, and PEERS fields as needed.\
I used `TZ=America/Chicago`.
I found my SERVERURL (aka server IP address) on the DigitalOcean dashboard as `159.65.41.24` and thus used `SERVERURL=159.65.41.24`.\
I have a pc, a laptop, and a phone, so I changed the PEERS field to `PEERS=pc1, laptop1, phone1`.\

Now hit `CTRL` + `X`, `Y`, `ENTER` to save and exit the file.

### Start Wireguard:

```
cd ~/wireguard/
docker-compose up -d
```

#### Connect your phone to Wireguard

To begin, since I use iOS, I had to download the WireGuard app from the appstore. 

Use the following command to show the execution log and QR codes of the Wireguard VPN connection settings:

```
docker-compose logs -f wireguard
```

Now, open the Wireguard VPN connection application and click on `Add a tunnel`, then `Create from QR code`. I scanned the phone1 QR code. Next, I checked the IP address using IPLeak.net. Then I activated the VPN on the Wireguard app, and refreshed the page:

<p align="middle">
  <img src="https://user-images.githubusercontent.com/56270862/143801491-5a0b647d-baa8-450d-888c-1b94f4b54827.png" width="450" />
  <img src="https://user-images.githubusercontent.com/56270862/143801563-c571b166-16cd-4289-8f21-1cc6895b1519.png" width="450" /> 
</p>

#### Connect your laptop to Wireguard

Since I use a Windows laptop, I navigated to the Wireguard website and downloaded Wireguard from the windows installer.

Next, on the DigitalOcean droplet console, cd into `~/wireguard/configs/{username}`. For me, that was `~/wireguard/configs/peer_laptop1`.

Next read and copy the `.conf` file contents:

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

![Laptop VPN test](https://user-images.githubusercontent.com/56270862/143802476-ad137df0-57b1-4ee6-8ecd-bc4223cd7294.png)

