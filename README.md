#Linux File Mounts
Edit the fstab file by issuing
sudo /etc/fstab

At a line per share type.  So for instance:
/mnt/storage/media/<share_name>

If you want to use a different path for your content, then you need to edit
the docker-compose file and modify the volumes at the end of the file.

#Docker Compose

My default path:
/mnt/storage/appdata/docker/

clone repo at that location


#Traefik setup:
1.
sudo touch /mnt/storage/appdata/docker/shared/.htpasswd
nano /mnt/storage/appdata/docker/shared/.htpasswd
go to http://www.htaccesstools.com/htpasswd-generator/ and generate the password
Add generated items to the file:



2.
Add the following to the /etc/environment file
EDIT:  I couldn;'t get the environment file to work.  As an alt, I created an .env in root
of this repo.  This seamed to work great.


TZ="America/New_York"
HTTP_USERNAME=username
HTTP_PASSWORD=mystrongpassword
DOMAINNAME=example.com
CLOUDFLARE_EMAIL=email@example.com
CLOUDFLARE_API_KEY=XXXXXXXXXXXX
PLEX_CLAIM=

Get Plex claim from here: https://www.plex.tv/claim/

3.  
chmod 600 /mnt/storage/appdata/docker/htpmanager/traefik/acme/acme.json

4. 
nano /mnt/storage/appdata/docker/htpmanager/traefik/traefik.toml

EDIT the following in that file:
a. email@domain.com: with your email.
b. EXAMPLE.COM: with your private domain name.
c. InsecureSkipVerify = true: I had to add at the beginning to allow some apps (eg. UniFi controller) be accessible through Traefik.
d. provider = "cloudflare": Change to your DNS provider for DNS challenge.
e. exposedbydefault = false: This will force you to use traefik.enable=true label in docker compose to put apps behind traefik.

5. 
mnt/storage/appdata/sabnzbd/config/sabnzbd.ini
update sabnzbd.ini and add host to the white list

6. 
create docker network
docker network create traefik_proxy

7.
create a mount for traefik and copy the /htpmanager/traefik folder to the mount.

8.
generate a acme.json file in the /traefik/acme/acme.json folder 

If you get stuck, check out:
https://www.smarthomebeginner.com/traefik-reverse-proxy-tutorial-for-docker/

I used this site heavily to get it running.  