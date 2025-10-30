#linux #snapshot #datensicherung #filesystem #dd #fsfreeze #sysadmin
# Einen Snapshot von einer Partition erstellen

## mit `dd`- Kopiert die ganze Partition

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


## mit `fsarchiver`: 

### Snapshot erstellen
1. **Partition vorbereiten** (falls gemountet):
```bash
sudo fsfreeze -f /mountpoint  # Für Konsistenz bei RW-Mount
```

2. **Archiv erstellen**:
```bash
sudo fsarchiver savefs -v -z 3 /pfad/zur/sicherung.fsa /dev/sdX1
```
- `-v`: Verbose Ausgabe
- `-z 3`: Komprimierung (1=schnell, 9=stark)
- Letzter Parameter: Zielarchiv und Quellpartition

3. **Partition freigeben** (falls eingefroren):
```bash
sudo fsfreeze -u /mountpoint
```

### Partition zurücksetzen
1. **Live-System booten** (wenn Systempartition betroffen)
2. **Zielpartition unmounten**:
```bash
sudo umount /dev/sdY1
```
3. **Archiv restaurieren**:
```bash
sudo fsarchiver restfs -v /pfad/zur/sicherung.fsa id=0,dest=/dev/sdY1
```
- `id=0`: Erste Partition im Archiv
- `dest`: Zielpartition

### Wichtige Hinweise
- ✅ Unterstützt ext2/3/4, btrfs, xfs, reiserfs
- 🔧 Bei NTFS/APFS: Andere Tools verwenden
- 📦 Spart Platz durch Komprimierung (vs. dd)
- 🚨 Zielpartition muss mindestens gleich groß sein wie Original
- 🔍 Integrität prüfen mit:
```bash
fsarchiver archinfo /pfad/zur/sicherung.fsa
```

> **<font color="#ffc000">Pro-Tipp:</font>**  
> *Für Multicore-Systeme `-j 4` hinzufügen (parallelisierte Kompression). Bei inkrementellen Backups `-A @timestamp-file` verwenden.*