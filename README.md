# README

Devteds [Episode #2](https://devteds.com/episodes/2-setup-private-docker-registry-secure-with-ssl-password)

Learn how to setup a private secure docker registry in the cloud.

[Episode video link](https://youtu.be/KMldBtbJ4qI)

[![Episode Video Link](https://i.ytimg.com/vi/KMldBtbJ4qI/hqdefault.jpg)](https://youtu.be/KMldBtbJ4qI)

Visit https://devteds.com to watch all the episodes

## Tested on

* Mac OSX - 10.10.5
* Docker - 1.12.1
* Docker compose - 1.8.0
* Docker Machine - 0.8.1
* Ubuntu 16.x (Droplet on Digitalocean)

## Instructions / commands

Login to digitalocean.com, sign up for an account if you don't have one already, generate ACCESS_TOKEN and save

### Create VM / Droplet on DigitalOcean
```
mkdir ~/projects/private-registry
cd ~/projects/private-registry

docker-machine create -d digitalocean --digitalocean-access-token=<ACCESS_TOKEN> my-private-registry

# Get the SERVER IP ADDRESS using,
docker-machine ip my-private-registry
```

If you don’t have a DigitalOcean account, [Register now](https://m.do.co/c/a9b9aef156d6) and get some credit and that should get you running a VM of about 2 months (promo as of 10/30/16) - https://m.do.co/c/a9b9aef156d6

### Configure & Run Services

```
# create nginx root 
docker-machine ssh my-private-registry mkdir /root/nginx-root

# create/copy basic nginx.conf,
docker-machine scp nginx.conf my-private-registry:/root/nginx-root/

# create/copy an index.html file,
docker-machine scp index.html my-private-registry:/root/nginx-root/

# create docker-compose.yml for nginx service. and,
eval $(docker-machine env my-private-registry)
env | grep DOCKER
# verify the docker host which should be pointing to the public IP Address of the my-private-registry
docker-compose start

# Verify nginx on http://<SERVER IP ADDRESS>/ and that should work

# Pick a domain name - free ones, buy one, sub domain off of an existing one or if you have a spare
# Set the A record pointing to the SERVER IP ADDRESS

# Verify nginx using http://<DOMAIN NAME>/ and that should work

# Add registry service to docker-compose.yml
# Update nginx to define upstream for registry service
docker-compose stop
docker-machine scp nginx.conf my-private-registry:/root/
docker-compose start

# Verify registry http://<DOMAIN NAME>/v2/_catalog and that should work

docker-compose stop

mkdir certs
# Get SSL certificate from sslforfree.com (certificate.crt, ca_bundle.crt & private.key)
# Unzip the files into certs folder create server.crt using,
cat certs/certificate.crt certs/ca_bundle.crt > certs/server.crt

docker-machine ssh my-private-registry mkdir /root/certs
docker-machine scp certs/private.key my-private-registry:/root/certs/
docker-machine scp certs/server.crt my-private-registry:/root/certs/

# Update nginx to add virtual server for 443 with SSL ON
docker-machine scp nginx.conf my-private-registry:/root/
docker-compose start
# or docker-compose up -d

# Verify SSL https://<DOMAIN NAME>/

docker-compose stop
# Update nginx to redirect all HTTP to HTTPS
docker-machine scp nginx.conf my-private-registry:/root/
docker-compose start
# Verify the redirects

docker-compose stop
# Generate htpasswd on the server
# Update nginx for basic_auth
docker-machine scp nginx.conf my-private-registry:/root/
docker-compose start
# Verify basic auth is working

```


## Create a dev machine

Switch to a separate terminal window to create a separate docker machine to test the registry

```
docker-machine create -d virtualbox dev1
docker-machine ssh dev1
docker pull busybox
docker login <DOMAIN-NAME>
# Provide login details
docker tag busybox <DOMAIN-NAME>/busybox
docker push <DOMAIN-NAME>/busybox

# Verify on http://<DOMAIN NAME>/v2/_catalog
```
