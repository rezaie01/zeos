
## Was ich gelernt habe?
1. **`ls -la` (Interpretation)** – Die Fähigkeit, die Ausgabe von `ls -la` zu lesen, ist für mich wichtig, um nach der Installation von Shadow die korrekten Berechtigungen von Binärdateien wie `/usr/bin/passwd` zu prüfen.  
2. **System- & Dienstverwaltung (`ps`, `kill`)** – Ich installiere Procps-ng und Psmisc, was mir die Werkzeuge gibt, um Prozesse zu überwachen (`ps`) und zu steuern (`killall`). Ich verstehe die Prozessverwaltung auf Betriebssystemebene.  
3. **Log-Verwaltung (`syslogd`, `/var/log`)** – Ich installiere Sysklogd und erstelle die notwendigen Log-Verzeichnisse. Ich kenne die Architektur der Systemprotokollierung und deren Bedeutung für die Fehlerverfolgung.  
4. **`df` / `du` (Speicherplatz anzeigen)** – Da ich `df` und `du` als Teil der Coreutils selbst gebaut habe, kann ich sie zur Kapazitätsplanung und Überwachung der Systemressourcen einsetzen.  
5. **Netzwerkschnittstellen & Routing (`ip`)** – Das `ip`-Kommando (IPRoute2) ist für die moderne Netzwerkkonfiguration unerlässlich. Ich habe gelernt, Schnittstellen- und Routing-Informationen ohne veraltete Tools zu verwalten.  
6. **Berechtigungen (`chmod`) & `ls -l`)** – Im Zuge der Shadow-Installation ändere ich die Berechtigungen von sensiblen Dateien, was mir zeigt, wie SUID/SGID-Bits die Sicherheit des Systems steuern.  
7. **Kern-Dateimanipulation (`cp`/`mv`/`rm`)** – Die Bereinigung des Systems am Ende von Kapitel 8 (z. B. das Entfernen von `.la`-Dateien und der temporären Toolchain) erfordert den sicheren Umgang mit diesen kritischen Befehlen.