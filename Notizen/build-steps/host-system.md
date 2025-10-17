#partitionierung
# Host System

---
## parttionierung des Hostsystems:

| Nummer | Größe             | Name       | Beschreibung                                                                                                                                      |
| ------ | ----------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1      | 300 MiB - 550 MiB | EFI System | am Besten fat32, wird auf /boot/efi eingebunden                                                                                                   |
| 2      | 500               | /boot      | 0% reservierte Blöcke, Eingebundene auf on /boot, besser als ext2 *(Kein Journaling = weniger Schreibzugriffe = schneller & ressourcenschonend.)* |
| 3      | 2 swap            | swap       |                                                                                                                                                   |
| 4      | 20gb              | /          | ext4                                                                                                                                              |

---
### Fachwörte
1. **Journaling**: **Journaling** ist ein Sicherheitsmechanismus in Dateisystemen, der Änderungen zuerst in einem Protokoll (Journal) speichert, bevor sie wirklich auf die Festplatte geschrieben werden. 
	- **Vorteile**: **schnell und sicher wiederhergestelltung** während einer Stromausfällung/des Absturz (crush)
2. **Reservierte Blöcke**: Wie viel Speicherplatz der Partition soll exklusiv dem Root-Benutzer zugewiesen werden? Prävention von Abstürze, wenn die Festplatte voll ist. 

---
## Vorbereitung des Hostsystems:

### 1. Mein Hostsystem:

- Zorin Os
- Details: 
	![[Pasted image 20251015031513.png]]

### 2. alle benötigten Packages installieren
**Benötigte Packages:** `Bash-3.2 Binutils-2.13.1  Bison-2.7 Coreutils-8.1 • Diffutils-2.8.1 Findutils-4.2.31 Gawk-4.0.1 GCC-5.4 g++ Grep-2.5.1a Gzip-1.3.12 Linux Kernel-5.4 M4-1.4.10 Make-4.0 Patch-2.5.4 Perl-5.8.8 Python-3.4 Sed-4.1.5 Tar-1.22 Texinfo-5.0 Xz-5.0.0`
`sudo apt install binutils bison texinfo build-essential`
### 3. Partitionierung für LFS:

> *Eine #Partition ist nur ein **Sektorbereich** auf einer #Festplatte, der durch Grenzen in einer #Partitionstabelle festgelegt ist. 
> Eine Partition muss formatiert werden, um ein Dateisystem zu enthalten, das Dateien verwaltet, Speicherplatz organisiert und Datenintegrität sichert.*

1. Die Hauptpartition beschränken: Benutze ein Live-Boot-Betriebssystem um `/dev/sda3` zu verkleinern.
2. Erstelle eine neue Partition für einen Linux-Auslagerungsbereich (swap): 
	1. Erstelle die Partition `fdisk`, und #Partitionstyp `Linux Filesystem`
	2. Setze den Partitiontyp (Dateisystem) auf `Linux Swap` ein: `sudo mkswap -L swap-partition /dev/sda4` #mkswap
3. Erstelle eine neue Partition für LFS (Linux from Scratch).
	1. Erstelle die Partition `fdisk`, 
	2. Setze den #Partitionstyp (Dateisystem) auf `ext4` ein: `sudo mkfs.ext4 -L LFS-root /dev/sda5` #mkfs

### 4. Festlegen der Umgebungsvariable `$LFS`  und der `Umask` (root & Benutzer):
>- Login (TTY, SSH) → `.bash_profile`
>-  GUI-Terminal/graphisches Login→ `.bashrc`
>- Skript → nichts automatisch → `source .bashrc`/`source .bash_profile`/ definiere die Variablen

- lege `umask` als `022` fest.

> #umask #berechtigung
>**Benützer Arte**: Eigentümer, Gruppe, Andere
>**Berechtigungsarte**: 
	- **Datei** r = Lesen = 4, w = schreiben (Ändern und Entfernen) = 2, x = Ausführen = 1
	- **Verzeichnis**: r = Listen, w = Hinzufügen/Löschen, x = Verzeichnis betreten
	

- lege `$LFS` als `/mnt/lfs` oder ein anderes Verzeichnis.

### 5.  Das Einbinden der neuen Partition:
>Man bindet nicht die Partition ein sondern das Dateisystem. **Aber** jede Partition kann nur ein Dateisystem enthalten. 


```sh
sudo mkdir -pv $LFS
sudo mount -v -t ext4 /dev/sda5 $LFS
```

Wenn ich es immer einbindende möchte, soll ich dann im `/etc/fstab` die Zeile `/dev/sda5 /mnt/lfs ext4 1 1` hinzufügen. 