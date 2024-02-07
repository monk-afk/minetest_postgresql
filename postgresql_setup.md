PostgreSQL-15 Setup
===================
Table of Contents
------------------
   - [Repository Keyring](#repository-keyring)
   - [Install PostgreSQL](#install-postgresql)
   - [User Database](#user-database)
   - [Socket Connection](#socket-connection) **to do
   - [Shared Buffer](#shared-buffer) **to do
##

1. ### Repository Keyring

- Install GnuPG from apt
```sh
root@host:~# apt-get install gnupg2
```

- Get postgres-common from Debian source:
```sh
user@host:~$ wget http://deb.debian.org/debian/pool/main/p/postgresql-common/postgresql-common_225.tar.xz
user@host:~$ tar -xf postgresql-common_225.tar.xz
```

- Copy the required bash script from the extracted folder:
```sh
root@host:~# mkdir -p /usr/share/postgresql-common/pgdg && \
mv /home/user/postgresql-common/pgdg/apt.postgresql.org.sh /usr/share/postgresql-common/pgdg/ && \
chown root:root /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh && \
chmod 755 /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
```

- Run the script to add keys to apt-keyring
```sh
root@host:~# /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
```

2. ### Install PostgreSQL
- Add Packages
```sh
root@host:~# apt-get install postgresql-15
root@host:~# apt-get install postgresql-server-dev-15
```

- Change the postgres user password
```sh
root@host:~# passwd postgres
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

- Create one or more databases (one for each backend)
```sh
postgres@host:~$ createdb -O psqluser mapdb && \
createdb -O psqluser playerdb && \
createdb -O psqluser authdb && \
createdb -O psqluser mapserverdb
```

- Find the database(s) in the dump
```sh
postgres@host:~$ pg_dumpall
```

- Verify you can log in with the user
```sh
postgres@host:~$ psql -d mapdb -h localhost -U psqluser
```

You now have a running PostgreSQL service.


##
<img decoding="async" loading="lazy" src="https://cdn.discordapp.com/emojis/1194038093775376455.webp?size=64&quality=lossless">
mønκ

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