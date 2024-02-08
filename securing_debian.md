Securing Debian 11 (bullseye)
=============================
Table of Contents
------------------
   - [Create User](#create-user)
   - [SSH with Auth Keys](#ssh-with-auth-keys)
   - [Uncomplicated Firewall](#uncomplicated-firewall)
##

1. ### Create User
- Assuming there is no user account on your fresh installed Debian
- Log in to the host server as root
```sh
user@local:~$ ssh root@host
```
- Create new user
```sh
root@host:~# adduser username
```
- Install rsync
```sh
root@host:~# apt-get install rsync
```

2. ### SSH with Auth Keys
> With SSH Keypair Authentication, the server will only allow connections from the user(s) listed in the authorized_keys file.
> [!WARNING]
> After following these steps, root login attempts will be denied, and authentication via password will be disabled. This means *the only way* to connect with SSH is by having the correct username and keypair. Not even a root password reset will allow recovery if your keys are lost!
**Never Share the Private Key! Create a backup of your keypair and store it in a secure location!**

- Create SSH Auth file and folder, change ownership and permissions
```sh
root@host:~# mkdir -p /home/user/.ssh && \
touch /home/user/.ssh/authorized_keys && \
chown -R user:user /home/user/.ssh && \
chmod 700 /home/user/.ssh && \
chmod 600 /home/user/.ssh/authorized_keys
```

- Generate new SSH keypair on local machine
    - Creating with password will ask for authentication when using the keys (good security)
    - Creating without password will authenticate without password (medium security)
    - **DO NOT OVERWRITE EXISTING KEYPAIR!** Default is named `id_rsa` and `id_rsa.pub
```sh
user@local:~$ ssh-keygen -t rsa -b 4096
```

- Add the **public key** to remote host authorized_keys file
```sh
user@local:~$ cat ~/.ssh/id_rsa.pub | ssh user@host -T "cat >> /home/remoteuser/.ssh/authorized_keys"
```

- Edit sshd_config
```sh
root@host:~# nano /etc/ssh/sshd_config
```

- Uncomment or add lines. In this example, a non-standard port (7743) will be used for SSH.
```sh
Port 7743
PasswordAuthentication no
PermitRootLogin no
AuthorizedKeysFile      .ssh/authorized_keys .ssh/authorized_keys2
```

- Restart ssh daemon
```sh
root@host:~# systemctl restart sshd
```

- Now you will need to specify the port when connecting to the remote host
  - ssh
```sh
user@local:~$ ssh -p 7743 user@host
```
  - rsync
```sh
user@local:~$ rsync -av -e 'ssh -p 7743' user@host:/home/user/example.txt ./
```

3. ### Uncomplicated Firewall
> [!NOTE]
> This will prevent all incoming connections except on port 7743 (ssh) and 30000 (minetest). UDP is denied on port 7443 since we don't need it for SSH. These rules will also limit the connection attempts to a defaulted amount of 6 attempts within a 30 second window.

- Install Firewall UFW
```sh
root@host:~# apt-get install ufw
```

- Add rules to firewall. 30000 is the standard Minetest port.
```sh
root@host:~# /usr/sbin/ufw default deny incoming && \
/usr/sbin/ufw deny 7743/udp && \
/usr/sbin/ufw limit 7743/tcp && \
/usr/sbin/ufw limit 30000/udp && \
/usr/sbin/ufw limit 30000/tcp
```

- The firewall rules should look like this:
```sh
root@host:~# /usr/sbin/ufw show added

	ufw limit 7743/tcp
	ufw deny 7743/udp
	ufw limit 30000/tcp
	ufw limit 30000/udp
```

- Enable the Firewall
```sh
root@host:~# /usr/sbin/ufw enable
```

- View Status of UFW
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

You now have a secure host server. Next step is [PostgreSQL Setup](/postgresql_setup.md).


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