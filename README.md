# Description

This role configures [CodiMD](https://github.com/hackmdio/codimd), an open-source document editing platform.

# Configuration

```yaml
codimd_domain: 'notes.status.im'
# GitHub OAuth
codimd_gh_oauth_id: 'super-secret-github-oauth-id'
codimd_gh_oauth_secret: super-secret-github-oauth-key'
# Google OAuth
codimd_gg_oauth_id: 'super-secret-google-oauth-id'
codimd_gg_oauth_secret: super-secret-google-oauth-key'
```

# Management

You can manage the containers using `docker-compose` command:
```
admin@node-01.do-ams3.todo.misc:~ % cd /docker/codimd
admin@node-01.do-ams3.todo.misc:/docker/codimd % docker-compose --compatibility up --force-recreate -d
Recreating codimd-db ... done
Recreating codimd-app ... done
admin@node-01.do-ams3.todo.misc:/docker/codimd % docker ps --filter=name=codimd
CONTAINER ID        NAMES               IMAGE                       CREATED             STATUS
15ebf1522b78        codimd-app          hackmdio/codimd:2.3.2       3 seconds ago       Up 1 second
fd7bf9523578        codimd-db           postgres:9.6-alpine         15 seconds ago      Up 13 seconds
```
For user management you can see the [`USERS.md`](./USERS.md) document.

# Backups

Backups are done via a [systemd timer](https://www.freedesktop.org/software/systemd/man/systemd.timer.html) and [`mongodump`](https://docs.mongodb.com/manual/reference/program/mongodump/):
```
 > sudo systemctl list-timers '*-codimd-db.timer'
NEXT                         LEFT    LAST PASSED UNIT                   ACTIVATES
Sat 2020-01-25 00:00:00 UTC  7h left n/a  n/a    backup-codimd-db.timer backup-codimd-db.service
Sat 2020-01-25 00:00:00 UTC  7h left n/a  n/a    dump-codimd-db.timer   dump-codimd-db.service
```
You can create an SQL backup of the PostgreSQL database by running:
```
 > sudo systemctl start dump-codimd-db.service
 > sudo systemctl status dump-codimd-db.service
● dump-codimd-db.service - Dump CodiMD PostgreSQL database.
     Loaded: loaded (/etc/systemd/system/dump-codimd-db.service; static; vendor preset: enabled)
     Active: inactive (dead) since Thu 2021-03-04 19:39:15 UTC; 7s ago
TriggeredBy: ● dump-codimd-db.timer
       Docs: https://github.com/status-im/infra-role-systemd-timer
    Process: 867920 ExecStart=/usr/local/bin/dump-codimd-db (code=exited, status=0/SUCCESS)
   Main PID: 867920 (code=exited, status=0/SUCCESS)

systemd[1]: Starting Dump CodiMD PostgreSQL database....
systemd[1]: dump-codimd-db.service: Succeeded.
systemd[1]: Finished Dump CodiMD PostgreSQL database..
```

# Known Issues

A bug in S3 library configuration makes S3 uploads unusable.
For more details see: https://github.com/hackmdio/codimd/issues/1572
