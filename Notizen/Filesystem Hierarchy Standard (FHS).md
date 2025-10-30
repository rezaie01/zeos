# Filesystem Hierarchy Standard (FHS)

## Kernverzeichnisse

| Verzeichnis | Funktion | Wichtige Inhalte |
|-------------|----------|------------------|
| **/**       | Wurzelverzeichnis | Basis aller Verzeichnisse |
| **/bin**    | Essenzielle Benutzerbefehle | `ls`, `cp`, `cat`, `mv` |
| **/sbin**   | Systemadmin-Tools | `fdisk`, `iptables`, `reboot` |
| **/usr**    | Statische Anwendungsdaten | Binärdateien (`/usr/bin`), Libraries (`/usr/lib`), Dokumentation |
| **/etc**    | Systemweite Konfiguration | `/etc/passwd`, Netzwerkeinstellungen, Dienstekonfigs |
| **/var**    | Variable Daten | Logs (`/var/log`), Datenbanken (`/var/lib`), Mailqueue |
| **/home**   | Benutzerverzeichnisse | Persönliche Dateien pro User |
| **/root**   | Home des Root-Users | Systemadministrator-Dateien |
| **/tmp**    | Temporäre Dateien | Wird beim Booten geleert |
| **/dev**    | Gerätedateien | `/dev/sda` (Festplatten), `/dev/tty` (Terminals) |
| **/proc**   | Virtuelles Prozess-Info-System | Echtzeit-Systemdaten als Dateien |
| **/boot**   | Boot-Dateien | Kernel, Initramfs, Bootloader |

## Wichtige Prinzipien
- **Strikte Trennung** zwischen:
  - Systemkritischen (`/bin`, `/sbin`) und nicht-kritischen Dateien (`/usr`)
  - Statischen Konfigs (`/etc`) und dynamischen Daten (`/var`)
- **/usr ist read-only** bei normalem Betrieb
- **Konsistenz** über alle Linux-Distributionen hinweg
- **/opt** für proprietäre Software (z.B. Google Chrome)
- **/srv** für dienstespezifische Daten (Webserver, FTP)
