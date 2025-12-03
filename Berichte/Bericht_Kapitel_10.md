
## Was ich gelernt habe?

1. **Dateisystemeinbindung & `/etc/fstab` (Erstellung)** – Ich erstelle die finale `/etc/fstab` und definiere darin die Mount-Optionen (`nosuid,noexec,nodev`) für alle Dateisysteme, einschließlich der virtuellen, was die Systemintegrität gewährleistet.  
2. **Boot-Prozess (BIOS/UEFI, GRUB2)** – Ich installiere GRUB und schreibe die Boot-Konfigurationsdatei `grub.cfg` manuell. Dies demonstriert mein tiefes Verständnis der gesamten Boot-Kette, von der Firmware bis zum Kernel-Start.  
3. **Kernel-Boot-Parameter (GRUB)** – Ich trage den Kernel-Pfad und die `root=`-Parameter direkt in `grub.cfg` ein. Ich kann Boot-Probleme durch die Anpassung der Kernel-Startbefehle beheben.  
4. **Kern-Dateimanipulation (`cp`/`mv`/`rm`)** – Ich kopiere das kompilierte Kernel-Image (`vmlinuz-6.16.1-lfs-12.4`) und die `System.map` in das `/boot`-Verzeichnis, was den Abschluss der Kernel-Installation darstellt.