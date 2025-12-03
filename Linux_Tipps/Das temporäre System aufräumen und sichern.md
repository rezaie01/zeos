## Aufräumen
- Die Dokumentation-Dateien aufräumen, um sie nicht ins finale System reinlassen:
```bash
rm -rf /usr/share/{info,man,doc}/*
```

- In modernen Linux-Betiebssysteme `.la`-Dateien sind nur für `libtdl` benutzt, aber in LFS keine Bibliothek wird mit `libtdl` aufgeladen, und auch manche `.la`-Dateien könnte LFS- oder BLFS-Packet Fehler verursachen.
```bash
 find /usr/{lib,libexec} -name \*.la -delete 
```

- Die `/tools` Verzeichnis ist nicht mehr benötigt:
```bash
 rm -rf /tools 
```

## Sichern

- Ch-root verlassen: `exit`
- Die virtuelle Dateisysteme un-einbindigen 

```bash
mountpoint -q $LFS/dev/shm && umount $LFS/dev/shm
umount $LFS/dev/pts
umount $LFS/{sys,proc,run,dev}

# meine Methode
sudo umount -vR $LFS
sudo mount $LFS
```

- Versicherungs-Datei erstellen
```bash
cd $LFS
tar -cJpf $HOME/lfs-temp-tools-12.4.tar.xz .

# Meine Methode
sudo umount $LFS
sudo fsarchiver -v savefs /files/LFS/snaptshots/chp07.fsa /dev/nvme0n1p7
```

- Zurücksetzen

```bash
cd $LFS
rm -rf *
tar -xpf $HOME/lfs-temp-tools-12.4.tar.xz

# Meine Methode
sudo umount $LFS
sudo fsarchiver -v restfs /files/LFS/snaptshots/chp07.fsa id=0,dest=/dev/nvme0n1p7
```

