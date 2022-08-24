Most part of this is old and will be changed in future

# vaultwarden-backup-nginx-le

This is docker-compose that run [vaultwarden](https://github.com/dani-garcia/vaultwarden) with [nginx-le](https://github.com/nginx-le/nginx-le) reverse proxy and backups by [vaultwarden-backup](https://github.com/ttionya/vaultwarden-backup/).

Also [watchtower](https://github.com/containrrr/watchtower) used to auto upgrade images.

It has two version: with ipv6 support and without it. The main difference is ipv6 version can log client ipv6 addresses but requaired some additional actions to setup.

Steps of runing:

### 1 step

Only when used ipv6 version.

You need to run [ipv6nat](https://github.com/robbertkl/docker-ipv6nat)

```
docker run -d --name ipv6nat --privileged --network host --restart unless-stopped -v /var/run/docker.sock:/var/run/docker.sock:ro -v /lib/modules:/lib/modules:ro robbertkl/ipv6nat
```

### 2 step

You need to [configure](https://github.com/ttionya/vaultwarden-backup/#backup) rclone. RemoteName will be used later.

I recomended to use [clouds](https://rclone.org/overview/) that save token or api key like OneDrive and Dropbox and not like mega where used your username and password.

### 3 step

Change docker-compose.yml with your own settings.

You remote name(you can also change other env settings of vaultwarden-backup as you like):
```
RCLONE_REMOTE_NAME: 'BitwardenBackup'
```
You email and domain(recomended to use real email, Let’s Encrypt will send you warning if something will goes wrong and Certificate not updated automatic.)
```
LE_EMAIL: 'name@example.com'
LE_FQDN: 'www.example.com'
```

## Restore backup in new system

### 1 step

You need to run ```docker-compose up``` first time to create vaultwarden-data volume correctly. Please wait until DH will be generated(or nginx will not start correctly without deling  ssl volume)

### 2 step

You need to put backup archive to local path(I created clear dir only for backup archive) and use path in this command instead of /local/backup/path/ and your backup file archive name instead of backup.zip:

```
docker run --rm -it \
  --mount type=volume,source=vaultwarden-data,target=/bitwarden/data/ \
  --mount type=bind,source=/local/backup/path/,target=/bitwarden/restore/ \
  ttionya/vaultwarden-backup:latest restore --zip-file backup.zip
```
