---
layout: post
title: Linux Command Notes
date: 2018-08-28 22:43 +0800
tags:
  - SysAdmin
---

Linux command notes.

{% include toc.html %}

### `chattr`

```shell
chattr +i file # read only
chattr +a file # append only
```

### `tar`

```shell
# -c: create, -x: extract, -v: verbose, -f: the file
tar -cvf etc.tar etc
tar -xvf etc.tar

# List: -t,--list 
tar -tvf etc.tar

# Append: -r,--append
tar -rvf etc.tar file.txt

# Exclude: --exclude=<file>
tar -cvf etc.tar --exclude=file1.txt --exclude=file2.txt 

# Gzip
tar -zcvf etc.tar.gz etc
tar -zxvf etc.tar.gz

# Xz
tar -Jcvf etc.tar.xz etc
tar -Jxvf etc.tar.xz

# Bzip2
tar -jcvf etc.tar.bz2 etc
tar -jxvf etc.tar.bz2

# Too many files
ls > ../files_to_copy.txt
tar -T ../files_to_copy.txt -czf ../tarfile.tgz

# Tarcopy: preserves timestamps and file permissions, and faster
time tar -cf - file* | (cd ../files_copied/ && tar -xf -)

# Backing up a system
cd /
tar zcf backup.tgz --exclude=backup.tgz \
    --exclude=/dev --exclude=/run --exclude=/sys \
    --exclude=/tmp --exclude=/proc --exclude=/lost+found \
    --exclude=/mnt --exclude=/media --exclude=/cdrom --exclude=/snap \
    /
tar zcf backup.tgz --exclude={/backup.tgz,/dev,/run,/sys,/tmp,/proc,/lost+found,/mnt,/media,/cdrom,/snap} /
tar zxf backup.tgz --recursive-unlink # recovering (unlink new files as well)
```

### `ps`

```shell
# To see every (-e) process on the system using standard syntax
ps -eTf   # -T,-m,-L: include threads, -f: more info, -F: even more info
ps -ely   # -l: long format, -y: no flags, rss
ps -ejH   # -j: BSD format, -H: process tree

# To see every process on the system using BSD syntax
ps axu    # a: all, x: non tty also, u: with user info
ps af     # f: process tree
ps axjf   # j: BSD job control format
ps axH    # H: show threads
ps axms   # m: show threads, s: signal format

# To get security info:
ps -eo euser,ruser,suser,fuser,f,comm,label
ps axZ
ps -eM

# To see every process running as root (real & effective ID) in user format:
ps -U root -u root u

# To see every process with a user-defined format:
ps -eo pid,tid,class,rtprio,ni,pri,psr,pcpu,stat,wchan:14,comm
ps axo stat,euid,ruid,tty,tpgid,sess,pgrp,ppid,pid,pcpu,comm
ps -Ao pid,tt,user,fname,tmout,f,wchan

ps -C nginx # command name
```

### `runlevel`

```shell
# Show current run level
runlevel

# Changing run level:
#   0 = halted
#   1 = single user (maintenance mode)
#   2 = multi-user mode
#   3-5 = same as 2
#   6 = reboot
init 0 # halt

# See: /etc/init
```

### `sleep`

```shell
sleep 0.1
sleep 1
sleep 1h # s, m, h, d
sleep infinity
```

### `grep`

```shell
egrep # = `grep -e`, regex
grep -i # icase
grep -v # exclude
grep -o # only
grep -r $pattern $dir # search $pattern from files in $dir

apt install ack-grep
brew install ack
```

### `sed`

```shell
cat -n /etc/passwd # with line numbers
cat -n /etc/passwd | sed '100,$c\
fuck
' # replace line 100~ with fuck
cat -n /etc/passwd | sed '100c\
fuck
' # replace line 100 with fuck
cat -n /etc/passwd | sed '/*/c\
fuck
' # replace lines containing '*' with fuck, not regex
cat -n /etc/passwd | sed '100i\
fuck
' # insert fuck before line 100
cat -n /etc/passwd | sed '100a\
fuck
' # append fuck after line 100
cat -n /etc/passwd | sed '100d' # delete line 100
cat -n /etc/passwd | sed '100,105d' # delete line 100~105
cat -n /etc/passwd | sed -n # slient; nothing
cat -n /etc/passwd | sed -n 100p # only print line 100
sed -i 's/#.*$//g' code.py # inplace greedy substitution (gnu sed)
cat -n /etc/passwd | sed '70,100s/[a-zA-Z0-9]/ /g' # only substitute in line 70~100
cat -n /etc/passwd | sed -f script.sed # operate with sed script
```

### `tr`

```shell
echo abcde | tr abc ABC # ABCde
echo abcde | tr abc AB # ABBde
echo abcde | tr -d ace # bd
echo aaabbbcccddd | tr -s ac # abbbcddd

# Caesar cipher
echo 7h15 15 p1ain t3xt \
  | tr abcdefghijklmnopqrstuvwxyz xyzabcdefghijklmnopqrstuvw \
  | tr ABCDEFGHIJKLMNOPQRSTUVWXYZ XYZABCDEFGHIJKLMNOPQRSTUVW \
  | tr 0123456789 7890123456
# 4e82 82 m8xfk q0uq
```

### `awk`

https://www.shortcutfoo.com/app/dojos/awk/cheatsheet

```shell
# Print the first word of a line
ps ax | awk '{print $1}'
ps ax | tr -s ' \t' ' ' | cut ' ' -f1
ps ax | sed -e 's/\s.*$//'
```

### `cut`

```shell
cut -d: -f1 /etc/passwd # print field 1 with : as the delimiter
cut -d: -f1,3,5 /etc/passwd # field 1, 3, and 5, containing delimiters
cut -d: -f1,3,5 --output-delimiter ' ' /etc/passwd # replace delimiters

cut -d: -f3- /etc/passwd # fields after 3
cut -d: -f-3 /etc/passwd # fields before 3

cut -b-2 /etc/passwd # first 2 bytes
cut -c-2 /etc/passwd # first 2 chars
```

### `yes`

```shell
yes | COMMAND
yes yes | COMMAND
ncat -lk 80 -c 'yes fuck'
```

### `xargs`

Usage

```shell
# Remove all docker containers
docker -aq | xargs docker rm -f

# Kill all processes that match something
ps ax | grep backdoor | cut -d ' ' -f1 | xargs kill -9
```

Reverse of `xargs`

```shell
rxargs() {
  local array
  read -a array
  printf '%s\n' ${array[@]}
}

alias rxargs="tr -s ' ' '\n'"

# With prefix and postfix
rxargs() {
  local array
  read -a array
  for item in ${array[*]}; do
    echo -e "$1""$item""$2"
  done
}
```

### `find`

```shell
find /etc -name hosts # ignore case
find /etc -iname HOSTS # ignore case

find -O2 /etc -name hosts # optimization

find ./ -type d # find directory (f: file, d: directory, l: symlink)
find ./ -executable
find ./ -regex '.+\.pyc'

# time (day), min (minutes)
# a (access), m (modify content), c (status change)
find ./ -mmin -20 # find things changed 20 min ago
find ./ -daystart -mtime -1 # find things changed today (count from the end of today)
find ./ -mtime +31 # find things older than 31 days

find ./ -printf "%y %i %prn\n" # printing type of item, inode, and filename

# Deleting
find ./ -type f -name 'garbage' -exec rm {} ;
find ./ -type f -name 'garbage' -delete
find ./ -type f -name 'garbage' | xargs rm
find ./ -type f -name '*.pyc' -exec rm '{}' + -print

find ./ -user USER
find ./ -group GROUP
```

### `sort`, `uniq`

```shell
# Always uniq after sort
last | cut -d ' ' -f1 | grep -v '^$' | sort | uniq -c # count

# Unique lines
last | cut -d ' ' -f1 | grep -v '^$' | sort -u
```

### `printf`

```shell
# Repeat words
printf 'Penis%.0s' {1..5} # PenisPenisPenisPenisPenis

# Generate specific length of string
printf "%0${n}s"
```

### `curl`

```shell
curl -XPOST -d passwd=PW URL
curl -XPOST -F file=@/etc/shells -F name=NAME URL
curl -XDELETE -H 'Origin: http://dick.in/the_box/' URL
curl -s URL
curl --path-as-is --globoff URL
```

### `wget`

```shell
# Same as `curl -s http://example.com/`
wget -qO- http://example.com/

# Set download file name
wget URL -O saved_file.txt

# Download from URL list
wget -i url_list.txt -P save_dir # P stands for Prefix
cat url_list.txt | wget -i - -P save_dir # download from stdin list

# Download recursively
# https://stackoverflow.com/questions/273743/using-wget-to-recursively-fetch-a-directory-with-arbitrary-files-in-it
wget --recursive --no-parent URL # download recursively

# Debug
wget -d URL

# Cookie
wget --cookies=on --keep-session-cookies --save-cookies=cookie.txt URL
wget --load-cookie cookie.txt URL

# Headers
wget --referer='http://example.com/' URL

# HTTP POST
wget --post-data='username=djosix&password=djosix' URL

```

### Package Managers



#### `dpkg`

```shell
dpkg -i PKG.deb     # install
dpkg -l | grep PKG  # list
dpkg -r PKG         # remove
dpkg -P PKG         # purge
```

#### `pacman`

```shell
pacman -S PKG   # install
pacman -Ss PKG  # search
pacman -Sy      # update
pacman -Su PKG  # upgrade

pacman -Rsc PKG # remove package (s: recursive, c: cascade)

pacman -Q       # list installed
pacman -Qe      # list explicitly installed
pacman -Ql PKG  # list files
pacman -Qii PKG # info
pacman -Qs PKG  # search installed

pactree PKG     # dependencies of PKG
pactree -r PKG  # reverse dependencies
```
