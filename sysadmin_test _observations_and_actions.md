### Password policy.

**Note:** This was not implemented as I'm scared I restrict access from the system. I checked the `/root/.ssh` directory and no contents exist, meaning nobody has a ssh access to the system without a password. 

To ensure that "interns" do not wreak havoc on systems by gaining access from written down passwords, the following is recommended. 

### Disable password login for shell sessions.

1. Create user accounts for all parties that need access to the system.
2. Either add users to the default `admin` group to allow them sudo priviliges, or create a new file for each group or individual user in `/etc/sudoers.d`

	The format will be:
	
	```bash
	Defaults:<username> !fqdn
	Defaults:<username> !requiretty
	<username> ALL=(ALL) NOPASSWD: ALL
	```
	
	or 
	```bash
	Defaults:<%groupname> !fqdn
	Defaults:<%groupname> !requiretty
	<%groupname> ALL=(ALL) NOPASSWD: ALL
	```


3. generate ssh key-pairs for each user and add the public keys into `~/.ssh/authorized_keys` for each user instead of the root account.
	1. Make sure you copy the keys and distribute them to relevant users so that they can use the private keys to establish ssl connections. 
4. Disable password login
	1. edit `/etc/ssh/sshd_config` and find the line cotaining `PasswordAuthentication yes` and change it to `PasswordAuthentication no` . If the line starts with a `#`, be sure to uncomment it.
	2. restart the sshd by running `systemctl restart ssh
5. NB: Remember to change the existing root password to prevent access to any current of future applications using the root password, i.e. webmin or cockpit. 




### Bash profile for user account.

-bash: /etc/profile.d/bash_completion.sh: line 14: syntax error near unexpected token `fi'

This file was missing a few lines and replaced contents with example from own ubuntu server. 

path variable set to wrong path in **/etc/environment.** Commented line is the wrong one, active one is the correct one. Copied this from own ubuntu server.
 ```bash
#export PATH="/ldr"
export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin"

```

Replaced .bashrc and .profile in /root with defaults from /etc/skel/ to ensure that there aren't anymore surprises.

**apt** 
`/etc/apt/sources.list` was cleared of most contents.  Replaced this with default sources from own ubuntu server and also add docker repo.  Also made a copy of the file and placed it in `/root/sources.list` to ensure there's a backup


Docker.

Checked logs using `journalctl -b u docker`  to see docker logs since last boot, and noticed error: `unknown flag: --containerd`

Tried to start docker manually running dockerd, but had components missing. Managed to install the missing components, `containerd`  package, but still had the same issue due to other missing components. 

Docker has managed to start after deciding to perform a fresh install of docker and it's services as recommend by docker's docs. 
1. sudo apt-get remove docker docker-engine docker.io containerd runc
2. apt update
3. apt-get install ca-certificates curl gnupg lsb-release
4. Replace existing GPG key for docker repo
	1. curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
	2. echo   "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \ $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
5. apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin
6. systemctl enable docker

Docker Compose:

Volumes' syntax was incorrect. 

Changed 
```docker
volumes:
  db_data
  wordpress_data
```

to 

```docker
volumes:
  db_data: {}
  wordpress_data: {}

```

using `docker compose up -d` started the wordpress images 
_________________________________________________________________________________________________________________________________________________________
root@test20:/var/log# docker ps
CONTAINER ID   IMAGE              COMMAND                  CREATED          STATUS          PORTS                                           NAMES
c30897de24ab   wordpress:latest   "docker-entrypoint.s…"   15 minutes ago   Up 15 minutes   80/tcp, 0.0.0.0:80->8000/tcp, :::80->8000/tcp   containers-wordpress-1
af439eed0e08   mysql:5.7          "docker-entrypoint.s…"   15 minutes ago   Up 15 minutes   3306/tcp, 33060/tcp                             containers-db-1
_________________________________________________________________________________________________________________________________________________________

Default wordpress docker image listens on port 80. Changed mappings for docker network to listen on 8000 and forward to instead of 80 to 8000.

