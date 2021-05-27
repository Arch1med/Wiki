# Contexte 

Ce script provient du livre "_Penetration Testing: A Hands-On Introduction to Hacking_" écrit pas Georgia Weidman.

C'est un script shell qui permet de "scanner" un réseau à l'aide de ping.

Usage (pour scanner le réseau 192.168.0.0/24): 

./ping.sh 192.168.0

# Script 

```sh
#!/bin/bash
if [ "$1" == "" ]
then
	echo "Usage: ./pingscript.sh [network]"
	echo "example: ./pingscript.sh 192.168.0"
else
	for x in $(seq 1 254); do
		ping -W 100 -c 1 $1.$x | grep "64 bytes" | cut -d" " -f4 | sed 's/.$//'
	done
fi
```
