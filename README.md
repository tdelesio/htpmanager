Traefik setup:


1.
sudo touch /mnt/storage/appdata/docker/shared/.htpasswd
nano /mnt/storage/appdata/docker/shared/.htpasswd
go to http://www.htaccesstools.com/htpasswd-generator/ and generate the password
Add generated items to the file:



2.
Add the following to the /etc/environment file

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