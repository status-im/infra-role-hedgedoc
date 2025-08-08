# Backups

Backups are done via a [systemd timer](https://github.com/status-im/infra-role-systemd-timer) and [`pg_dump`](https://www.postgresql.org/docs/current/app-pgdump.html):
```
 > sudo systemctl list-timers '*-hedgedoc-db.timer'
NEXT                        LEFT     LAST                        PASSED UNIT                     ACTIVATES
Sat 2025-08-09 00:00:00 UTC 15h left Fri 2025-08-08 00:00:00 UTC 8h ago backup-hedgedoc-db.timer backup-hedgedoc-db.service
Sat 2025-08-09 00:00:00 UTC 15h left Fri 2025-08-08 00:00:00 UTC 8h ago dump-hedgedoc-db.timer   dump-hedgedoc-db.service
```
You can create an SQL backup of the PostgreSQL database by running:
```
 > sudo systemctl start dump-hedgedoc-db.service
 > sudo systemctl status dump-hedgedoc-db.service
○ dump-hedgedoc-db.service - Dump HedgeDoc PostgreSQL database.
     Loaded: loaded (/etc/systemd/system/dump-hedgedoc-db.service; static)
     Active: inactive (dead) since Fri 2025-08-08 00:00:25 UTC; 8h ago
TriggeredBy: ● dump-hedgedoc-db.timer
       Docs: https://github.com/status-im/infra-role-systemd-timer
    Process: 2288994 ExecStart=/usr/local/bin/dump-hedgedoc-db (code=exited, status=0/SUCCESS)
   Main PID: 2288994 (code=exited, status=0/SUCCESS)
        CPU: 61ms

dump-hedgedoc-db[2288998]: removed '/docker/hedgedoc/db/backup/hackmd/2196.dat.gz'
dump-hedgedoc-db[2288998]: removed '/docker/hedgedoc/db/backup/hackmd/toc.dat'
dump-hedgedoc-db[2288998]: removed '/docker/hedgedoc/db/backup/hackmd/2194.dat.gz'
dump-hedgedoc-db[2288998]: removed directory '/docker/hedgedoc/db/backup/hackmd'
systemd[1]: dump-hedgedoc-db.service: Deactivated successfully.
systemd[1]: Finished Dump HedgeDoc PostgreSQL database..
```

# Restoring

## Backup Existing Data

Create a copy of HedgeDoc folder:
```bash
cd /docker/hedgedoc
docker compose down
cd ../
sudo cp -r hedgedoc hedgedoc_bkp
```

## Restore Backup

To restore backup to the same directory the destination permissions need to be changed:
```bash
sudo chmod g+w -R /docker/hedgedoc/db/backup/hedgedoc
sudo -i -u restic restic restore --target=/ 033ce34d
sudo chmod g-w -R /docker/hedgedoc/db/backup/hedgedoc
```
```
Summary: Restored 5 files/dirs (0 B) in 0:00, skipped 8 files/dirs 164.683 MiB
```
By using `--target=/` we restore the backup to the same location from which it was copied.

## Load Database Dump

:warning: This is a destructive operation, especially when done with `--clean`.

You can do it from the host:
```bash
source /docker/hedgedoc/.psql.env
sudo -E \
    pg_restore --clean -d hedgedoc -f /docker/hedgedoc/db/backup/hedgedoc
```
Of from within the container:
```bash
docker exec -it nextcloud-db 
    pg_restore --clean -d hedgedoc -f /docker/hedgedoc/db/backup/hedgedoc
```

If you only want to restore deleted records you can try `--data-only` flag.

The `hedgedoc-app` container needs to be restarted afterewards.
