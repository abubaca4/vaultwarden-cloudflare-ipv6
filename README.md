Work with readme in progress.

This image for situation when you has only ipv6 address on your vps and your want to has access from ipv4. In cloudflare reverse proxy you receive ipv4&ipv6 address even you have only one address.

# vaultwarden-cloudflare-ipv6

This is docker-compose that run [vaultwarden](https://github.com/dani-garcia/vaultwarden) with [nginx](https://github.com/nginxinc/docker-nginx) reverse proxy and backups by [vaultwarden-backup](https://github.com/ttionya/vaultwarden-backup/).

Steps of runing:

### 0 step system prepare

You should set docker mirror with ipv6 support. [Issue about ipv6 support in docker](https://github.com/docker/roadmap/issues/89#issuecomment-817263625). You can use google mirror https://mirror.gcr.io or docker ipv6-only mirror registry.ipv6.docker.com

Мery desirable to set [nat64](https://nat64.xyz/) o somefing can be broken. You shoud set selected dns as single proxy in the system(look [arch wiki](https://wiki.archlinux.org/title/systemd-resolved) for systemd.)

### 1 step

You need to run [ipv6nat](https://github.com/robbertkl/docker-ipv6nat) (it is simplest way to give network access for docker containers.

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
