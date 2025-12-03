#BLFS #UEFI #GRUB #EFISystemPartition #Bootloader

## Using GRUB to Set Up the Boot Process with UEFI

### Secure Boot ausschalten

Bevor ich anfange, muss ich _Secure Boot_ im Konfigurations-Interface der Firmware ausschalten. Das ist notwendig, weil BLFS die nötigen Pakete für Secure Boot nicht enthält. Ich muss das Handbuch meines Systems lesen, um zu erfahren, wie das geht.

### Kernel-Konfiguration für UEFI-Unterstützung

Ich muss sicherstellen, dass mein Kernel die nötige Unterstützung für UEFI hat, sonst muss ich ihn neu kompilieren. Folgende Optionen sind wichtig:

- `EFI runtime service support` und `EFI stub support` (unter _Processor type and features_).
- `EFI GUID Partition support` (unter _Partition Types_). Wenn `CONFIG_PARTITION_ADVANCED` aktiviert ist, muss `CONFIG_EFI_PARTITION` ebenfalls aktiviert sein.
- `VFAT (Windows-95) fs support` (unter _DOS/FAT/EXFAT/NT Filesystems_).
- `EFI Variable filesystem` (`EFIVAR_FS`) (unter _Pseudo filesystems_).
- Codepage 437 und NLS ISO 8859-1 (unter _Native language support_).

### Notfall-Boot-Disk erstellen

Es ist super wichtig, eine Notfall-Boot-Disk (z. B. einen USB-Stick) zu haben, um das System zu retten, falls es nicht mehr bootet.

1. **Dateisystem erstellen:** Ich muss `dosfstools-4.2` installieren und dann die Partition formatieren. **Achtung:** Ich muss die Gerätedatei (hier: `sdx1`) richtig setzen, damit ich nicht versehentlich meine Festplatte lösche.
    
    ```
    CODEBLOCK 1:
    mkfs.vfat /dev/sdx1
    ```
    
    **Erklärung:** Dieser Befehl formatiert die erste Partition (`/dev/sdx1`) des USB-Sticks mit dem `vfat`-Dateisystem (was für die ESP notwendig ist).
    
2. **Partitionstyp setzen:** Die erste Partition des USB-Sticks muss der Typ _EFI system_ sein. Ich benutze dafür `fdisk`.
    
    ```
    CODEBLOCK 2:
    fdisk /dev/sdx
        Welcome to fdisk (util-linux 2.39.1).
        Changes will remain in memory only, until you decide to write them.
        Be careful before using the write command.
    
        Command (m for help): t
        Partition number (1-9, default 9): 1
        Partition type or alias (type L to list all): uefi
        Changed type of partition 'Linux filesystem' to 'EFI System'.
    
        Command (m for help): w
        The partition table has been altered.
        Syncing disks.
    ```
    
    **Erklärung:** Ich starte `fdisk` für das Gerät (`/dev/sdx`), drücke `t` um den Typ zu ändern, wähle Partition 1 und setze den Typ auf `uefi` (was _EFI System_ entspricht). `w` speichert die Änderungen auf der Platte.
    
3. **Partition mounten:** Ich erstelle einen Mountpoint und mounte die Partition.
    
    ```
    CODEBLOCK 3:
    mount --mkdir -v -t vfat /dev/sdx1 -o codepage=437,iocharset=iso8859-1 \
    /mnt/rescue
    ```
    
    **Erklärung:** Ich mounte `/dev/sdx1` im Verzeichnis `/mnt/rescue` als `vfat` und nutze spezifische Optionen für Codepage 437 und Zeichensatz ISO 8859-1.
    
4. **GRUB installieren:** Ich installiere GRUB für EFI auf den Stick.
    
    ```
    CODEBLOCK 4:
    grub-install --target=x86_64-efi --removable \
    --efi-directory=/mnt/rescue --boot-directory=/mnt/rescue
    ```
    
    **Erklärung:** Dieser Befehl installiert GRUB für die 64-Bit-EFI-Plattform. Der Schalter `--removable` sorgt dafür, dass der Stick auf jedem x86-64 UEFI-System bootbar ist. Ich definiere das EFI-Verzeichnis und das Boot-Verzeichnis beide als `/mnt/rescue`.
    
5. **Partition unmounten:**
    
    ```
    CODEBLOCK 5:
    umount /mnt/rescue
    ```
    
    **Erklärung:** Trennt den USB-Stick vom Dateisystem.
    

### EFI System Partition (ESP) finden oder erstellen

Auf UEFI-Systemen werden Bootloader in einer speziellen FAT32-Partition, der _EFI System Partition_ (ESP), installiert.

1. **ESP finden:** Ich benutze `fdisk -l`, um meine Festplatte (`sda`) zu überprüfen und nach einer Partition mit dem Typ _EFI System_ zu suchen.
    
    ```
    CODEBLOCK 6:
    fdisk -l /dev/sda
    ```
    
    **Erklärung:** Listet die Partitionen auf `/dev/sda` auf, damit ich die ESP identifizieren kann.
    
2. **ESP erstellen:** Falls keine ESP existiert, muss ich sie erstellen (FAT32, Typ _EFI system_). Manchmal muss die ESP die erste Partition sein (bei älteren UEFI-Implementierungen).
    
3. **ESP mounten:** Ich erstelle den Mountpoint `/boot/efi` und mounte die ESP dorthin (hier `/dev/sda1`).
    
    ```
    CODEBLOCK 7:
    mount --mkdir -v -t vfat /dev/sda1 -o codepage=437,iocharset=iso8859-1 \
    /boot/efi
    ```
    
    **Erklärung:** Mountet die ESP, wobei dieselben `vfat`-Optionen wie bei der Notfall-Disk verwendet werden.
    
4. **In `/etc/fstab` eintragen:** Damit die ESP bei jedem Boot automatisch gemountet wird.
    
    ```
    CODEBLOCK 8:
    cat >> /etc/fstab << EOF
      /dev/sda1 /boot/efi vfat codepage=437,iocharset=iso8859-1 0 1
      EOF
    ```
    
    **Erklärung:** Fügt den Eintrag für die automatische Einhängung der ESP in die Konfigurationsdatei `/etc/fstab` ein.
    

### Minimale Boot-Konfiguration mit GRUB und EFI (optional)

Normalerweise vermeidet man den _hardcoded path_, aber in bestimmten Fällen (z. B. wenn das System nicht mit EFI gebootet ist, 32-Bit LFS auf 64-Bit Firmware, oder Live USB) muss ich den fest codierten Pfad `EFI/BOOT/BOOTX64.EFI` nutzen.

Dazu muss ich die Boot-Partition nach `/boot` und die ESP nach `/boot/efi` gemountet haben.

```
CODEBLOCK 9:
grub-install --target=x86_64-efi --removable
```

**Erklärung:** Dieser Befehl installiert die GRUB EFI-Anwendung in den fest codierten Pfad `/boot/efi/EFI/BOOT/BOOTX64.EFI`, damit die Firmware sie laden kann, falls sie die EFI-Variablen nicht nutzen kann. **Vorsicht:** Dies überschreibt jeden Bootloader, der sich dort bereits befindet.

### EFI Variable File System mounten

Für die Standard-GRUB-Installation auf UEFI muss das _EFI Variable File System_ (`efivarfs`) gemountet sein.

1. **`efivarfs` mounten:** Ich prüfe, ob es gemountet ist, und wenn nicht, mounte ich es.
    
    ```
    CODEBLOCK 10:
    mountpoint /sys/firmware/efi/efivars || mount -v -t efivarfs efivarfs /sys/firmware/efi/efivars
    ```
    
    **Erklärung:** Der Befehl prüft mit `mountpoint`, ob das Verzeichnis bereits gemountet ist. Wenn nicht (`||`), wird `efivarfs` im Verzeichnis `/sys/firmware/efi/efivars` gemountet.
    
2. **In `/etc/fstab` eintragen:** Damit `efivarfs` automatisch gemountet wird.
    
    ```
    CODEBLOCK 11:
    cat >> /etc/fstab << EOF
      efivarfs /sys/firmware/efi/efivars efivarfs defaults 0 0
      EOF
    ```
    
    **Erklärung:** Fügt den Eintrag für `efivarfs` in `/etc/fstab` hinzu.
    

### Konfiguration einrichten (Standard-Installation)

Die Standardmethode installiert die EFI-Anwendung (`grubx64.efi`) in einen spezifischen Pfad und registriert einen Starteintrag in den EFI-Variablen, sodass die Firmware den Bootloader findet.

Ich verwende `LFS` als Kennung (`[id]`). Der Bootloader wird in `/boot/efi/EFI/LFS/grubx64.efi` installiert und die GRUB-Dateien nach `/boot/grub`.

```
CODEBLOCK 12:
grub-install --bootloader-id=LFS --recheck
```

**Erklärung:** Installiert GRUB und erstellt einen neuen Starteintrag in den EFI-Variablen mit der Kennung `LFS`.

Die erfolgreiche Ausgabe sieht so aus:

```
CODEBLOCK 13:
Installing for x86_64-efi platform.
Installation finished. No error reported.
```

Anschließend kann ich die Boot-Konfiguration mit `efibootmgr` überprüfen, um sicherzustellen, dass `LFS` im `BootOrder` an erster Stelle steht.

```
CODEBLOCK 14:
BootCurrent: 0000
Timeout: 1 seconds
BootOrder: 0005,0000,0002,0001,0003,0004
Boot0000* ARCH
Boot0001* UEFI:CD/DVD Drive
Boot0002* Windows Boot Manager
Boot0003* UEFI:Removable Device
Boot0004* UEFI:Network Device
Boot0005* LFS
```

**Erklärung:** Diese Beispielausgabe zeigt, dass der Eintrag `LFS` (`Boot0005`) erfolgreich erstellt wurde und an erster Stelle in der Boot-Reihenfolge (`BootOrder`) steht.

### Erstellen der GRUB-Konfigurationsdatei

Jetzt erstelle ich die Datei `/boot/grub/grub.cfg`, um das Bootmenü zu konfigurieren.

```
CODEBLOCK 15:
cat > /boot/grub/grub.cfg << EOF
  # Begin /boot/grub/grub.cfg
  set default=0
  set timeout=5
  insmod part_gpt
  insmod ext2
  set root=(hd0,2)
  insmod efi_gop
  insmod efi_uga
  if loadfont /boot/grub/fonts/unicode.pf2; then
    terminal_output gfxterm
  fi
  menuentry "GNU/Linux, Linux 6.16.1-lfs-12.4" {
    linux /boot/vmlinuz-6.16.1-lfs-12.4 root=/dev/sda2 ro
  }
  menuentry "Firmware Setup" {
    fwsetup
  }
  EOF
```

**Erklärung:** Die Konfiguration setzt den Standardeintrag auf 0 und das Timeout auf 5 Sekunden. Es lädt die Module `part_gpt` und `ext2`. `set root=(hd0,2)` definiert die Root-Partition aus GRUB-Sicht. `efi_gop` und `efi_uga` sind für die Grafikausgabe (Video-Support) nötig. Der `if loadfont` Block sorgt dafür, dass die grafische Terminalschnittstelle (`terminal_output gfxterm`) nur aktiviert wird, wenn die Schriftart geladen werden kann. Der Menüeintrag startet den Kernel, wobei `root=/dev/sda2` auf meine Root-Partition zeigen muss. Der Eintrag `Firmware Setup` erlaubt den direkten Zugriff auf das UEFI/BIOS-Interface.

### Dual-Boot mit Windows

Wenn ich Windows parallel installiert habe, füge ich folgenden Eintrag zu `grub.cfg` hinzu.

```
CODEBLOCK 16:
cat >> /boot/grub/grub.cfg << EOF
  # Begin Windows addition
  menuentry "Windows 11" {
    insmod fat
    insmod chain
    set root=(hd0,1)
    chainloader /EFI/Microsoft/Boot/bootmgfw.efi
  }
  EOF
```

**Erklärung:** Dieser Block hängt einen Menüeintrag für Windows 11 an. Er lädt die Module `fat` und `chain`. `set root=(hd0,1)` muss auf die ESP (EFI System Partition) verweisen, und der `chainloader`-Befehl übergibt die Kontrolle an den Windows Boot Manager.

## Glossar

|Begriff|Erklärung (LFS/BLFS-Kontext)|
|:--|:--|
|**Secure Boot**|Eine Sicherheitsfunktion der UEFI-Firmware, die verhindert, dass nicht signierte Software beim Booten ausgeführt wird. Muss für BLFS ausgeschaltet werden.|
|**UEFI**|Unified Extensible Firmware Interface, der moderne Ersatz für das BIOS.|
|**EFI System Partition (ESP)**|Eine spezielle FAT32-Partition, die Bootloader (wie GRUB) auf UEFI-Systemen speichert.|
|**`efivarfs`**|Das EFI Variable File System, das gemountet werden muss, um auf die EFI-Variablen (z. B. für die Boot-Reihenfolge) zugreifen zu können.|
|**`--removable`**|Ein Flag bei `grub-install`, das den Bootloader in den fest codierten Pfad (Hardcoded Path) installiert, nützlich für Notfall- oder USB-Medien.|
|**Hardcoded Path**|Der Pfad `EFI/BOOT/BOOTX64.EFI`, den die Firmware standardmäßig absucht, wenn keine EFI-Variablen verwendet werden können.|
|**`grubx64.efi`**|Die eigentliche EFI-Anwendung (Bootloader) von GRUB, die von der Firmware gestartet wird.|
|**`--bootloader-id=LFS`**|Ein Flag, das den Bezeichner für den Eintrag in den EFI-Variablen festlegt, hier "LFS".|
|**`chainloader`**|Ein GRUB-Befehl, der die Kontrolle an einen anderen Bootloader (z. B. den Windows Boot Manager) übergibt.|
|**`efi_gop`, `efi_uga`**|GRUB-Module, die für die grafische Video-Unterstützung im GRUB-Menü (gfxterm) auf EFI-Systemen sorgen.|