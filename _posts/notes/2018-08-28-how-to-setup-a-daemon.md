---
layout: post
title: How to setup a daemon
date: 2018-08-28 22:05 +0800
tags:
  - SysAdmin
---

Setting up a daemon to run your program in the background.


{% include toc.html %}

## Debian

Edit `/etc/init.d/example`
```shell
#!/bin/sh

### BEGIN INIT INFO
# Provides:          exampledaemon
# Required-Start:    $local_fs $network $syslog
# Required-Stop:     $local_fs $network $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Example
# Description:       Example start-stop-daemon - Debian
### END INIT INFO

NAME="example"
PATH="/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin"
APPDIR="/"
APPBIN="/usr/bin/python"
APPARGS="-m SimpleHTTPServer 8000"
USER="example"
GROUP="example"

# Include functions 
set -e
. /lib/lsb/init-functions

start() {
  printf "Starting '$NAME'... "
  start-stop-daemon --start --chuid "$USER:$GROUP" --background --make-pidfile --pidfile /var/run/$NAME.pid --chdir "$APPDIR" --exec "$APPBIN" -- $APPARGS || true
  printf "done\n"
}

#We need this function to ensure the whole process tree will be killed
killtree() {
    local _pid=$1
    local _sig=${2-TERM}
    for _child in $(ps -o pid --no-headers --ppid ${_pid}); do
        killtree ${_child} ${_sig}
    done
    kill -${_sig} ${_pid}
}

stop() {
  printf "Stopping '$NAME'... "
  [ -z `cat /var/run/$NAME.pid 2>/dev/null` ] || \
  while test -d /proc/$(cat /var/run/$NAME.pid); do
    killtree $(cat /var/run/$NAME.pid) 15
    sleep 0.5
  done 
  [ -z `cat /var/run/$NAME.pid 2>/dev/null` ] || rm /var/run/$NAME.pid
  printf "done\n"
}

status() {
  status_of_proc -p /var/run/$NAME.pid "" $NAME && exit 0 || exit $?
}

case "$1" in
  start)
    start
    ;;
  stop)
    stop
    ;;
  restart)
    stop
    start
    ;;
  status)
    status
    ;;
  *)
    echo "Usage: $NAME {start|stop|restart|status}" >&2
    exit 1
    ;;
esac

exit 0
```

Enable

```shell
sudo chmod +x /etc/init.d/example
sudo update-rc.d example defaults
sudo service example start
```

Remove

```shell
sudo rm /etc/init.d/example
sudo systemctl daemon-reload
```

## Systemd Daemon

https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/sect-managing_services_with_systemd-unit_files

Example: `/usr/lib/systemd/system/postifix.service`

```
[Unit]
Description=Postfix Mail Transport Agent
After=syslog.target network.target
Conflicts=sendmail.service exim.service

[Service]
Type=forking
PIDFile=/var/spool/postfix/pid/master.pid
EnvironmentFile=-/etc/sysconfig/network
ExecStartPre=-/usr/libexec/postfix/aliasesdb
ExecStartPre=-/usr/libexec/postfix/chroot-update
ExecStart=/usr/sbin/postfix start
ExecReload=/usr/sbin/postfix reload
ExecStop=/usr/sbin/postfix stop

[Install]
WantedBy=multi-user.target
```

Commands

```shell
touch /etc/systemd/system/name.service
chmod 664 /etc/systemd/system/name.service
```


Template: `/etc/systemd/system/name.service`

```
[Unit]
Description=service_description
After=network.target

[Service]
ExecStart=path_to_executable
Type=forking
PIDFile=path_to_pidfile

[Install]
WantedBy=default.target
```

Reload

```shell
systemctl daemon-reload
systemctl start name.service
```

Another example: `/etc/systemd/system/emacs.service`

```
[Unit]
Description=Emacs: the extensible, self-documenting text editor
           
[Service]
Type=forking
ExecStart=/usr/bin/emacs --daemon
ExecStop=/usr/bin/emacsclient --eval "(kill-emacs)"
Environment=SSH_AUTH_SOCK=%t/keyring/ssh
Restart=always
           
[Install]
WantedBy=default.target
```

https://www.devdungeon.com/content/creating-systemd-service-files

## systemctl

```shell
# Control whether service loads on boot
systemctl enable
systemctl disable

# Manual start and stop
systemctl start
systemctl stop

# Restarting/reloading
systemctl daemon-reload # Run if .service file has changed
systemctl restart

# See if running, uptime, view latest logs
systemctl status
systemctl status [service_name]

# See all systemd logs
journalctl

# Tail logs and follow
journalctl -f

# Show logs for specific service
journalctl -u my_daemon.service
```

## Reference

Please refer to [scripts](https://github.com/frdmn/service-daemons/) and [tutorial](https://blog.frd.mn/how-to-set-up-proper-startstop-services-ubuntu-debian-mac-windows/) by [this nice guy](https://github.com/frdmn).
