
## 1) Was sind Coreutils?

Coreutils (Core Utilities) enthalten die grundlegenden Kommandozeilenprogramme für Dateiverwaltung, Textbearbeitung, Dateisystemabfragen und andere Standardoperationen in Linux.  
Im LFS-Kontext werden sie für die temporäre Build-Umgebung kompiliert, um alle grundlegenden Tools wie `ls`, `cp`, `mv`, `rm`, `chmod` usw. verfügbar zu haben.

---

## 2) **Vorbereiten zur Kompilierung**

```
./configure --prefix=/usr \
  --host=$LFS_TGT \
  --build=$(build-aux/config.guess) \
  --enable-install-program=hostname \
  --enable-no-install-program=kill,uptime
```

> **ERKLÄRUNG:**  
> *`--build=$(build-aux/config.guess)` — ermittelt automatisch die Build-Maschine.*  
> *`--enable-install-program=hostname` — stellt sicher, dass das Programm `hostname` installiert wird.*  
> *`--enable-no-install-program=kill,uptime` — verhindert, dass die Programme `kill` und `uptime` installiert werden, da diese bereits durch das Host-System verfügbar sind und Konflikte vermeiden sollen.*

---

## 3) **Kompilieren**

```
make
```

---

## 4) **Installieren**

```
make DESTDIR=$LFS install
```

---

## 5) **Anpassen von `chroot`**

```
mv -v $LFS/usr/bin/chroot $LFS/usr/sbin
mkdir -pv $LFS/usr/share/man/man8
mv -v $LFS/usr/share/man/man1/chroot.1 $LFS/usr/share/man/man8/chroot.8
sed -i 's/"1"/"8"/' $LFS/usr/share/man/man8/chroot.8
```

> **ERKLÄRUNG:**  
> _`mv -v $LFS/usr/bin/chroot $LFS/usr/sbin` — verschiebt das Programm `chroot` von `/usr/bin` nach `/usr/sbin`. Viele Distributionen halten `chroot` in `/sbin`, da es eher für Administratoren gedacht ist._  
> _`mkdir -pv $LFS/usr/share/man/man8` — erstellt das Verzeichnis für Manpages der Kategorie 8 (Systemadministration) falls es noch nicht existiert._  
> _`mv -v $LFS/usr/share/man/man1/chroot.1 $LFS/usr/share/man/man8/chroot.8` — verschiebt die Manpage von Kapitel 1 (normale Benutzerbefehle) in Kapitel 8 (Systemadministration), passend zur neuen Position des Programms._  
> _`sed -i 's/"1"/"8"/' $LFS/usr/share/man/man8/chroot.8` — ersetzt in der Manpage die Kapitelangabe von "1" zu "8", damit sie korrekt zur neuen Kategorie passt._


---

# Problem Während des Compilieren:

## `error "Assumed value of MB_LEN_MAX wrong"`
```
In file included from /mnt/lfs/usr/include/stdlib.h:1165,  
                from ./lib/stdlib.h:51,  
                from lib/mbscasecmp.c:26:  
/mnt/lfs/usr/include/bits/stdlib.h: In function 'wctomb':  
/mnt/lfs/usr/include/bits/stdlib.h:98:3: error: #error "Assumed value of MB_LEN_MAX wrong"  
  98 | # error "Assumed value of MB_LEN_MAX wrong"  
     |   ^~~~~  
 CC       lib/libcoreutils_a-mbsrtowcs.o  
In file included from /mnt/lfs/usr/include/wchar.h:1070,  
                from ./lib/wchar.h:83,  
                from ./lib/uchar.h:61,  
                from lib/mbscasecmp.c:27:
```
### Lösung:

Diese Fehlermeldung weist normalerweise darauf hin, dass das von GCC bereitgestellte limits.h nicht wie vorgesehen limits.h von Glibc enthält. Man soll `limits.h` erstellen. Wie bei der `GCC Pass 1`: (In GCC-Quellverzeichnis)

```
cat gcc/limitx.h gcc/glimits.h gcc/limity.h > \
`dirname $($LFS_TGT-gcc -print-libgcc-file-name)`/include/limits.h
```
