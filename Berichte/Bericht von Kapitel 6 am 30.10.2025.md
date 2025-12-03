1. Erfolgreiches Compilierung mit logs versichert:
- Die logs sind: ![[logs-von-kapitel-6.png]]
--- 
2. Snapshots gemacht: 
	```bash
	sudo umount $LFS
	
	sudo fsarchiver savefs /files/LFS/snapshots/chp06.fsa /dev/nvme0n1p7
	```


***

## Was ich gelernt habe?

1. find (Erweiterte Suche) – Ich verwende find und xargs, um unerwünschte .la-Dateien (Libtool-Archive) und Build-Dateien aus der temporären Toolchain zu entfernen. Das ist eine entscheidende Wartungsaufgabe zur Bereinigung von Build-Artefakten.
2. Kommando-Pipelines & xargs – Durch die Verknüpfung von find und xargs demonstriere ich meine Fähigkeit, komplexe Batch-Operationen auszuführen, was die Effizienz in der Systemverwaltung stark erhöht.
3. Dateiinhalt anzeigen (cat/less/tail -f) – Während des Glibc-Builds muss ich die Log-Dateien überprüfen, um sicherzustellen, dass die Compiler- und Linker-Pfade korrekt auf $LFS/tools verweisen. Die Analyse von Log-Dateien mit Tools wie less ist entscheidend für die Fehlerdiagnose.