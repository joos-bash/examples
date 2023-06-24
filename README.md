# examples
Examples

arch_tools.sh
```bash
#!/bin/bash

Z="compress"; unZ="uncompress" ; Zlist=""
gz="gzip" ; ungz="gunzip" ; gzlist=""
bz="bzip2" ; unbz="bunzip2" ; bzlist=""

for arg
do
	if [ -f "$arg" ] ; then
		case "$arg" in
		*.Z) $unZ "$arg"
			arg="$(echo $arg | sed 's/\.Z$//')"
			Zlist="$Zlist \"$arg\""
			;;
		*.gz) $ungz "$arg"
			arg="$(echo $arg | sed 's/\.gz$//')"
			gzlist="$gzlist \"$arg\""
			;;
		*.bz2) $unbz "$arg"
			arg="$(echo $arg | sed 's/\.bz2$//')"
#			echo "arg:" $arg
			bzlist="$bzlist \"$arg\""
			echo "bzlist" $bzlist
			;;

		esac
	fi
newargs="${newargs:-""} \"$arg\""
#echo "newargs:" $newargs
done

case $0 in
	*zcat* ) eval cat $newargs ;;
	*zmore* ) eval more $newargs ;;
	*zgrep* ) eval grep $newargs ;;
	* ) echo "$0: unknown options" >&2
	exit 1
esac

if [ ! -z "$Zlist" ] ; then
	eval $Z $Zlist
fi

if [ ! -z "$gzlist" ] ; then
	eval $gz $gzlist
fi

if [ ! -z "$bzlist" ] ; then
	eval $bz $bzlist
fi
```

arping.sh
```bash
#!/bin/bash
PREFIX=$1
INTERFACE=$2

trap 'echo "Ping exit (Ctrl-C)"; exit 1' 2

for SUBNET in {0..255}
do
	for HOST in {1..255}
	do
		echo "[*] IP : "$PREFIX"."$SUBNET"."$HOST
		arping -c 3 -i $INTERFACE $PREFIX"."$SUBNET"."$HOST 2> /dev/null
	done
done
```

create_ram.sh
```bash
#!/bin/bash
ROOTUSER_NAME=root
MOUNTPT=/mnt/ramdisk
SIZE=2000
BLOCKSIZE=1024
DEVICE=/dev/ram0

username=`id -nu`
if [ "$username" != "$ROOTUSER_NAME" ]
then
	echo "Must be root to run \"`basename $0`\"."
	exit 1
fi

if [ ! -d "$MOUNTPT" ]
then
	mkdir $MOUNTPT
fi

dd if=/dev/zero of=$DEVICE count=$SIZE bs=$BLOCKSIZE

mke2fs $DEVICE
mount $DEVICE $MOUNTPT
chmod 777 $MOUNTPT

echo "\"$MOUNTPT\" is ready."
exit 0
```

kill_all.sh
```bash
#!/bin/bash

if test -z "$1"
then
	echo "Usage: `basename $0` Process(es)_to_kill"
	exit 1
fi

PROCESS_NAME="$1"
ps ax | grep "$PROCESS_NAME" | awk '{print $1}' | xargs -i kill {} 2&>/dev/null

exit $?
```

pass_gen.sh
```bash
#!/bin/bash

MATRIX="0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"
LENGTH="10"

while [ "${n:=1}" -le "$LENGTH" ]
do
	PASS="$PASS${MATRIX:$((RANDOM%${#MATRIX})):1}"
	(( n+=1 ))
done

echo "$PASS"
exit 0
```

web_scrap.sh
```bash
#!/bin/bash

if [ $# -eq 0 ] ; then
	echo "Usage: $0 [-d|-i|-x] url" >&2
	echo "-d=domains only, -i=internal refs only, -x=external only" >&2
	exit 1
fi

if [ $# -gt 1 ] ; then
	case "$1" in
	-d) lastcmd="cut -d/ -f3|sort|uniq"
		shift
		;;
	-i) basedomain="$(echo "$2" | cut -d/ -f3)/"
		lastcmd="grep -E \"^https?://$basedomain\"|sed -E \"s|https?://$basedomain||g\"|sort|uniq"
		shift
		;;
	-x) basedomain="$(echo "$2" | cut -d/ -f3)/"
		lastcmd="grep -Ev \"^https?://$basedomain\"|sort|uniq"
		shift
		;;
	*) echo "$0: unknown option specified: $1" >&2
		exit 1
	esac
else
	lastcmd="sort|uniq"
fi

lynx -dump "$1"|\
	sed -n '/^References$/,$p'|\
	grep -E '[[:digit:]]+\.'|\
	awk '{print $2}'|\
	cut -d\? -f1|\
	eval $lastcmd
```
