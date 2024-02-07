Minetest Server with PostgreSQL Tutorial
========================================
- Requires root and ssh access
- Assumes a clean installation of Debian 11 (bullseye)
- Written for Minetest 5.9-dev and PostgreSQL 15
- Pay attention to the user@where and ~/directory!
- Localhost commands: `user@local`/`root@local`
- Remote server commands: `user@host`/`root@host`
##
Table of Contents
------------------
1. [Debian](#debian)
   - [SSH with Auth Keys](#ssh-with-auth-keys)
   - [Uncomplicated Firewall](#uncomplicated-firewall)
2. [PostgreSQL](#postgresql)
   - [Repository Keyring](#repository-keyring)
   - [User Database](#user-database)
3. [Minetest](#minetest)
   - [Make and Compile](#make-and-compile)
   - [Configuration](#configuration)
4. [License](#license)

##
Debian
------
1. Log in to the host server as root
```sh
user@local:~$ ssh root@host
```

2. Create new user
```sh
root@host:~# adduser username
```

3. Install rsync
```sh
root@host:~# apt-get install rsync
```

### SSH with Auth Keys
With SSH Keypair Authentication, the server will only allow connections from the user(s) listed in the authorized_keys file.

> [!WARNING]
> After following these steps, root login attempts will be denied, and authentication via password will be disabled. This means *the only way* to connect with SSH is by having the correct username and keypair. Not even a root password reset will allow recovery if your keys are lost!
**Never Share the Private Key! Create a backup of your keypair and store it in a secure location!**

4. Create SSH Auth file and folder, change ownership and permissions
```sh
root@host:~# mkdir -p /home/user/.ssh && \
touch /home/user/.ssh/authorized_keys && \
chown -R user:user /home/user/.ssh && \
chmod -R 700 /home/user/.ssh
```

5. Generate new SSH keypair on local machine
    - Creating with password will ask for authentication when using the keys (good security)
    - Creating without password will authenticate without password (medium security)
    - **DO NOT OVERWRITE EXISTING KEYPAIR!** Default is named `id_rsa` and `id_rsa.pub
```sh
user@local:~$ ssh-keygen -t rsa -b 4096
```

6. Add the **public key** to remote host authorized_keys file
```sh
user@local:~$ cat ~/.ssh/id_rsa.pub | ssh user@host -T "cat >> /home/remoteuser/.ssh/authorized_keys"
```

7. Edit sshd_config
```sh
root@host:~# nano /etc/ssh/sshd_config
```

8. Uncomment or add lines. In this example, a non-standard port (7743) will be used for SSH.
```sh
Port 7743
PasswordAuthentication no
PermitRootLogin no
AuthorizedKeysFile      .ssh/authorized_keys .ssh/authorized_keys2
```

9. Restart ssh daemon
```sh
root@host:~# systemctl restart sshd
```

Now you will need to specify the port when connecting to the remote host

- ssh:
```sh
user@local:~$ ssh -p 7743 user@host
```
- rsync
```sh
user@local:~$ rsync -av -e 'ssh -p 7743' user@host:/home/user/example.txt ./
```

### Uncomplicated Firewall

This will prevent all incoming connections except on port 7743 (ssh) and 30000 (minetest). UDP is denied on port 7443 since we don't need it for SSH. These rules will also limit the connection attempts to a defaulted amount of 6 attempts within a 30 second window.

10. Install Firewall UFW
```sh
root@host:~# apt-get install ufw
```

11. Add rules to firewall. 30000 is the standard Minetest port.
```sh
root@host:~# /usr/sbin/ufw default deny incoming && \
/usr/sbin/ufw deny 7743/udp && \
/usr/sbin/ufw limit 7743/tcp && \
/usr/sbin/ufw limit 30000/udp && \
/usr/sbin/ufw limit 30000/tcp
```

The firewall rules should look like this:
```sh
root@host:~# /usr/sbin/ufw show added

	ufw limit 7743/tcp
	ufw deny 7743/udp
	ufw limit 30000/tcp
	ufw limit 30000/udp
```

12. Enable the Firewall
```sh
root@host:~# /usr/sbin/ufw enable
```
```sh
root@host:~# /usr/sbin/ufw status
Status: active

To                         Action      From
--                         ------      ----
7743/tcp                   LIMIT       Anywhere                  
7743/udp                   DENY        Anywhere                  
30000/tcp                  LIMIT       Anywhere                  
30000/udp                  LIMIT       Anywhere                  
7743/tcp (v6)              LIMIT       Anywhere (v6)             
7743/udp (v6)              DENY        Anywhere (v6)             
30000/tcp (v6)             LIMIT       Anywhere (v6)             
30000/udp (v6)             LIMIT       Anywhere (v6)             
```

You now have a secure host server.
##

PostgreSQL
----------

### Repository Keyring

1. Get postgres-common from Debian source:
```sh
user@host:~$ wget http://deb.debian.org/debian/pool/main/p/postgresql-common/postgresql-common_225.tar.xz
user@host:~$ tar -xf postgresql-common_225.tar.xz
```

2. Copy the required bash script from the extracted folder:
```sh
root@host:~# mkdir -p /usr/share/postgresql-common/pgdg && \
mv /home/user/postgresql-common/pgdg/apt.postgresql.org.sh /usr/share/postgresql-common/pgdg/ && \
chown root:root /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh && \
chmod 755 /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
```

3. Run the script to add keys to apt-keyring
```sh
root@host:~# /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
```

4. Now you can install
```sh
root@host:~# apt-get install postgresql-15
root@host:~# apt-get install postgresql-server-dev-15
```

### User Database

5. Change the postgres user password
```sh
root@host:~# passwd postgres
```

6. Verify the service is running, and start it if not
```sh
user@host:~$ /etc/init.d/postgresql status
user@host:~$ /etc/init.d/postgresql start
```

7. Log in as postgres user
```sh
user@host:~$ su - postgres
```

8. Create database user
```sh
postgres@host:~$ createuser --pwprompt psqluser
```

9. Create one or more databases (one for each backend)
```sh
postgres@host:~$ createdb -O psqluser mapdb && \
createdb -O psqluser playerdb && \
createdb -O psqluser authdb && \
createdb -O psqluser mapserverdb
```

10. Verify you can log in with the user
```sh
postgres@host:~$ psql -d mapdb -h localhost -U psqluser
```
##

Minetest
========
These steps will build the master branches of: Minetest (Server), Minetest_Game, and Irrlicht.

1. Get the required dependencies
```sh
root@host:~# apt install g++ make libc6-dev cmake libpng-dev libjpeg-dev libxi-dev libgl1-mesa-dev libsqlite3-dev libogg-dev libvorbis-dev libopenal-dev libcurl4-gnutls-dev libfreetype6-dev zlib1g-dev libgmp-dev libjsoncpp-dev libzstd-dev libluajit-5.1-dev gettext libsdl2-dev
```

### Make and Compile 
2. In one step:
```sh
user@host:~$ wget https://github.com/minetest/minetest/archive/master.tar.gz && \
tar -xzf master.tar.gz && rm master.tar.gz && \
cd minetest-master/lib/ && \
wget https://github.com/minetest/irrlicht/archive/master.tar.gz && \
tar -xzf master.tar.gz && rm master.tar.gz && \
mv irrlicht-master irrlichtmt && \
cd ../games/ && \
wget https://github.com/minetest/minetest_game/archive/master.tar.gz && \
tar -xzf master.tar.gz && rm master.tar.gz && \
mv minetest_game-master minetest_game && \
cd .. && \
cmake . -DRUN_IN_PLACE=TRUE -DBUILD_CLIENT=FALSE -DBUILD_SERVER=TRUE -DENABLE_POSTGRESQL=ON -DIRRLICHT_INCLUDE_DIR=${HOME}/minetest-master/lib/irrlichtmt/include && \
make -j$(nproc)
```

When compiling is complete, the `minetestserver` binary can be found in ~/minetest-master/bin/

### Configuration
- Configure world.mt file to include postgresql backend(s)
- Each backend type (map, players, auth, mapserver) requires it's own database
- It's recommended to use sqlite3 for mod_storage for performance reason
- **Make sure to change the username, password, and dbname to your own setting!**

```sh
user@host:~$ echo "# map
backend = postgresql
pgsql_connection = host=127.0.0.1 port=5432 user=psqluser password=securepassword dbname=mapdb
# auth
auth_backend = postgresql
pgsql_auth_connection = host=127.0.0.1 port=5432 user=psqluser password=securepassword dbname=authdb
# players
player_backend = postgresql
pgsql_player_connection = host=127.0.0.1 port=5432 user=psqluser password=securepassword dbname=playerdb
# mapserver
pgsql_mapserver_connection = host=127.0.0.1 port=5432 user=psqluser password=securepassword dbname=mapserverdb
" >> ~/minetest-master/worlds/yourworld/world.mt
```
Mapserver is optional, and not covered by this tutorial. Source can be found here: https://github.com/minetest-mapserver/mapserver

3. Configure minetest.conf with the settings you want, here is a few to get started
```sh
user@host:~$ echo "## Server
name = FullPrivsADMIN
server_announce = false
server_address = yourdomain.com 
server_name = AFK Server
port = 30000
max_users = 64

## Game
default_game = minetest
static_spawnpoint = 0,12,0
time_speed = 50
creative_mode = false
enable_damage = true
enable_pvp = true
default_privs = shout, interact
give_initial_stuff = true
initial_stuff = default:torch 99,default:apple 99,default:pick_steel,default:axe_steel

## Logging
profiler.load = true
debug_log_level = action
debug_log_size_max = 5000
log_mod_memory_usage_on_load = true
log_mods = true
enable_mapgen_debug_info = true

## Security
disallow_empty_password = true
disable_anticheat = false
secure.enable_security = true
csm_restriction_flags = 63
csm_restriction_noderange = 1
enable_client_modding = false" >> ~/minetest-master/minetest.conf
```
A complete list of config settings can be found here: https://github.com/minetest/minetest/blob/master/minetest.conf.example#L700

4. Start the server, and enjoy.
```sh
user@host:~/minetest-master$ ./bin/minetestserver --world ./worlds/yourworld --config ./minetest.conf --gameid minetest_game
```

##
<img decoding="async" loading="lazy" src="https://cdn.discordapp.com/emojis/1194038093775376455.webp?size=64&quality=lossless">
mønκ

License
-------
CC0 1.0 Universal

No Copyright

The person who associated a work with this deed has dedicated the work to the public domain by waiving all of his or her rights to the work worldwide under copyright law, including all related and neighboring rights, to the extent allowed by law.

You can copy, modify, distribute and perform the work, even for commercial purposes, all without asking permission.

In no way are the patent or trademark rights of any person affected by CC0, nor are the rights that other persons may have in the work or in how the work is used, such as publicity or privacy rights.

Unless expressly stated otherwise, the person who associated a work with this deed makes no warranties about the work, and disclaims liability for all uses of the work, to the fullest extent permitted by applicable law.
When using or citing the work, you should not imply endorsement by the author or the affirmer.