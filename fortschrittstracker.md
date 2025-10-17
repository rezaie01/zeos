# Linux From Scratch – Bauplan (LFS 12.2 Systemd)
**Zeitraum:** 15. Oktober – 29. Oktober 2025  
**Ziel:** Vollständig selbst gebautes LFS-System in einer virtuellen Maschine  
**Durchschnitt:** 6 Stunden pro Tag  

---

## 🧩 Teil 1: Kapitel 2.4 bis 3.1  
### Ziel: Partitionierung und Vorbereitung des Host-Systems

#### 16.10.2025:
- [x] Lies Kapitel 2.4–2.7 sorgfältig, um den Partitionsaufbau zu verstehen.  
- [x] Entscheide über Partitionsgröße (EFI, /boot, swap, /).  
- [x] Erstelle die Partitionen über das Terminal.  
- [x] Formatiere jede Partition in den empfohlenen Dateisystemen.  
- [x] Aktiviere und überprüfe Swap.  
- [x] Mounte das Root-Dateisystem unter `/mnt/lfs`.  
- [x] Überprüfe Host-System-Anforderungen (Kapitel 3.1).  
- [x] 📸 **Snapshot 1:** „Nach Vorbereitung Host + Partitionierung“  
#### 17.10.2025:
- [ ] Stelle sicher, dass alle benötigten Tools installiert sind (GCC, Bash, Make usw.).  
- [ ] Lege ein LFS-Verzeichnis auf der Host-Platte an.  
- [ ] Dokumentiere dein System-Setup (Kernel-Version, Distro, VM-RAM, Speicherplatz).  
- [ ] 📸 **Snapshot 1:** „Nach Vorbereitung Host + Partitionierung“  

---

## ⚒️ Tag 2 – Kapitel 4 (Vorbereitung auf Toolchain)
### Ziel: LFS-User und Verzeichnisstruktur anlegen

- [ ] Erstelle den Benutzer `lfs` und die passende Gruppenstruktur.  
- [ ] Erstelle die Verzeichnisstruktur (`/mnt/lfs/tools`, `/mnt/lfs/sources`).  
- [ ] Setze die richtigen Berechtigungen.  
- [ ] Lade alle Quellpakete und Patches aus Kapitel 4.  
- [ ] Prüfe die Integrität der Downloads (md5sum).  
- [ ] Verknüpfe das `/tools`-Verzeichnis symbolisch, falls nötig.  
- [ ] Richte die Umgebungsvariablen für den Benutzer `lfs` ein.  
- [ ] 📸 **Snapshot 2:** „LFS-Benutzer eingerichtet und Struktur erstellt“  

---

## 🧱 Tag 3 – Kapitel 5.2 bis 5.9 (Toolchain Phase 1)
### Ziel: Erste Cross-Compiler-Toolchain erstellen

- [ ] Lies die Einführung zu Kapitel 5 (Ziel der temporären Toolchain).  
- [ ] Baue und installiere Binutils (Pass 1).  
- [ ] Baue und installiere GCC (Pass 1).  
- [ ] Installiere Linux API Headers.  
- [ ] Baue und installiere Glibc.  
- [ ] Kompiliere Binutils (Pass 2).  
- [ ] Kompiliere GCC (Pass 2).  
- [ ] Teste, ob der Compiler richtig funktioniert.  
- [ ] 📸 **Snapshot 3:** „Toolchain Phase 1 fertig“  

---

## ⚙️ Tag 4 – Kapitel 6.1 bis 6.6 (Toolchain Phase 2)
### Ziel: Temporäre Werkzeuge in der isolierten Umgebung

- [ ] Lies Kapitel 6 sorgfältig (zweite Phase).  
- [ ] Erstelle das notwendige Verzeichnislayout für die neue Umgebung.  
- [ ] Installiere die grundlegenden Pakete (M4, Ncurses, Bash, Coreutils usw.).  
- [ ] Prüfe nach jedem Paket die Tests (optional, wenn Zeit).  
- [ ] Überprüfe Pfade und Symbole (keine Host-Abhängigkeiten).  
- [ ] 📸 **Snapshot 4:** „Temporäre Werkzeuge abgeschlossen“  

---

## 🧍‍♂️ Tag 5 – Kapitel 7 (Chroot Environment)
### Ziel: In das neue System wechseln

- [ ] Lies genau, was beim Wechsel ins `chroot` geschieht.  
- [ ] Mounte virtuelle Dateisysteme (`/dev`, `/proc`, `/sys`, `/run`).  
- [ ] Wechsel in das neue System mit `chroot`.  
- [ ] Setze neue Umgebungsvariablen.  
- [ ] Überprüfe, ob `/tools` erreichbar ist.  
- [ ] Teste einfache Kommandos (ls, echo, pwd) innerhalb von chroot.  
- [ ] 📸 **Snapshot 5:** „System erfolgreich im chroot“  

---

## 🧩 Tag 6 – Kapitel 8.1 bis 8.10  
### Ziel: Aufbau des Basissystems – erste Systempakete

- [ ] Lies den Überblick von Kapitel 8.  
- [ ] Installiere grundlegende Bibliotheken und Tools (z. B. Man-Pages, Findutils, Diffutils).  
- [ ] Prüfe nach jedem Abschnitt die LFS-Anweisungen sorgfältig.  
- [ ] Dokumentiere jedes installierte Paket mit kurzer Notiz (Name, Zweck).  
- [ ] Überprüfe die neuen Binärdateien in `/usr/bin` und `/usr/lib`.  
- [ ] 📸 **Snapshot 6:** „Basissystem Teil 1 abgeschlossen“  

---

## ⚙️ Tag 7 – Kapitel 8.11 bis 8.25  
### Ziel: Komplette Systemtools installieren

- [ ] Setze die Arbeit im chroot fort.  
- [ ] Installiere Werkzeuge wie Grep, Sed, Gawk, Bash, Coreutils, Tar.  
- [ ] Nach jedem Hauptpaket: kurze Funktionsprüfung.  
- [ ] Notiere Fehler oder Warnungen in Logdateien.  
- [ ] Erstelle `/etc/passwd` und `/etc/group`.  
- [ ] 📸 **Snapshot 7:** „Basissystem vollständig installiert“  

---

## ⚙️ Tag 8 – Kapitel 8.26 bis 8.41  
### Ziel: Abschluss der Systemsoftware und Tests

- [ ] Installiere temporäre Bibliotheken, Header und systemrelevante Tools.  
- [ ] Überprüfe Library-Links mit `ldd` (Lernschritt).  
- [ ] Stelle sicher, dass Systemdienste bereit sind.  
- [ ] Teste Bash und Glibc-Funktionalität.  
- [ ] 📸 **Snapshot 8:** „Systemtools fertig – bereit für Konfiguration“  

---

## 🪪 Tag 9 – Kapitel 9.1 bis 9.6 (Systemkonfiguration)
### Ziel: Systemparameter und Grundkonfiguration

- [ ] Setze Zeitzone und Locale.  
- [ ] Richte `/etc/fstab` korrekt ein.  
- [ ] Konfiguriere Netzwerkskripte (`/etc/systemd/network` oder `ifconfig.eth0`).  
- [ ] Erstelle Hostname und Hosts-Dateien.  
- [ ] Stelle sicher, dass Systemd-Dienste aktivierbar sind.  
- [ ] 📸 **Snapshot 9:** „Basiskonfiguration abgeschlossen“  

---

## ⚙️ Tag 10 – Kapitel 9.7 bis 9.10 (Kernel & Bootloader)
### Ziel: Kernel konfigurieren und GRUB installieren

- [ ] Lies Abschnitt 9.7 aufmerksam (Kernelkonfiguration).  
- [ ] Führe eine vollständige Kernelkonfiguration durch.  
- [ ] Baue den Kernel manuell und überprüfe, ob Module korrekt installiert sind.  
- [ ] Installiere GRUB als Bootloader.  
- [ ] Erstelle die Datei `grub.cfg` manuell.  
- [ ] 📸 **Snapshot 10:** „Kernel und Bootloader bereit“  

---

## 💻 Tag 11 – Kapitel 10.1 bis 10.3
### Ziel: Abschlussarbeiten und erste Boot-Vorbereitung

- [ ] Entferne temporäre Dateien aus `/tools`.  
- [ ] Überprüfe Dateistruktur mit der Checkliste im Buch.  
- [ ] Kontrolliere `/etc/passwd`, `/etc/group`, `/etc/fstab`, `/boot`.  
- [ ] Bereite das System auf den Reboot vor (Unmounts etc.).  
- [ ] 📸 **Snapshot 11:** „Kurz vor erstem Boot“  

---

## 🚀 Tag 12 – Erster Bootversuch
### Ziel: Eigenes LFS-System starten

- [ ] Reboote die virtuelle Maschine in das neue System.  
- [ ] Wenn Fehler auftreten, dokumentiere sie.  
- [ ] Teste grundlegende Kommandos (ls, pwd, whoami).  
- [ ] Überprüfe `systemd-analyze` Bootzeit.  
- [ ] Verifiziere Kernel-Version und GRUB-Menü.  
- [ ] 📸 **Snapshot 12:** „Erster Boot erfolgreich“  

---

## 🧠 Tag 13 – Nachbereitung (Technisch)
### Ziel: Funktionalität testen & Logs prüfen

- [ ] Überprüfe alle Systemdienste mit `systemctl list-units`.  
- [ ] Teste Netzwerkschnittstellen und Ping-Konnektivität.  
- [ ] Teste Benutzerwechsel und Berechtigungen.  
- [ ] Prüfe `/var/log` auf Fehlermeldungen.  
- [ ] Dokumentiere eventuelle Fixes oder Workarounds.  
- [ ] 📸 **Snapshot 13:** „System stabil – technische Nachbereitung abgeschlossen“  

---

## 📝 Tag 14 – Dokumentation & GitHub Upload (Teil 1)
### Ziel: Dokumentation strukturieren

- [ ] Sammle alle Build-Notizen und Snapshots.  
- [ ] Erstelle Markdown-Dateien:  
  - `build-log.md` (Fehler, Lösungen, Besonderheiten)  
  - `system-info.md` (Kernel-Version, Tools, Systemzeit, Architektur)  
- [ ] Beginne mit der README.md für dein GitHub-Repository.  
- [ ] Erstelle Ordnerstruktur (`/docs`, `/images`, `/notes`).  
- [ ] 📸 **Snapshot 14:** „Dokumentation vorbereitet“  

---

## 📘 Tag 15 – Dokumentation & GitHub Upload (Teil 2)
### Ziel: Projekt online stellen & Abschluss

- [ ] Fertigstellung der README.md (Ziel, Vorgehen, Lessons Learned).  
- [ ] Screenshots oder Bilder der Bootausgabe einfügen.  
- [ ] Überprüfe Markdown-Formatierung (Überschriften, Listen, Links).  
- [ ] Lade das gesamte Projekt auf GitHub hoch.  
- [ ] Schreibe abschließenden Commit:  
  *“LFS 12.2 completed – documented and booted successfully.”*  
- [ ] 📸 **Final Snapshot:** „Projekt vollständig dokumentiert und veröffentlicht“  

---

## ✅ Abschluss


**Projektende:** Mittwoch, 29. Oktober 2025  
