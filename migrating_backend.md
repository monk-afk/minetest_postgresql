Migrating Backend (Incomplete)
=================
Table of Contents
-----------------
   - [Edit world.mt](#edit-worldmt)
   - [Perform Migration](#perform-migration)
##
> Make a backup copy of your files before migrating!
1. ### Edit world.mt
   - Add the pgslq_connection of the backend to be migrated.
   - Do not change the backend type, it will be updated by the migration automation.
   - Migrate only one backend at a time.
```conf
# map
backend = sqlite3
# Socket connection:
pgsql_connection = postgresql:///mapdb?host=/var/run/postgresql&user=psqluser&password=securepassword&dbname=mapdb
gameid = minetest
world_name = world
```

2. ### Perform Migration
```sh
user@host:~/minetest-master$ ./bin/minetestserver --migrate postgresql --world /home/user/minetest-master/worlds/world
```

Depending on the size of your map file, the migration may take several hours.


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