---
layout: post
title: Linux Advanced Notes
date: 2018-08-28 23:02 +0800
tags:
  - SysAdmin
---

Get more deep into Linux administration.

{% include toc.html %}


## Network Hacks

Routing

```shell
# Get default gateway
route -n get default # macOS
netstat -nr
```

ARP

```shell
# Writing ARP table (apt install -y net-tools)
arp -s 10.0.0.1 00:8b:8a:4c:25:1f

# ARP Spoofing (apt install -y dsniff ssldump)
arpspoof -i wls1 -t 192.168.0.109 192.168.0.1
# 0:22:fa:7a:3e:98 8c:f5:a3:81:44:0 0806 42: arp reply 192.168.0.1 is-at 0:22:fa:7a:3e:98
```

MAC Address Spoofing

```shell
# Generate a fake one
MAC=`openssl rand -hex 6 | sed 's/\(..\)/\1:/g; s/.$//'`

# Set the fake MAC address to an interface
sudo ifconfig en0 ether $MAC
```

Scan hosts in the network

```shell
BST=192.168.0.255
ping -c3 $BST | grep 'bytes from' | awk '{print $4}' | sed 's/://g' | sort -u > /tmp/hosts

nmap -iL /tmp/hosts
```

Network Interfaces

```shell
cat /proc/net/fib_trie  # interfaces and ip address
cat /proc/net/route     # routing

# And a lot of things in /proc/net
# --------------------------------
# anycast6      icmp6          ip_tables_matches  packet     rt_cache      udp
# arp           if_inet6       ip_tables_names    protocols  snmp          udp6
# connector     igmp           ip_tables_targets  psched     snmp6         udplite
# dev           igmp6          ipv6_route         ptype      sockstat      udplite6
# dev_mcast     ip6_flowlabel  mcfilter           raw        sockstat6     unix
# dev_snmp6/    ip6_mr_cache   mcfilter6          raw6       softnet_stat  wireless
# fib_trie      ip6_mr_vif     netfilter/         route      stat/         xfrm_stat
# fib_triestat  ip_mr_cache    netlink            rt6_stats  tcp
# icmp          ip_mr_vif      netstat            rt_acct    tcp6

```

## What if there is only `sh`

Using `sh` without any other command

```shell
# cat < file
while read s; do echo "$s"; done < file

# cat > file
while read s; do echo "$s"; done > file

# ls -a
echo * .*
```


