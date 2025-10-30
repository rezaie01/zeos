1. Erfolgreiches Compilierung mit logs versichert:
- Die logs sind: ![[logs-von-kapitel-6.png]]
--- 
2. Snapshots gemacht: 
	```bash
	sudo umount $LFS
	
	sudo fsarchiver savefs /files/LFS/snapshots/chp06.fsa /dev/nvme0n1p7
	```
