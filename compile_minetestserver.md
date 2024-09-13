Building Minetest Server
========================

Steps to compile Minetest server from source.

Please follow [Securing Debian](/securing_debian.md) before starting.

Table of Contents
------------------
   - [Install Dependencies](#install-dependencies)
   - [Make and Compile](#make-and-compile)
   - [world.mt](#worldmt)
   - [minetest.conf](#minetestconf)
   - [Run Server](#run-server)

___

1. ### Install dependencies

```sh
root@host:~# apt update ; apt install g++ make libc6-dev cmake libpng-dev libjpeg-dev libgl1-mesa-dev libsqlite3-dev libogg-dev libvorbis-dev libopenal-dev libcurl4-gnutls-dev libfreetype6-dev zlib1g-dev libgmp-dev libjsoncpp-dev libzstd-dev libluajit-5.1-dev gettext libsdl2-dev
```

2. ### Make and Compile 
  - Download Minetest Source https://github.com/minetest/minetest
    - Development (unstable) https://github.com/minetest/minetest/archive/master.tar.gz
    - Production (stable) https://github.com/minetest/minetest/archive/refs/heads/stable-5.tar.gz

   - Download Minetest_Game https://github.com/minetest/minetest_game

   - ~~Irrlicht https://github.com/minetest/irrlicht/~~ *IrrlichtMt was merged into the Minetest source tree and is no longer required as a separate dependency or download.*

```sh
# Download and extract minetest
wget -O - https://github.com/minetest/minetest/archive/refs/heads/stable-5.tar.gz | tar -xzf -

# Download and extract minetest_game
wget -O - https://github.com/minetest/minetest_game/archive/refs/tags/5.8.0.tar.gz | tar -xzf -

# Place the minetest_game folder into the games folder
mv minetest_game-5.8.0 minetest-stable-5/games/minetest_game

# Move into the minetest engine folder
cd minetest-stable-5

# Make with options, https://github.com/minetest/minetest/blob/master/doc/compiling/README.md
cmake . -DRUN_IN_PLACE=TRUE -DBUILD_CLIENT=FALSE -DBUILD_SERVER=TRUE -DENABLE_POSTGRESQL=ON -DPostgreSQL_TYPE_INCLUDE_DIR=/usr/include/postgresql/15/server/libpq -DENABLE_CURSES=ON -DENABLE_LUAJIT=ON && \
make -j$(nproc)
```

When compiling is complete, the `minetestserver` binary can be found in ~/minetest-master/bin/

3. ### world.mt
- Default backend type is SQLite3
- Configure `~/minetest-master/worlds/yourworld/world.mt` file to include postgresql backend(s)
- Each backend type (map, players, auth, mapserver) requires its own database
- **Make sure to change the username, password, and dbname to your own setting!**
- Mapserver is optional, and not yet covered by this tutorial. Source can be found here: https://github.com/minetest-mapserver/mapserver

For Socket Connection (faster)

```conf
# map
backend = postgresql
pgsql_connection = postgresql:///mapdb?host=/var/run/postgresql&user=psqluser&password=securepassword&dbname=mapdb
# auth
auth_backend = postgresql
pgsql_auth_connection = postgresql:///authdb?host=/var/run/postgresql&user=psqluser&password=securepassword&dbname=authdb
# player
player_backend = postgresql
pgsql_player_connection = postgresql:///playerdb?host=/var/run/postgresql&user=psqluser&password=securepassword&dbname=playerdb
# mapserver
pgsql_mapserver_connection = postgresql:///mapserverdb?host=/var/run/postgresql&user=psqluser&password=securepassword&dbname=mapserverdb
```

For IP connection

```conf
# map
backend = postgresql
pgsql_connection = host=127.0.0.1 port=5432 user=psqluser password=securepassword dbname=mapdb
# auth
auth_backend = postgresql
pgsql_auth_connection = host=127.0.0.1 port=5432 user=psqluser password=securepassword dbname=authdb
# player
player_backend = postgresql
pgsql_player_connection = host=127.0.0.1 port=5432 user=psqluser password=securepassword dbname=playerdb
# mapserver
pgsql_mapserver_connection = host=127.0.0.1 port=5432 user=psqluser password=securepassword dbname=mapserverdb
```


4. ### minetest.conf
- Configure `~/minetest-master/minetest.conf` with the settings you want, here is a few to get started
```conf
## Server
name = FullPrivsADMIN
server_announce = false
server_address = yourdomain.com 
server_name = AFK Server
server_description = """
Welcome to AFKS!
AFK All Day Every Day!
"""
port = 30000
max_users = 64
serverlist_url = servers.minetest.net

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
enable_client_modding = false
```
A complete list of config settings can be found here: https://github.com/minetest/minetest/blob/master/minetest.conf.example#L700

5. ### Run Server

```sh
user@host:~/minetest-master$ ./bin/minetestserver --world ./worlds/yourworld --config ./minetest.conf --gameid minetest_game
```

You now have a functional Minetest server.


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
