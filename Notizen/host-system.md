#partitionierung
# Host System

---
## Partitionierung

### Festplattenlayout

|Nr.|Größe|Mountpunkt|Dateisystem|Besonderheiten|
|---|---|---|---|---|
|1|300 MiB - 550 MiB|/boot/efi|FAT32|EFI-Systempartition|
|2|500 MiB|/boot|ext2|0% reservierte Blöcke, kein Journaling|
|3|2 GB|SWAP|swap|Linux-Swap|
|4|20 GB|/|ext4|Root-Partition|

**Begründungen:**

- **/boot mit ext2**: Reduziert Schreibzugriffe durch Verzicht auf Journaling
- **Swap-Größe**: 2 GB als Daumenregel für Systeme mit 4-8 GB RAM

---

## Systemvorbereitung

### 1. Basisinformationen

- **Betriebssystem**: Zorin OS 16
- **Hardware-Übersicht**:  
    ![](app://818e2278647f361570af409fd3fedd8edfbb/C:/Users/green/Desktop/rezos-linux/rezos-linux/Notizen/Pasted%20image%2020251015031513.png?1760490913391)

### 2. Paketinstallation

```bash
sudo apt install \
  binutils bison texinfo \
  build-essential coreutils \
  findutils gawk gcc g++ \
  grep m4 make patch \
  perl python3 sed tar xz-utils
```

---

## Systemkonfiguration

### Umgebungsvariablen

```bash
export LFS=/mnt/lfs
echo "export LFS=/mnt/lfs" >> ~/.bashrc
```

### Dateiberechtigungen

```bash
umask 022
```

**Bedeutung der Umask 022**:

- Dateien: `644` (rw-r--r--)
- Verzeichnisse: `755` (rwxr-xr-x)

---

## Glossar

### Fachbegriffe

- **Journaling**  
    Protokollierung von Dateisystemänderungen zur Crash-Sicherheit
    
- **Reservierte Blöcke**  
    Für Root reservierter Speicherplatz (Standard: 5%) zur Systemstabilisierung
    
- **Partitionstypen**
    
    - `82`: Linux Swap
    - `83`: Linux Filesystem
    - `EF`: EFI System