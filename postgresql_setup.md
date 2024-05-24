PostgreSQL-15 Setup
===================
Table of Contents
------------------
   - [Repository Keyring](#repository-keyring)
   - [Install PostgreSQL](#install-postgresql)
   - [User Database](#user-database)
   - [Socket Connection](#socket-connection)
   - [Migrating Backend](#migrating-backend)
   - [Backup and Restore](#backup-and-restore)
##

1. ### Repository Keyring

- Install GnuPG from apt, and get postgres-common package which contains a shell script for importing the postgresql repository keyring:
```sh
root@host:~# apt-get install gnupg2 && \
wget http://deb.debian.org/debian/pool/main/p/postgresql-common/postgresql-common_225+deb11u1.tar.xz && \
tar -xf postgresql-common_225\+deb11u1.tar.xz && rm postgresql-common_225\+deb11u1.tar.xz && \
mkdir -p /usr/share/postgresql-common/pgdg && \
mv ./postgresql-common/pgdg/apt.postgresql.org.sh /usr/share/postgresql-common/pgdg/ && \
rm -r ./postgresql-common && \
/usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
```

2. ### Install PostgreSQL
- Add Packages
```sh
root@host:~# apt-get install postgresql-15 postgresql-server-dev-15
```

- Change the postgres user password
```sh
root@host:~# passwd postgres
```

- Increase shared memory buffer

`nano /etc/postgresql/15/main/postgresql.conf`
```
shared_buffers = 8192MB                  # min 128kB

```


- Verify the service is running, and start it if not
```sh
user@host:~$ /etc/init.d/postgresql status
user@host:~$ /etc/init.d/postgresql start
```

3. ### User Database
- Log in as postgres user
```sh
user@host:~$ su - postgres
```

- Create database user
```sh
postgres@host:~$ createuser --pwprompt psqluser
```

- Create one or more databases
```sh
postgres@host:~$ createdb -O psqluser mapdb && \
createdb -O psqluser playerdb && \
createdb -O psqluser authdb && \
createdb -O psqluser mapserverdb
```

- Verify you can connect to a database with the user
```sh
postgres@host:~$ psql -d mapdb -h localhost -U psqluser
```

4. ### Socket Connection
> Faster than IP connection for connecting services within localhost
   - Edit `/etc/postgresql/15/main/pg_hba.conf` and change `peer` to `scram-sha-256`:
```conf
# "local" is for Unix domain socket connections only
local   all             all                                     scram-sha-256
```
   - Edit `/etc/postgresql/15/main/pg_ident.conf` and add:
```conf
# MAPNAME       SYSTEM-USERNAME         PG-USERNAME
local           user                    psqluser
```
 - `SYSTEM-USERNAME` is the username which starts minetest service, ie; `user`
 - `PG-USERNAME` is the database owner name to which `user` will connect; `psqluser`
 - `MAPNAME` is the map name that was used in pg_hba.conf; set it to `local`

- Restart the PostgreSQL service
```sh
root@host:~# /etc/init.d/postgresql restart
```

- Create a .pgpass file with the correct permissions
```sh
user@host:~$ touch .pgpass && chmod 600 .pgpass && \
echo "localhost:5432:mapdb:psqluser:securepassword
localhost:5432:authdb:psqluser:securepassword
localhost:5432:playerdb:psqluser:securepassword"
```

- It is now possible to connect to the database and the `postgres` user with your `user` account
```sh
# Connect to database
user@host:~$ psql -d mapdb -U psqluser
# Change to postgres user
user@host:~$ su - postgres
```

You now have a running PostgreSQL service.

Next step is [Compile Minetest](/compile_minetestserver.md) or [MultiCraft](/compile_multicraftserver.md)


5. ### Migrating Backend
> Make a backup copy of your files before migrating!
   - Edit `world.mt` add the pgslq_connection of the backend to be migrated.
   - Do not change the backend type, it will be updated by the migration automation.
```conf
# Socket connections:
backend = sqlite3
pgsql_connection = postgresql:///mapdb?host=/var/run/postgresql&user=psqluser&password=securepassword&dbname=mapdb
auth_backend = sqlite3
pgsql_auth_connection = postgresql:///mapdb?host=/var/run/postgresql&user=psqluser&password=securepassword&dbname=mapdb
player_backend = sqlite3
pgsql_player_connection = postgresql:///mapdb?host=/var/run/postgresql&user=psqluser&password=securepassword&dbname=mapdb
# IP Connections
# pgsql_connection = host=127.0.0.1 port=5432 user=psqluser password=securepassword dbname=mapdb
# pgsql_auth_connection = host=127.0.0.1 port=5432 user=psqluser password=securepassword dbname=mapdb
# pgsql_player_connection = host=127.0.0.1 port=5432 user=psqluser password=securepassword dbname=mapdb
```

- Run the minetestserver binary with migration flags
```sh
user@host:~$ ./minetest-master/bin/minetestserver --migrate postgresql --world /home/user/minetest-master/worlds/world && \
./minetest-master/bin/minetestserver --migrate-auth postgresql --world /home/user/minetest-master/worlds/world && \
./minetest-master/bin/minetestserver --migrate-players postgresql --world /home/user/minetest-master/worlds/world
```

> Depending on the size of your map file, migration may take several hours.


5. ### Backup and Restore
   - Dump databases
```sh
user@host:~$ pg_dump -U psqluser -h localhost -d mapdb --clean -O | gzip -c > dump_mapdb.gz && \
pg_dump -U psqluser -h localhost -d authdb --clean -O | gzip -c > dump_mapdb.gz && \
pg_dump -U psqluser -h localhost -d playerdb --clean -O | gzip -c > dump_mapdb.gz

```

   - Dump and compress local database to remote server
```sh
user@host:~$ pg_dump -U psqluser -h localhost -d mapdb --clean -O | ssh -p 7743 user@remote -T "gzip -c > mapdb.gz"
```

   - Dump all databases into single file:
```sh
user@host:~$ su - postgres bash -c 'pg_dumpall --clean -O' | gzip -c > dumpall_db.gz
```

- Restore from dump file to existing database
```sh
user@host:~$ gunzip -k -c mapdb.gz | psql -U mapdb -h localhost -d mapdb
```
https://www.postgresql.org/docs/current/backup-dump.html



##
### mønκ
<img decoding="async" loading="lazy" src="https://cdn.discordapp.com/emojis/1194038093775376455.webp?size=64&quality=lossless">

##
License
-------
CC0 1.0 Universal

No Copyright

The person who associated a work with this deed has dedicated the work to the public domain by waiving all of his or her rights to the work worldwide under copyright law, including all related and neighboring rights, to the extent allowed by law.

You can copy, modify, distribute and perform the work, even for commercial purposes, all without asking permission.

In no way are the patent or trademark rights of any person affected by CC0, nor are the rights that other persons may have in the work or in how the work is used, such as publicity or privacy rights.

Unless expressly stated otherwise, the person who associated a work with this deed makes no warranties about the work, and disclaims liability for all uses of the work, to the fullest extent permitted by applicable law.
When using or citing the work, you should not imply endorsement by the author or the affirmer.
