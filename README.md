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
