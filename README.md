# Overview

This is my setup for my home automation entertainment center.  I have docker running on a headless [micropc](https://www.amazon.com/dp/B00JQYJ8B8/?coliid=I13SMIGSEFBZRV&colid=A0LAA40HO9AX&psc=0&ref_=lv_ov_lig_dp_it) running Ubuntux linux and all my data on a [Synology DS 1515+](https://www.amazon.com/dp/B00PTGQJL4/?coliid=I2IHNIZG3QSPGN&colid=A0LAA40HO9AX&psc=0&ref_=lv_ov_lig_dp_it) NAS running RAID 5.  I use the unifi suit of networking gear and have a bunch of WAPs around the house.  You can see my setup for that in my other repo.  I use docker-compose to run the containers and they are broken up into two separate files.  They are split into two separate files based on the network requirements.  Running Plex and PiHole I want them to have dedicated IP addresses on the network (vs having a port on the host) so these are broken out into a separate file.  The containres that are running are:


## Main File (docker-compose.yml):
* Traefik: Reverse proxy that sits in front of all the containers and allows me to map subdomains to ports.
* portainer:  Container admin software to simplify the managemetn and problem solving of the containres
* Organizr: Landing page/portal for all the subapps.
* sabnzbd: Usenet downloader
* nzbhydra: Usenet search aggregator.  Allows you to search one across all the indexes
* Sonarr - TV management and downloader
* lazylibarian - Book management and downloader
* Watchtower - Container updater.  When new versions come out, this will grab them
* Radarr - Movie management and downloader


## Separate Network File (called pihole.yml right now):
* Plex - Media Management and player
* Cloudflared: Proxy DNS server
* Pihole: Network wide Ad killer.  


# Setup

Disclaimer: It's been a while since I have set this up from scratch.  Hopefully I have captured all the steps but they may be incomplete.  Obviously my exact set up isn't going to be 1-1 for you.  


## Linux File Mounts

The linux box needs to mount to the NAS.  This allows you to treat the NAS as if it was a local file system.  The root of all my files config and docker files are in the base of

```
/mnt/storage/appdata
```

NAS is mapped to
```
/mnt/storage/media
```

In order to get that mapped, we need to edit the fstab file.
```
sudo /etc/fstab
```

Add one line per share type.  Depending on how you have your NAS organized, you probably will have one share type for each type of media (movies, tv, music, pics, etc)  So for instance:

Mine looks like
```
10.0.0.20:/volume1/movies /mnt/storage/media/movies nfs auto 0 0
10.0.0.20:/volume1/4kmovies /mnt/storage/media/4kmovies nfs auto 0 0
10.0.0.20:/volume1/books /mnt/storage/media/books nfs auto 0 0
10.0.0.20:/volume1/music /mnt/storage/media/music nfs auto 0 0
10.0.0.20:/volume1/christmas /mnt/storage/media/christmas nfs auto 0 0
10.0.0.20:/volume1/home_movies /mnt/storage/media/home_movies nfs auto 0 0
10.0.0.20:/volume1/kids /mnt/storage/media/kids nfs auto 0 0
10.0.0.20:/volume1/photos /mnt/storage/media/photos nfs auto 0 0
10.0.0.20:/volume1/tv /mnt/storage/media/tv nfs auto 0 0
10.0.0.20:/volume1/plex_server /mnt/storage/appdata nfs auto 0 0
#10.0.0.20:/volume1/comics /mnt/storage/media/comics nfs auto 0 0
#10.0.0.20:/volume1/other /mnt/storage/media/other nfs auto 0 0
```

This will eventually need to map back to your docker-compose file so that the containres can find this media.  


If you want to use a different path for your content, then you need to edit
the docker-compose file and modify the volumes at the end of the file.  For NFS, you also have to set the permissions correct on the NAS.  

![NAS NFS](/nas_nfs.png)

##Traefik Setup

Traefik is a reverse proxy that sits in front of all the containres.  All the data routes through it to get to the containers.  With that in mind, we need to configure traefik to map subdomains to ports/containers.  There is some set up with that as well.

1. Create a .htpasswd file.
```
sudo touch /mnt/storage/appdata/docker/shared/.htpasswd
nano /mnt/storage/appdata/docker/shared/.htpasswd
```

##Docker Compose

You will need to install docker and docker-compose on the linux box.  You can Google that for yourself and is outside of the scope of this.  I put my docker-compose files in the path:

```
/mnt/storage/appdata/docker/
```

###Clone Repo and Edit docker-compose file

I did a git clone at that root directory and it creates a htpmanager folder.  You can cd into that directory.  Edit the docker-compose file to map it correctly to your media mounts.  

###Environment
Add the following to the /etc/environment file
EDIT:  I couldn't get the environment file to work.  As an alt, I created an .env in root of this repo.  This seamed to work great.  Get Plex claim from here: https://www.plex.tv/claim/

```
TZ="America/New_York"
HTTP_USERNAME=username
HTTP_PASSWORD=mystrongpassword
DOMAINNAME=example.com
CLOUDFLARE_EMAIL=email@example.com
CLOUDFLARE_API_KEY=XXXXXXXXXXXX
PLEX_CLAIM=
```

###Traefik
You need to create a acme.json file in the /traefik/acme/acme.json folder

If you get stuck, check out:
https://www.smarthomebeginner.com/traefik-reverse-proxy-tutorial-for-docker/

I used this site heavily to get it running.  

You need to set the permissions on the acme.json file.
```
chmod 600 /mnt/storage/appdata/docker/htpmanager/traefik/acme/acme.json
```

You need to also edit the traefik.toml and edi the following:
```
nano /mnt/storage/appdata/docker/htpmanager/traefik/traefik.toml
```
1. email@domain.com: with your email.
1. EXAMPLE.COM: with your private domain name.
1. InsecureSkipVerify = true: I had to add at the beginning to allow some apps (eg. UniFi controller) be accessible through Traefik.
1. provider = "cloudflare": Change to your DNS provider for DNS challenge.
1. exposedbydefault = false: This will force you to use traefik.enable=true label in docker compose to put apps behind traefik.

For security reasons, you don't wants the traefik under the git repo.  The one I have is the base default.  You need to copy that folder to another directory.

```
cp -r /mnt/storage/appdata/docker/htpmanager/traefik /mnt/storage/appdata/traefik
```

###sabnzbd
You need to add the traefik domain to the whitelist of sabnzbd.  Otherwise, you won't be able to access it.
```
nano mnt/storage/appdata/sabnzbd/config/sabnzbd.ini
```

###Traefik network
You need to manually create a network.

```
docker network create traefik_proxy
```

###PiHole
You also need to pull up the pihole admin console and change the DNS to listen to all interfaces.  Then you need to go into the router and point DNS to the pihole server ip.  You probably only need to do this if your linux box has multiple NICs.

###Run and Test
Once you are happy with your docker-compose files, you can start the containers.  You can do this by doing:

```
docker-compose up -d
docker-compose up -f pihole.yml -p pihole up -d
```
