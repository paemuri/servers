# servers

Dockerfiles and configs for my servers.

## Server bootstrapping

First, if there are still no SSH keys for a server, create it with:

```sh
# Example variables.
server=portuga
domain=paemuri.com

ssh-keygen -t ed25519 -f $server
ssh-copy-id -i $server.pub root@$domain
cat <<'EOF' >> ~/.ssh/config
Host $domain
  Port 22069
  IdentityFile ~/$server
EOF
```

Then access the server and configure it.

``` sh
# Example variables (again).
server=portuga
domain=paemuri.com

ssh root@$domain
mkdir /etc/terminfo/f
ln -svn /etc/terminfo/x/xterm-256color /etc/terminfo/f/foot

apk upgrade
apk update
apk add docker rsync

# You can configure SSH by hand if it's preferred.
sed --in-place 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sed --in-place 's/PermitEmptyPasswords no/PermitEmptyPasswords yes/' /etc/ssh/sshd_config
sed --in-place 's/#Port 22/Port 22069/' /etc/ssh/sshd_config

service sshd restart
passwd -d root
exit
```

## Minecraft

It's still a todo. Here's the old script for it:

``` sh
useradd -mrU -s /sbin/nologin -c "minecraft server" minecraft

# scp server folder to /srv/minecraft/server
chown -R minecraft:minecraft /srv/minecraft
mkdir /var/log/minecraft
chown -R minecraft:minecraft /var/log/minecraft

apk add openjdk17

cat <<'EOF' > /etc/init.d/minecraft
#!/sbin/openrc-run

description="minecraft server"
pidfile="/run/minecraft.pid"
output_log="/var/log/minecraft/minecraft.log"
error_log="/var/log/minecraft/minecraft.err"

depend () {
  need net
  after ipsec
}

start() {
  ebegin "Starting Minecraft."
  start-stop-daemon \
    --start \
    --chdir="/srv/minecraft/server" \
    --pidfile="$pidfile" \
    --make-pidfile \
    --user="minecraft" \
    --group="minecraft" \
    --background \
    --exec="/usr/bin/java" \
    -- -Xmx512M -jar /srv/minecraft/server/server.jar nogui
  eend $?
}

stop() {
  ebegin "Stopping Minecraft."
  start-stop-daemon \
    --stop \
    --chdir="/srv/minecraft/server" \
    --pidfile="$pidfile" \
    --user="minecraft" \
    --group="minecraft" \
    --retry=5
  eend $?
}
EOF
chmod +x /etc/init.d/minecraft

rc-update add minecraft default
rc-service minecraft start
```
