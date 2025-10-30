#linux #snapshot #datensicherung #filesystem #dd #fsfreeze #sysadmin
# Einen Snapshot von einer Partition erstellen

## Schritt-für-Schritt Anleitung

1. **Dateisystem einfrieren** (für Konsistenz):
```bash
sudo fsfreeze -f /mountpoint
```
- `-f`: Friert das Dateisystem ein
- `/mountpoint`: Mount-Punkt der Partition (z.B. /mnt/data)

2. **Snapshot mit dd erstellen**:
```bash
sudo dd if=/dev/sdX1 of=/pfad/zur/snapshot.img bs=4M status=progress
```
- `if`: Input Device (Quellpartition)
- `of`: Output File (Zieldatei)
- `bs=4M`: Blockgröße für bessere Performance
- `status=progress`: Zeigt Fortschrittsbalken

3. **Dateisystem wieder freigeben**:
```bash
sudo fsfreeze -u /mountpoint
```

## Wichtige Hinweise
- 🔴 Immer Gerätebezeichnung (z.B. /dev/sda1) doppelt prüfen!
- 💾 Ausreichend freien Speicherplatz sicherstellen
- ⏱️ Am besten während geringer Systemlast ausführen
- 🔒 Root-Rechte erforderlich (sudo)
- 🖧 Bei Netzwerk-Filesystems vorher Dokumentation prüfen

> **<font color="#ffc000">Hinweis:</font>** 
> *Die Blockgröße `bs=4M` kann je nach System optimiert werden (typisch 1M-64M). Für bessere Performance bei großen Partitionen `conv=noerror,sync` hinzufügen.*