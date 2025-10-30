1. Erfolgreiches Compilierung mit logs versichert:
	- Eigene Helfer-Skript zum Loggen Namens `LFS-logger`:
		```bash
		#!/usr/bin/sh  
		outdir=/files/LFS  
		name="$1"  
		  
		tee $outdir/"${name}.log"
		```


	- Nutzung: 
		```bash
		./configure .... | LFS-logger 5.2.binutils.configure
		```
- Die logs sind: ![[logs-von-kapitel-5.png]]
--- 
2. Snapshots gemacht: 
- wie: 
	```bash
	sudo umount $LFS
	
	sudo fsarchiver savefs /files/LFS/snapshots/chp05.fsa /dev/nvme0n1p7
	```
- Snapshot: 
	![[snapshots.png]]
