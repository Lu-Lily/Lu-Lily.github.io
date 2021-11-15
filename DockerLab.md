# Option C: PiHole

Start with https://pi-hole.net/

Click Docker Install: https://github.com/pi-hole/docker-pi-hole/#running-pi-hole-docker

Followed this guide: https://www.geeksforgeeks.org/create-your-own-secure-home-network-using-pi-hole-and-docker/

sudo systemctl start docker

sudo docker pull pihole/pihole

sudo systemctl stop systemd-resolved.service

sudo systemctl disable systemd-resolved.service 

sudo nano /etc/resolv.conf -? change to 8.8.8.8

sudo nano docker-compose.yml

copy docker-compose.yml.example from github

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

sudo docker-compose up -d

sudo docker exec -it pihole bash

Change password

pihole -a -p

exit

http://localhost/admin/

Settings -> Network -> IPv4 -> manual DNS -> pihole IP

![Screenshot](https://github.com/Lu-Lily/Lu-Lily.github.io/blob/main/2021-11-14%2022_46_32.png?raw=true)
