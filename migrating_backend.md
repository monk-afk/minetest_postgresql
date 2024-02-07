Migrating Backend (Incomplete)
=================
Table of Contents
-----------------
   - [Incomplete Guide](#migrating-backend)
##
1. ### Edit world.mt
- Add the pgslq connection of the backend to be migrated.
- Migrate only one backend at a time.
- Make sure the backend setting is the current type, by default sqlite3.
```conf
backend = sqlite3
# IP connection
# pgsql_connection = host=127.0.0.1 port=5432 user=psqluser password=securepassword dbname=mapdb
#
# Socket connection:
# pgsql_connection = postgresql:///mapdb?host=/var/run/postgresql&user=psqluser&password=securepassword&dbname=mapdb
gameid = minetest
world_name = world
```

2. ### Perform the migration
```sh
user@host:~/minetest-master$ ./bin/minetestserver --migrate postgresql --world /home/user/minetest-master/worlds/world
```
Or if using multicraft
```sh
user@host:~/MultiCraft-main$ ./bin/multicraftserver --migrate postgresql --world /home/user/MultiCraft-main/worlds/world
```

Depending on the size of your map file, the migration may take several hours.


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