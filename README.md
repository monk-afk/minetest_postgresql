Minetest Server with PostgreSQL Tutorial
========================================

- Requires root and ssh access
- Assumes a clean installation of Debian 11 (bullseye)
- Written for Minetest 5.9-dev and PostgreSQL 15
- Pay attention to the user@where and ~/directory!
- Localhost commands: `user@local`/`root@local`
- Remote server commands: `user@host`/`root@host`

Table of Contents
------------------
1. [Securing Debian](/securing_debian.md)
   - [Create User](/securing_debian.md#create-user)
   - [SSH with Auth Keys](/securing_debian.md#ssh-with-auth-keys)
   - [Uncomplicated Firewall](/securing_debian.md#uncomplicated-firewall)
   
2. [PostgreSQL Setup](/postgresql_setup.md)
   - [Repository Keyring](/postgresql_setup.md#repository-keyring)
   - [Install PostgreSQL](/postgresql_setup.md#install-postgresql)
   - [User Database](/postgresql_setup.md#user-database)
   - [Socket Connection](/postgresql_setup.md#socket-connection)
   - [Migrating Backend](/postgresql_setup.md#migrating_backend)
   - [Creating Backups](/postgresql_setup.md#creating-backups)

3. [Minetest Server](/compile_minetestserver.md)
   - [Install Dependencies](/compile_minetestserver.md#install-dependencies)
   - [Make and Compile](/compile_minetestserver.md#make-and-compile)
   - [world.mt](/compile_minetestserver.md#worldmt)
   - [minetest.conf](/compile_minetestserver.md#minetestconf)
   - [Run Server](/compile_minetestserver.md#run-server)

4. [MultiCraft Server](/compile_multicraftserver.md)
   - [Install Dependencies](/compile_multicraftserver.md#install-dependencies)
   - [Make and Compile](/compile_multicraftserver.md#make-and-compile)
   - [world.mt](/compile_multicraftserver.md#worldmt)
   - [minetest.conf](/compile_multicraftserver.md#minetestconf)
   - [Run Server](/compile_multicraftserver.md#run-server)
   

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