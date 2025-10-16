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
	