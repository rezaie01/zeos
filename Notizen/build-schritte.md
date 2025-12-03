# Build-Schritte
### Vorbereitung
1. [[host-system]] vorbereiten
2. [[Partitionierung erstellen]]

3. [[Packages & Patches herunterladen]] 
4. [finale Vorbereitungen](finale-vorbereitungen.md)

# Bauen von Temporäre System
🧩 Die temporäre Phase erzeugt eine in sich geschlossene Mini-Toolchain, die das endgültige System aufbauen kann, ohne auf das Hostsystem angewiesen zu sein.
## [[Wichtig grundlegende Matriele]]
1. [[Phase 1 - Kapitel 5  Ein Cross-Toolchain compilieren]]
2. [[Phase 2 - Kapitel 6 Temporäre Tools compilieren]]
3. [[Phase 3 - Kapitel 7 - Chroot betreten und zusätzliche temporäre Tools erstellen]]

***
# Bauen von echte System
1. [[Bauen von LFS System]]
2. [[Konfigurieren vom System]]
3. [[Das System bootabel machen]]
4. Das Ende: (Was ich machen kann!)
	- **System härten und Basisfunktionen testen**  
	    Prüfen, ob Benutzer angelegt werden können, ob Netzwerk, Login, Shell, `ps`, `top`, `mount`, `dmesg` usw. funktionieren.
     - **Ein Paketmanagement-Konzept einführen**  
	    Z. B. eigene Build-Skripte, ein einfaches Logging-System oder ein minimalistisches Paketformat.
     - **Software aus BLFS installieren**  
	    Etwa: Editor (vim/nano), Browser, SSH-Server/Client, Grafikstack (Xorg/Wayland), Desktop-Umgebung.
     - **System für den Alltag oder Experimente erweitern**  
	    Kernel upgraden, Dienste wie `systemd` oder `busybox` ausprobieren, Benchmarks machen oder ein eigenes Mini-Distro-Layout bauen.	