
## Was ich gelernt habe?
1. **Skript-Steuerfluss (Args/If)** – Ich analysiere die LFS-Bootscripts, die mit `if` und `case` arbeiten, um Dienste basierend auf Runlevels korrekt zu starten und Dateisystemprüfungen durchzuführen (`checkfs`). Ich verstehe die Ablauflogik des Systemstarts.  
2. **Schleifen (`for`/`while`)** – Ich verwende Skript-Schleifen, um die Locales zu installieren oder um bei der Netzwerkkonfiguration alle Schnittstellen abzufragen. Dies ist ein Zeichen für fortgeschrittene Automatisierungsfähigkeiten.  
3. **Kernel-Modulverwaltung (`lsmod`/`modprobe`)** – Ich erstelle die Konfigurationsdatei `/etc/modprobe.d/usb.conf`, um die Ladereihenfolge der USB-Treiber zu fixieren. Ich habe gelernt, Modulkonflikte aktiv auf Kernel-Ebene zu beheben.  
4. **LVM-Verwaltung (PV/VG/LV)** – Die LFS-Bootscripts beinhalten `mdadm`- und `vgchange`-Befehle, was impliziert, dass ich die Integration fortgeschrittener Speichersysteme wie RAID oder LVM in den Boot-Prozess verstehen muss.

