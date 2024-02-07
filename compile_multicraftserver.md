Building MultiCraft Server 2.0.6 
================================
With minetest_game (5.4.1)

Please follow [Securing Debian](/securing_debian.md) before starting.

Table of Contents
------------------
   - [Install Dependencies](#install-dependencies)
   - [Make and Compile](#make-and-compile)
   - [world.mt](#worldmt)
   - [minetest.conf](#minetestconf)
   - [Run Server](#run-server)
##
========

1. ### Install Dependencies
```sh
root@host:~# apt install g++ make libc6-dev libirrlicht-dev cmake libbz2-dev libpng-dev libjpeg-dev libxxf86vm-dev libgl1-mesa-dev libsqlite3-dev libogg-dev libvorbis-dev libopenal-dev libcurl4-gnutls-dev libfreetype6-dev zlib1g-dev libgmp-dev libjsoncpp-dev
```

2. ### Make and Compile 
 - In one step, this will compile:
   - MultiCraft (server) https://github.com/MultiCraft/MultiCraft
   - Minetest_Game (5.4.1) https://github.com/minetest/minetest_game/releases/tag/5.4.1
```sh
user@host:~$ wget https://github.com/MultiCraft/MultiCraft/archive/main.tar.gz && \
tar -xzf MultiCraft-main.tar.gz && rm MultiCraft-main.tar.gz && \
cd MultiCraft-main/games/ && \
wget https://github.com/minetest/minetest_game/archive/refs/tags/5.4.1.tar.gz && \
tar -xzf 5.4.1.tar.gz && rm 5.4.1.tar.gz && \
mv minetest_game-5.4.1 minetest_game && cd .. && \
cmake . -DRUN_IN_PLACE=TRUE -DBUILD_CLIENT=FALSE -DBUILD_SERVER=TRUE -DENABLE_POSTGRESQL=ON -DPostgreSQL_TYPE_INCLUDE_DIR=/usr/include/postgresql/15/server/libpq && \
make -j$(nproc)
```

When compiling is complete, the `multicraftserver` binary can be found in ~/MultiCraft-main/bin/

3. ### world.mt
- MultiCraft only supports 'file' type mod_storage
- Default backend type is SQLite3

- Configure `~/MultiCraft-main/worlds/yourworld/world.mt` file to include postgresql backend(s)
- Each backend type (map, players, auth, mapserver) requires it's own database
- **Make sure to change the username, password, and dbname to your own setting!**
```conf
# map
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
```
Mapserver is optional, and not yet covered by this tutorial. Source can be found here: https://github.com/minetest-mapserver/mapserver

4. ### minetest.conf
- Configure `~/MultiCraft-main/minetest.conf` with the settings you want, here are a few to get started
```conf
## Server
name = FullPrivsADMIN
server_announce = false
server_address = yourdomain.com 
server_name = AFK Server
port = 30000
max_users = 64
serverlist_url = servers.multicraft.world
announce_mt = true

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
A complete list of config settings can be found here: https://github.com/MultiCraft/MultiCraft/blob/63cf8adad14318b25ccfe5a7557ac7870ceed753/builtin/settingtypes.txt#L1020

4. ### Run Server
```sh
user@host:~/MultiCraft-main$ ./bin/multicraftserver --world ./worlds/yourworld --config ./minetest.conf --gameid minetest_game
```

You now have a functional MultiCraft server.


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