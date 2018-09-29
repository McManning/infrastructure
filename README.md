
# Infrastructure

Collection of services running on my host(s).

Includes configurations for both Kubernetes clustering and Docker Compose for single hosts.

## Services Directory

### General

|Service                                        |Description
|---                                            |---
|[docker-nginx](https://github.com/McManning/docker-nginx)|Ingress to web services and RTMP
|watchtower                                     |Auto update for Docker containers
|sshd                                           |Endpoint for Travis CI deployments and media rsync
|certbot                                        |Auto update for Let's Encrypt certificates

### Social

|Service                                        |Description
|---                                            |---
|murmur                                         |Voice server
|[sybot](https://github.com/McManning/sybot)    |Bot for the murmur voice server

### Media

|Service                                        |Description
|---                                            |---
|transmission-openvpn                           |Download client
|sonarr                                         |Media management
|radarr                                         |Media management

## Volume Mounts

Mount points expected on the host - and services that share each volume

|Path                       |k8s Type           |Services       |Description
|---                        |---                |---            |---
|${LETS_ENCRYPT}            |Secret             |nginx, murmur  |Cert storage
|${MOUNT_POINT}/sshd        |ConfigMap          |sshd           |deployer public keys
|${MOUNT_POINT}/nginx/conf  |ConfigMap          |nginx          |proxy configs
|${MOUNT_POINT}/murmur/conf |ConfigMap          |murmur         |server configs
|${MOUNT_POINT}/murmur/data |PersistentVolume   |murmur         |SQLite storage
|${MOUNT_POINT}/transmission|PersistentVolume   |transmission, sonarr, radarr   |config and download staging
|${MOUNT_POINT}/sonarr/config|PersistentVolume  |sonarr         |SQLite storage
|${MOUNT_POINT}/radarr/config|PersistentVolume  |radarr         |SQLite storage
|${MOUNT_POINT}/videos      |PersistentVolume   |sonarr, radarr, sshd |staging for rsync
|${MOUNT_POINT}/public      |PersistentVolume   |nginx, sshd    |Static site deployment

## Configuration

When using the Docker Compose build, the following values need to be set in `.env`:

```sh
# Host subdirectory to mount persistent volumes (e.g. /srv)
MOUNT_POINT=<string>

# Certificate storage/renewal mount
LETS_ENCRYPT=/etc/letsencrypt

# User/group to run sonarr/radarr/transmission services under
# This resolves potential access issues with the shared mount
PGID=$(cut -d: -f3 < <(getent group docker))
PUID=$(id -u docker)

# VPN settings for transmission-ovpn
# For available configurations, see:
# https://github.com/haugene/docker-transmission-openvpn/tree/master/openvpn
VPN_PROVIDER=<string>
VPN_USERNAME=<string>
VPN_PASSWORD=<string>
VPN_CONFIG=<string>

# Murmur ICE password (rw) for sybot service
ICE_SECRET=<string>
```
