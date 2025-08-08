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

# Backup & Restore

See [BACKUP.md](./BACKUP.md) doc.
