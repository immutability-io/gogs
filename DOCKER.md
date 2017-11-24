# Dockerized Gogs git server and alpine postgres in 20 minutes or less

## Grab your containers
```
$ docker pull immutability/gogs
$ docker pull immutability/docker-alpine-postgres
```
# Start your containers
```
$ docker run --rm --name gogs immutability/gogs
Apr  8 00:03:03 syslogd started: BusyBox v1.24.1
2016/04/08 00:03:03 [W] Custom config '/data/gogs/conf/app.ini' not found, ignore this if you're running first time
2016/04/08 00:03:03 [T] Custom path: /data/gogs
2016/04/08 00:03:03 [T] Log path: /app/gogs/log
2016/04/08 00:03:03 [I] Gogs: Go Git Service 0.9.15.0323
2016/04/08 00:03:03 [I] Log Mode: Console(Trace)
2016/04/08 00:03:03 [I] Cache Service Enabled
2016/04/08 00:03:03 [I] Session Service Enabled
2016/04/08 00:03:03 [I] SQLite3 Supported
2016/04/08 00:03:03 [I] Run Mode: Development
2016/04/08 00:03:03 [I] Listen: http://0.0.0.0:3000
Apr  8 00:03:03 sshd[31]: Server listening on 0.0.0.0 port 22.
Note that this docker image listens on two ports: gogs on 3000 and ssh on 22. Usually running ssh in a container is considered bad form but in this case we are supporting git-over-ssh. Let's kill this container and restart it properly.
```
## Bind mount
```
$ docker stop gogs; docker rm gogs
$ mkdir -p /srv/lxc/gogs/data
$ mkdir -p /srv/lxc/postgres/data
$ docker run -d --name gogs \
  -p 3000:3000 -p 3022:22 -v /srv/lxc/gogs/data:/data immutability/gogs

```

## Inpsect

Let's take a look at that web service: open http://localhost:3000 gogs start page

Things are going so well at this point that I feel emboldened - what is it like running postgres in docker nowadays?

## Postgres
```
$ docker run -d --name postgres \
    -e POSTGRES_PASSWORD=supersecret \
    -v /srv/lxc/postgres/data:/var/lib/postgresql/data \
    -p 5432:5432 immutability/docker-alpine-postgres
```    

At this point we could spin up the gogs container and use the postgres superuser but lets see how we go about administrating postgres without installing psql.

```
$ docker exec -it postgres psql -U postgres
psql (9.4.6, server 9.5.1)
WARNING: psql major version 9.4, server major version 9.5.
         Some psql features might not work.
Type "help" for help.

postgres=# CREATE USER gogs WITH PASSWORD '^go4git&docker';
CREATE ROLE
postgres=# CREATE DATABASE gogs OWNER gogs;
CREATE DATABASE
postgres=# \q
Great now we spin up gogs this time linking it to postgres and data volumes:
```

## Link gogs to postgres
```
$ docker stop gogs; docker rm gogs
$ docker run --link postgres -d --name gogs \
  -p 3000:3000 -p 3022:22 -v /srv/lxc/gogs/data:/data immutability/gogs
$ open https://localhost:3000
```
Back at the start page select [[postgres]] with user:gogs pass:^go4git&docker
