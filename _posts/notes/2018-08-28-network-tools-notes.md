---
layout: post
title: Network Tools Notes
date: 2018-08-28 23:09 +0800
tags:
  - Network
---

Notes of several network tools on Linux.


{% include toc.html %}


## `ncat`

`ncat` is not `nc`, but a more powerful `nc`.

### HTTP and HTTPS requests

```shell
# HTTP
echo -en 'GET / HTTP/1.1\r\nHost: github.com\r\n\r\n' | ncat github.com 80

# HTTPS
echo -en 'GET / HTTP/1.1\r\nHost: github.com\r\n\r\n' | ncat --ssl github.com 443
```

### HTTP server

```shell
ncat -l 8080 -c "printf 'HTTP/1.1 200 OK\r\n\r\n'; echo '<h1>Fuck</h1>'"
ncat -lk 8080 -c "printf 'HTTP/1.1 200 OK\r\n\r\n'; echo '<h1>Fuck</h1>'" # permanent
```

### TCP proxy

```shell
ncat -lk 8080 -c 'ncat --ssl github.com 443' -o stdio.log -v 2> stderr.log
curl http://0:8080 -v -H 'Host: github.com' # this is a raw HTTP request
```

### Connecting two clients

```shell
ncat -l -p 8080 -c 'ncat -l -p 9090'
```

### Connecting two servers

```shell
ncat localhost 8080 -c 'ncat localhost 9090'
```

### Telnet

```shell
ncat -t remote.net 23
```

### Chat room

```shell
ncat -l 10101 --chat # serve
ncat 127.0.0.1 10101 # join

# SSL
ncat -l 10101 --chat --ssl # serve under SSL
ncat 127.0.0.1 10101 --ssl # join
```

### Sending files

```shell
# UDP
ncat -lu 9999 > passwd # stop it manually because of UDP
ncat -u 127.0.0.1 9999 < /etc/passwd

# TCP
ncat -l 9999 > passwd
ncat 127.0.0.1 9999 < /etc/passwd

# SSL
ncat -l 9999 --ssl > passwd
ncat 127.0.0.1 9999 --ssl < /etc/passwd
ncat 127.0.0.1 9999 --ssl --send-only < /etc/passwd
```

### Access control

```shell
ncat -l 80 --allow '127.0.0.1,192.168.*.*'
ncat -l 80 --allowfile whitelist.txt # separated by \n
ncat -l 80 --deny '192.168.*.*'
ncat -l 80 --denyfile blacklist.txt # separated by \n
```

### Bind shell

```shell
ncat -l 8888 -e /bin/bash
ncat -l 8888 -c 'bash -i 2>&1'
```

### Reverse shell

```shell
ncat -l 8888

ncat 127.0.0.1 8888 -e /bin/bash
ncat 127.0.0.1 8888 -c 'bash -i 2>&1'
```

### Help

```
Ncat 7.60 ( https://nmap.org/ncat )
Usage: ncat [options] [hostname] [port]

Options taking a time assume seconds. Append 'ms' for milliseconds,
's' for seconds, 'm' for minutes, or 'h' for hours (e.g. 500ms).
  -4                         Use IPv4 only
  -6                         Use IPv6 only
  -U, --unixsock             Use Unix domain sockets only
  -C, --crlf                 Use CRLF for EOL sequence
  -c, --sh-exec <command>    Executes the given command via /bin/sh
  -e, --exec <command>       Executes the given command
      --lua-exec <filename>  Executes the given Lua script
  -g hop1[,hop2,...]         Loose source routing hop points (8 max)
  -G <n>                     Loose source routing hop pointer (4, 8, 12, ...)
  -m, --max-conns <n>        Maximum <n> simultaneous connections
  -h, --help                 Display this help screen
  -d, --delay <time>         Wait between read/writes
  -o, --output <filename>    Dump session data to a file
  -x, --hex-dump <filename>  Dump session data as hex to a file
  -i, --idle-timeout <time>  Idle read/write timeout
  -p, --source-port port     Specify source port to use
  -s, --source addr          Specify source address to use (doesn't affect -l)
  -l, --listen               Bind and listen for incoming connections
  -k, --keep-open            Accept multiple connections in listen mode
  -n, --nodns                Do not resolve hostnames via DNS
  -t, --telnet               Answer Telnet negotiations
  -u, --udp                  Use UDP instead of default TCP
      --sctp                 Use SCTP instead of default TCP
  -v, --verbose              Set verbosity level (can be used several times)
  -w, --wait <time>          Connect timeout
  -z                         Zero-I/O mode, report connection status only
      --append-output        Append rather than clobber specified output files
      --send-only            Only send data, ignoring received; quit on EOF
      --recv-only            Only receive data, never send anything
      --allow                Allow only given hosts to connect to Ncat
      --allowfile            A file of hosts allowed to connect to Ncat
      --deny                 Deny given hosts from connecting to Ncat
      --denyfile             A file of hosts denied from connecting to Ncat
      --broker               Enable Ncat's connection brokering mode
      --chat                 Start a simple Ncat chat server
      --proxy <addr[:port]>  Specify address of host to proxy through
      --proxy-type <type>    Specify proxy type ("http" or "socks4" or "socks5")
      --proxy-auth <auth>    Authenticate with HTTP or SOCKS proxy server
      --ssl                  Connect or listen with SSL
      --ssl-cert             Specify SSL certificate file (PEM) for listening
      --ssl-key              Specify SSL private key (PEM) for listening
      --ssl-verify           Verify trust and domain name of certificates
      --ssl-trustfile        PEM file containing trusted SSL certificates
      --ssl-ciphers          Cipherlist containing SSL ciphers to use
      --ssl-alpn             ALPN protocol list to use.
      --version              Display Ncat's version information and exit

See the ncat(1) manpage for full options, descriptions and usage examples
```

## `nmap`

### Links

1. https://www.stationx.net/nmap-cheat-sheet/
2. https://hackertarget.com/nmap-cheatsheet-a-quick-reference-guide/


## `stunnel`

`stunnel` is an SSL tunnel tool.

### Server

Install

```shell
apt install stunnel4
# Modify: /etc/default/stunnel4 ENABLE=1
```

Certificate

```shell
# private key
openssl genrsa -out /etc/stunnel/key.pem 4096

# certificate
openssl req -new -x509 \
    -key /etc/stunnel/key.pem \
    -out /etc/stunnel/cert.pem \
    -days 1826

# combine
cat /etc/stunnel/key.pem /etc/stunnel/cert.pem > /etc/stunnel/private.pem

chmod 640 /etc/stunnel/key.pem /etc/stunnel/cert.pem /etc/stunnel/private.pem
```

Edit `/etc/stunnel/service.conf`

```
cert = /etc/stunnel/private.pem
pid = /var/run/stunnel.pid
[service]
accept = <client_ip>:<listen_port>
connect = <target_ip>:<target_port>
```

Start

```shell
/etc/init.d/stunnel4 start
```

### Client

Install

```shell
apt install stunnel4
# Modify: /etc/default/stunnel4 ENABLE=1
```

Certificate

- Copy `private.pem` from server to `/etc/stunnel/private.pem`

Edit `/etc/stunnel/service-client.conf`

```
cert = /etc/stunnel/private.pem
client = yes
pid = /var/run/stunnel.pid
[service]
accept = <listen_ip>:<listen_port>
connect = <server_ip>:<server_port>
```

Start

```shell
/etc/init.d/stunnel4 start
```



## `socat`

Socat is a command line based utility that establishes two bidirectional byte streams and transfers data between them.

### Serving an executable

```shell
socat tcp-listen:55688,reuseaddr,fork exec:python,echo=0,pty,stderr
socat -d -d tcp-listen:10000,reuseaddr,fork system:'/usr/bin/env python',echo=0,pty,stderr
```

### Port forwarding

```shell
socat tcp-listen:8080,reuseaddr tcp:127.0.0.1:80
```

### OpenSSL

```shell
genkey() {
    filename=$1
    openssl genrsa -out $1.key 4096
    openssl req -new -key $1.key -x509 -days 3653 -out $1.crt
    cat $1.key $1.crt > $1.pem
    chmod 600 $1.key $1.pem
}
genkey client
genkey server

# server
socat openssl-listen:4433,reuseaddr,cert=$HOME/etc/server.pem,cafile=$HOME/etc/client.crt echo 

# client
socat stdio openssl-connect:server.domain.org:4433,cert=$HOME/etc/client.pem,cafile=$HOME/etc/server.crt
```


## `tcpdump`

`tcpdump` is a common packet analyzer that runs under the command line.

### Basic

```shell
# Capture N packets
tcpdump -c N

# Write to file
tcpdump -w test.pcap

# Read from file
tcpdump -r test.pcap

# Specify interface
tcpdump -i eth0

# List interfaces
tcpdump -D
```

### Filter

```shell
tcpdump tcp
tcpdump tcp and host 192.168.0.109
tcpdump not icmp and not arp and host 192.168.0.109
tcpdump port 80
tcpdump portrange 80-10000
tcpdump dst port 80
tcpdump src host 192.168.0.1
tcpdump src net 192.168.0.0/24
tcpdump ether host aa:bb:cc:11:22:33
tcpdump ether src aa:bb:cc:11:22:33
tcpdump 'dst port 80 and tcp[13] & 8 != 0' # HTTP data
```

### Display

```shell
# Display with IP addresses
tcpdump -n

# Display with snapshot length = N
tcpdump -s N

# Display more information
tcpdump -v
tcpdump -vv
tcpdump -vvv

# Display packets in ASCII
tcpdump -A

# Display packets in HEX and ASCII
tcpdump -XX
```
