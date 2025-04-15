# Description

This role configures [HedgeDoc](https://github.com/hedgedoc/hedgedoc), an open-source document editing platform.

# Configuration

```yaml
hedgedoc_domain: 'notes.status.im'
# GitHub OAuth
hedgedoc_gh_oauth_id: 'super-secret-github-oauth-id'
hedgedoc_gh_oauth_secret: super-secret-github-oauth-key'
# Google OAuth
hedgedoc_gg_oauth_id: 'super-secret-google-oauth-id'
hedgedoc_gg_oauth_secret: super-secret-google-oauth-key'
```

# Management

You can manage the containers using `docker-compose` command:
```
admin@node-01.do-ams3.todo.misc:~ % cd /docker/hedgedoc
admin@node-01.do-ams3.todo.misc:/docker/hedgedoc % docker-compose --compatibility up --force-recreate -d
Recreating hedgedoc-db ... done
Recreating hedgedoc-app ... done
admin@node-01.do-ams3.todo.misc:/docker/hedgedoc % docker ps --filter=name=hedgedoc
CONTAINER ID        NAMES               IMAGE                       CREATED             STATUS
15ebf1522b78        hedgedoc-app          hackmdio/hedgedoc:1.10.3  3 seconds ago       Up 1 second
fd7bf9523578        hedgedoc-db           postgres:9.6-alpine       15 seconds ago      Up 13 seconds
```
For user management you can see the [`USERS.md`](./USERS.md) document.

# Backups

Backups are done via a [systemd timer](https://www.freedesktop.org/software/systemd/man/systemd.timer.html) and [`mongodump`](https://docs.mongodb.com/manual/reference/program/mongodump/):
```
 > sudo systemctl list-timers '*-hedgedoc-db.timer'
NEXT                         LEFT    LAST PASSED UNIT                   ACTIVATES
Sat 2020-01-25 00:00:00 UTC  7h left n/a  n/a    backup-hedgedoc-db.timer backup-hedgedoc-db.service
Sat 2020-01-25 00:00:00 UTC  7h left n/a  n/a    dump-hedgedoc-db.timer   dump-hedgedoc-db.service
```
You can create an SQL backup of the PostgreSQL database by running:
```
 > sudo systemctl start dump-hedgedoc-db.service
 > sudo systemctl status dump-hedgedoc-db.service
● dump-hedgedoc-db.service - Dump HedgeDoc PostgreSQL database.
     Loaded: loaded (/etc/systemd/system/dump-hedgedoc-db.service; static; vendor preset: enabled)
     Active: inactive (dead) since Thu 2021-03-04 19:39:15 UTC; 7s ago
TriggeredBy: ● dump-hedgedoc-db.timer
       Docs: https://github.com/status-im/infra-role-systemd-timer
    Process: 867920 ExecStart=/usr/local/bin/dump-hedgedoc-db (code=exited, status=0/SUCCESS)
   Main PID: 867920 (code=exited, status=0/SUCCESS)

systemd[1]: Starting Dump HedgeDoc PostgreSQL database....
systemd[1]: dump-hedgedoc-db.service: Succeeded.
systemd[1]: Finished Dump HedgeDoc PostgreSQL database..
```
