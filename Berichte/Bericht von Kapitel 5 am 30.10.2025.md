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


***

## Was ich gelernt habe?

1. **Ausführen von Kernbefehlen & Hilfe (make)** – Die mehrstufige Kompilierung von GCC und Binutils lehrt mich, dass ich `make` mit spezifischen Targets (z. B. `make mrproper`, `make install`) und Flags steuern muss. Dies ist Kernwissen für jeden, der Code baut.  
2. **Textverarbeitung (`sed`/`awk`) (Patchen)** – Ich nutze `sed` intensiv, um die Pfade in der Cross-Toolchain zu korrigieren und somit die Abhängigkeit vom Host-System zu brechen. Diese präzise Textmanipulation ist für das Patchen von Quellen unerlässlich.