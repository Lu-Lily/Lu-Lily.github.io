# Option C: PiHole

## Introduction

I chose to install PiHole on Docker. I followed the powerpoint slides in order to install Docker on my Ubuntu VM. Then to begin, I went to https://pi-hole.net/. After that, I clicked Docker Install: https://github.com/pi-hole/docker-pi-hole/#running-pi-hole-docker. At this point, I wasn't completely sure how to go through with the full installation, so I searched up a guide for installing PiHole on Docker and came across the following guide: https://www.geeksforgeeks.org/create-your-own-secure-home-network-using-pi-hole-and-docker/

## Starting Docker and downloading PiHole

Following the guide, I executed these commands in the terminal on my Ubuntu VM in order to start docker and download PiHole:

```
sudo systemctl start docker
sudo docker pull pihole/pihole
```

## Resolving port 53 conflicts on Ubuntu

In order to resolve conflicts with port 53 on Ubuntu, I had to stop `systemd-resolved` to free up the port.

```
sudo systemctl stop systemd-resolved.service
sudo systemctl disable systemd-resolved.service 
```

## Changing the DNS

Further following the guide, I executed

```
sudo nano /etc/resolv.conf
```

And then I changed the `nameserver` to `8.8.8.8`

## Building and starting PiHole

Next, I created a docker-compose.yml file:

```
sudo nano docker-compose.yml
```

I copied the following docker-compose.yml.example from github while uncommenting `WEBPASSWORD` and entering my own password:

```
version: "3"

# More info at https://github.com/pi-hole/docker-pi-hole/ and https://docs.pi-hole.net/
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "67:67/udp"
      - "80:80/tcp"
    environment:
      TZ: 'America/Chicago'
      # WEBPASSWORD: 'set a secure password here or it will be random'
    # Volumes store your data between container upgrades
    volumes:
      - './etc-pihole/:/etc/pihole/'
      - './etc-dnsmasq.d/:/etc/dnsmasq.d/'
    # Recommended but not required (DHCP needs NET_ADMIN)
    #   https://github.com/pi-hole/docker-pi-hole#note-on-capabilities
    cap_add:
      - NET_ADMIN
    restart: unless-stopped
```

I saved the file. Next, I ran the compose file:

```
sudo docker-compose up -d
```

Here, I had an issue because I had not actually installed docker-compose, so I had to get that installed before attempting to run the file again.

## Changing the PiHole password

Next, I entered PiHole with

```
sudo docker exec -it pihole bash
```

To change the password, I executed

```
pihole -a -p
```

At this point, I simply deleted the password rather than changing it to a new password. I then exited PiHole from the terminal.

## PiHole Web UI and setting up DNS

I opened Firefox and went to http://localhost/admin/ to open up the PiHole Web UI. Confirming that it was working, I needed to set my device to use PiHole as my DNS server. To do this, I entered the Ubuntu `Settings` which opened up on `Network` settings. I then navigated to `IPv4` where I set the DNS to `Manual` and entered the PiHole (device) IP for the DNS server. I then saved the settings.

I browsed around on a few sites to test and then came back to check my PiHole dashboard:

![Screenshot](https://github.com/Lu-Lily/Lu-Lily.github.io/blob/main/2021-11-14%2022_46_32.png?raw=true)

## Resource List

Here is a congregated list of all the resources used within this document for convenience:

[PiHole Website](https://pi-hole.net/)\
[PiHole Docker GitHub Guide](https://github.com/pi-hole/docker-pi-hole/#running-pi-hole-docker)\
[Configure devices to use PiHole as their DNS server](https://discourse.pi-hole.net/t/how-do-i-configure-my-devices-to-use-pi-hole-as-their-dns-server/245)\
[docker_compose.yml.example](https://github.com/pi-hole/docker-pi-hole/blob/master/docker-compose.yml.example)\
[GeeksforGeeks PiHole Docker Guide](https://www.geeksforgeeks.org/create-your-own-secure-home-network-using-pi-hole-and-docker/)
