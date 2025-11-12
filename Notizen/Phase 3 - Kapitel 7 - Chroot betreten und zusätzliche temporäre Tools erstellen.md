# Phase 3: (Kapitel 7) Chroot betreten und zusätzliche temporäre Tools erstellen

## 1. Einleitung
- In den ersten beiden Phasen wurden die zirkulären Abhängigkeiten aufgelöst. Jetzt können wir ein vollständig unabhängiges System (außer dem laufenden Kernel) verwenden.
- Nun bauen wir die restlichen Werkzeuge, die zum Kompilieren verschiedener Pakete benötigt werden (z. B. **Perl** für Build-Skripte).
## 2. Eigentümerwechsel

Die Verzeichnisse in **$LFS** gehören derzeit dem Benutzer _lfs_. Wenn das so bleibt, würden sie im Zielsystem einem Benutzer gehören, der dort nicht existiert, sondern nur einer Benutzer-ID zugeordnet ist. Dadurch könnte später jemand absichtlich einen Benutzer mit derselben ID erstellen, was ein Sicherheitsrisiko darstellen würde.

```bash
chown --from lfs -R root:root $LFS/{usr,var,etc,tools}
case $(uname -m) in
x86_64) chown --from lfs -R root:root $LFS/lib64 ;;
esac
```
> **<font color="#9bbb59">ERKLÄRUNG</font>:**
>***`-R`***: *Rekursiv*

--- 
## 3 Vorbereiten der virtuellen Dateisysteme
 
### **Haupt Punkt:** 
 Kommunikation mit den Kernel wird mit den Virtuellen Dateisysteme ausgeführt, dafür Keinen Speicherplatz ausgeben.
```bash
mkdir -pv $LFS/{dev,proc,sys,run}
```

### **Mehr Dazu**:
Alles klar — lass uns das Schritt für Schritt durchgehen. Ich erkläre es **genau und praxisnah**, damit du die Konzepte wirklich verstehst.

---

#### 1️⃣ Virtuelle Dateisysteme – was ist das?

Ein **virtuelles Dateisystem** (engl. _virtual filesystem_, VFS) ist ein „Dateisystem“, das **nicht auf der Festplatte gespeichert wird**, sondern **vom Kernel bereitgestellt wird**, um Informationen über das System, Prozesse oder Geräte verfügbar zu machen.

- **Warum sagen sie, es nutzt keinen Speicherplatz?**  
    Weil die Dateien **nicht wirklich auf der Festplatte liegen**. Sie existieren nur **im RAM und werden vom Kernel verwaltet**.
    
    - Beispiel: `/proc/cpuinfo` enthält Infos über die CPU. Du kannst es lesen wie eine normale Datei, aber wenn du `du -h /proc/cpuinfo` machst, zeigt es fast **0 Bytes auf der Platte**, weil der Inhalt dynamisch generiert wird.
        
- **Warum /dev trotzdem Inhalt hat:**  
    `/dev` enthält Geräte-Knoten. Manche sind „echte“ Device-Files, manche werden vom Kernel dynamisch erzeugt. Sie belegen **praktisch keinen großen Speicher**, da sie nur Schnittstellen zu Geräten sind.
    

---

#### 2️⃣ Die wichtigsten virtuellen Dateisysteme

| Verzeichnis | Zweck                                                                          | Besonderheit                                                                                                                  |
| ----------- | ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------- |
| **/dev**    | Geräte-Schnittstellen (z. B. Festplatten, Tastatur, Maus, virtuelle Terminals) | Dateien wie `/dev/sda`, `/dev/tty` sind Device Nodes, keine echten Dateien.                                                   |
| **/proc**   | Prozess- und Kernelinformationen                                               | Jedes laufende Programm hat z. B. ein Unterverzeichnis `/proc/<PID>`. Systeminfos wie CPU, Memory, Mounts sind hier sichtbar. |
| **/sys**    | Gerätekonfiguration, Kernel-Objekte                                            | Zeigt Hardware und Treiber-Strukturen, z. B. `/sys/class/net/eth0`. Man kann Treiberparameter ändern (writeable files).       |
| **/run**    | Laufzeitdaten / temporäre Daten für Prozesse                                   | Z. B. PID-Files, Locks. Inhalt verschwindet nach Neustart (tmpfs).                                                            |

**Merke:** `/proc`, `/sys`, `/run` sind **tmpfs-artige Filesystems**, also dynamisch im RAM. `/dev` ist ähnlich (devtmpfs), enthält Geräte-Knoten, keine „echten Dateien“.

---

#### 3️⃣ Unterschied zwischen Userspace und Kernel

- **Kernel (Kernelspace)**
    
    - Ist der **Herzteil des Systems**.
        
    - Steuert Hardware, Speicherverwaltung, Prozess-Scheduling, Dateisysteme.
        
    - Läuft mit **höchsten Rechten** (Ring 0).
        
    - Greift direkt auf Geräte zu (CPU, RAM, Festplatte).
        
- **Userspace**
    
    - Alles, was **Programme oder Prozesse nutzen**, die nicht Kernel-Code sind (z. B. `bash`, `vim`, `firefox`).
        
    - Läuft mit eingeschränkten Rechten, darf nicht direkt auf Hardware zugreifen.
        
    - Kommuniziert mit Kernel über Systemcalls (`read`, `write`, `open`, `/proc`, `/dev`, `/sys`).
        

**Beispiel:**

- Wenn du `cat /proc/cpuinfo` machst, greifst du **aus Userspace auf das virtuelle Dateisystem /proc** zu, das der Kernel erzeugt.
    
- Du siehst CPU-Daten, aber der Kernel hat sie dynamisch zusammengestellt, **nicht von einer Datei auf der Festplatte gelesen**.
    

---

💡 **Merksatz:**

> Virtuelle Dateisysteme = RAM-generierte Dateien, die Kernel- und Gerätezustand für Userspace sichtbar machen.  
> Userspace sieht sie wie normale Dateien, Kernel erzeugt und verwaltet sie.

---

### **`/dev` wird eingebunden und befüllt**

```bash
sudo mount --bind /dev $LFS/dev
# ACHTUNG! Führe nicht die Oben mit den Unten auf eine linie aus.

sudo mount -vt devpts devpts -o gid=5,mode=0620 $LFS/dev/pts
sudo mount -vt proc proc $LFS/proc
sudo mount -vt sysfs sysfs $LFS/sys
sudo mount -vt tmpfs tmpfs $LFS/run
```


#### 1️⃣ Was ist `devtmpfs`?

- `devtmpfs` ist ein **virtuelles Dateisystem für /dev**, das vom **Kernel selbst erzeugt wird**.
    
- Aufgabe: Alle **Geräte-Knoten (device nodes)** automatisch im Verzeichnis `/dev` bereitstellen.
    
- Vorteil: Man **braucht keinen Userspace-Dienst**, um die Gerätedateien zu erstellen.
    
- Es ist **temporär im RAM** und verschwindet beim Herunterfahren.
    

**Kurz:** `devtmpfs` = automatisches, kernel-generiertes `/dev`.

---

#### 2️⃣ Was sind „device nodes“?

- **Device nodes** sind spezielle Dateien unter `/dev`, die **Schnittstellen zu Geräten** darstellen.
    
- Es gibt zwei Haupttypen:
    
    1. **Character devices (c)** → Zugriff byteweise (z. B. `/dev/tty`, `/dev/random`)
        
    2. **Block devices (b)** → Zugriff blockweise (z. B. `/dev/sda`, `/dev/sdb`)
        
- Sie enthalten **keinen echten Inhalt**, sondern **verweisen auf ein Gerät im Kernel**.
    
- Operationen wie `read`/`write` auf diese Dateien werden vom Kernel zum Gerät weitergeleitet.
    

**Beispiel:**

```bash
ls -l /dev/sda
# brw-rw---- 1 root disk 8, 0 Okt  1 12:00 /dev/sda
```

- `b` = Blockgerät, `8,0` = major/minor number, zeigt dem Kernel, welches Gerät gemeint ist.
    

---

#### 3️⃣ Was ist der `udev`-Daemon? Und was ist ein daemon?

- **Daemon:** Ein Hintergrundprozess, der dauerhaft läuft und Aufgaben erledigt, ohne dass du manuell etwas startest.
    
    - Vergleich: Windows-Service.
        
- **`udev` daemon**
    
    - Ein Userspace-Dienst, der `/dev` dynamisch verwaltet.
        
    - Reagiert, wenn Geräte angeschlossen/entfernt werden.
        
    - Erstellt Device Nodes automatisch und setzt die richtigen Rechte.
        

**Kurz:**

- Früher musste man Device Nodes **manuell erstellen** (z. B. mit `mknod`).
    
- Heute erledigt das **udev** automatisch für angeschlossene Geräte.
    

---

#### 4️⃣ Manuell vs `devtmpfs`

- **Manuelle Methode:**
    
    - Du mountest einfach `/dev` vom Host ins `$LFS`-System:
        
        ```bash
        mount --bind /dev $LFS/dev
        ```
        
    - Vorteil: Einfach, keine Kernel-Funktion nötig.
        
    - Nachteil: Du übernimmst **alle Gerätedateien vom Host**, nicht automatisch neue Geräte oder dynamische Rechte.
        
- **`devtmpfs` Methode:**
    
    - Kernel erstellt **alle Gerätedateien selbst**.
        
    - Vorteil:
        
        - Aktuelle Geräte sind automatisch vorhanden
            
        - Gerätedateien verschwinden/erscheinen dynamisch, wenn Geräte angeschlossen werden
            
        - Bessere Kontrolle im neuen System
            
    - Nachteil: Funktioniert nur, wenn der Kernel `devtmpfs` unterstützt
        

**Unterschied:**

|Methode|Wer erstellt /dev|Dynamik|Abhängigkeit|
|---|---|---|---|
|Manuell bind|Hostsystem|Nein|Host|
|devtmpfs|Kernel|Ja|Kernel unterstützt devtmpfs|

---

💡 **Merksatz:**

- **Device nodes** = Schnittstellen zu Hardware
- **devtmpfs** = Kernel erstellt automatisch alle /dev-Dateien
- **udev** = Userspace-Dienst für dynamisches /dev
- **--bind** = zeigt /dev im Zielverzeichnis (z.B `/dev` wird auch im $LFS/dev, weniger dynamisch)
### **Die andere virtuelle Verzeichnisse einbinden**

```bash
mount -vt devpts devpts -o gid=5,mode=0620 $LFS/dev/pts
mount -vt proc proc $LFS/proc
mount -vt sysfs sysfs $LFS/sys
mount -vt tmpfs tmpfs $LFS/run
```

---

 **1️⃣ Warum `gid=5` und `mode=0620` für `/dev/pts`?**

- `/dev/pts` ist das **Pseudo-Terminal-Dateisystem**, in dem virtuelle Terminals (z. B. Konsolen, `xterm`, `ssh`) erstellt werden.
    
- **`gid=5`** → Gruppe `tty` (traditionelle Terminalgruppe)
    
    - Nur Nutzer in dieser Gruppe dürfen das Pseudo-Terminal benutzen.
        
- **`mode=0620`** → Berechtigungen:
    
    - `6` = owner (root) → lesen+schreiben
        
    - `2` = group (tty) → schreiben
        
    - `0` = andere → keine Rechte
        
- **Zweck:** Sicherheit, dass nur root oder tty-Gruppe Zugriff auf Pseudo-Terminals hat.
    

---

 **2️⃣ Was ist `devpts`?**

- **`devpts`** = virtuelles Dateisystem für **Pseudo-Terminals**.
    
- Jedes neue Terminal bekommt automatisch einen Eintrag in `/dev/pts`, z. B. `/dev/pts/0`, `/dev/pts/1`.
    
- Dient als Schnittstelle für Prozesse, die Terminals benutzen (Shells, SSH, Screen/Tmux).
    

---

 **3️⃣ Was ist `sysfs`?**

- **`sysfs`** = virtuelles Dateisystem, das **Kernel-Objekte und Hardwareinformationen** zeigt.
    
- Beispiel: `/sys/class/net/eth0` zeigt Netzwerkgeräte, `/sys/block/sda` zeigt Festplatten.
    
- Ermöglicht: Lesen und teilweise Schreiben von Geräteparametern durch Userspace.
    

---

 **4️⃣ Was ist `tmpfs`?**

- **`tmpfs`** = temporäres Dateisystem, komplett im RAM.
    
- Beispiel: `/run` wird darauf gemountet → enthält **laufzeitbezogene Daten** (PID-Files, Locks).
    
- Vorteil: Schnell, verschwindet beim Neustart, nutzt **nur so viel RAM, wie benötigt**.
    

---

💡 **Merksatz:**

| Mount  | Zweck             | Besonderheit                                  |
| ------ | ----------------- | --------------------------------------------- |
| devpts | Pseudo-Terminals  | dynamische Terminals, gid=5, mode=0620        |
| sysfs  | Kernel & Hardware | zeigt Geräte und Treiberinformationen         |
| tmpfs  | Laufzeitdaten     | temporär im RAM, `/run`, schnell, vergänglich |

### **Die Temporäre-Dateisystem `/dev/shm` einbindigen**

**1️⃣ Zweck von `/dev/shm`**

- `/dev/shm` = **Shared Memory** → temporärer Speicher im RAM für Prozesse.
    
- Anwendungen (z. B. `systemd`, `glibc`) nutzen `/dev/shm`, um **gemeinsame Speicherbereiche** zwischen Prozessen anzulegen.
    
- Wird normalerweise als **`tmpfs`** gemountet → RAM-basiert, automatisch beim Neustart leer.
    

---

**2️⃣ Erklärung des Codes**

- In manchen Host-Betriebssystemen ist `/dev/shm` ein symbolischer Link, meist zu `/run/shm`.
- In anderen Systemen ist es ein Mountpunkt für ein `tmpfs`.

```bash
if [ -h $LFS/dev/shm ]; then
    install -v -d -m 1777 $LFS$(realpath /dev/shm)
else
    mount -vt tmpfs -o nosuid,nodev tmpfs $LFS/dev/shm
fi
```

#### Schritt für Schritt

1. **`if [ -h $LFS/dev/shm ]`**
    
    - Prüft, ob `$LFS/dev/shm` **ein symbolischer Link** (`-h`) ist.
        
    - Manche Distributionen machen `/dev/shm → /run/shm`.
        
2. **`install -v -d -m 1777 $LFS$(realpath /dev/shm)`**
    
    - Falls `/dev/shm` ein Link ist:
        
        - `realpath /dev/shm` → löst den Link auf, z. B. `/run/shm`
            
        - `install -d -m 1777 …` → erstellt das Verzeichnis mit **Berechtigung 1777**:
            
            - `1` → Sticky-Bit → nur Eigentümer kann Dateien löschen
                
            - `777` → alle dürfen lesen, schreiben, ausführen
                
    - `-v` → zeigt, was erstellt wird
        
3. **`else`**
    
    - Wenn `/dev/shm` **kein Link** ist:
        
4. **`mount -vt tmpfs -o nosuid,nodev tmpfs $LFS/dev/shm`**
    
    - Mountet ein **tmpfs** auf `$LFS/dev/shm`
        
    - Optionen:
        
        - `nosuid` → keine SUID/SGID-Dateien
            
        - `nodev` → keine Geräte-Dateien
            
    - Ziel: `/dev/shm` ist jetzt **RAM-basiert und sicher**
        

---

**3️⃣ Zusammenfassung**

- Prüft, ob `/dev/shm` ein Link ist → behandelt entsprechend.
- Wenn Link → erstellt Zielverzeichnis mit 1777 (seihe [[Berechtigungssystem im Linux]])
- Wenn kein Link → tmpfs mounten
- Ergebnis: `/dev/shm` ist **korrekt eingerichtet**, dynamisch, sicher für Shared Memory.
---

💡 **Merksatz:**

> `/dev/shm` muss im LFS tmpfs sein oder auf ein richtiges Verzeichnis zeigen, sonst funktionieren Shared-Memory-Anwendungen nicht.

---




## `chroot` betreten

```bash
chroot "$LFS" /usr/bin/env -i \
	HOME=/root \
	TERM="$TERM" \
	PS1='(lfs chroot) \u:\w\$ ' \
	PATH=/usr/bin:/usr/sbin \
	MAKEFLAGS="-j$(nproc)" \
	TESTSUITEFLAGS="-j$(nproc)" \
	/bin/bash --login
```
> **<font color="#9bbb59">ERKLÄRUNG</font>:**
 ***`TERM="$TERM"`***: *Weist den Variable `TERM` denselben Wert so wie jetzigen Umgebung.
 Dies ist benötigt für die Programme so wie `vim` oder `less`*
> 
 ***`$nproc`***: *soll ersetzt werden, wenn man nicht alle Kerne dafür benutzen will.*

><font color="#938953">HINWEIS:</font>
>*`/tools/bin`* ist nicht immer im Pfad, weil der Cross-Toolchain nicht mehr benutzt wird.
>

--- 
### Problem `chroot: failed to run command ‘/usr/bin/env’: No such file or directory`

#### Was ist passiert?
Es gibt kein `env` in `/usr/bin`, oder die ,,shared Libraries'' sind nicht richtig verbunden.
-  überprüfen: `ldd $LFS/usr/bin/env`
	 `/lib64/ld-linux-x86-64.so.2 => /usr/lib64/ld-linux-x86-64.so.2 (0x00007f1a1f708000)` 
		Hier die `/usr/lib64/ld-linux-x86-64.so.2` existiert nicht. Aber `/usr/lib/ld-linux-x86-64.so.2` existiert. 
- Lösen:
		Erstelle eine Symlink zu `/usr/lib/ld-linux-x86-64.so.2`
	```bash
	cd $LFS/lib64
	sudo ln -sv ../lib/ld-linux-x86-64.so.2	# Das erstellt `ld-linux-x86-64.so.2` im `$LFS/lib64`. 
	# ACHTUNG! Der Pfad ist relativ. d.h In der chroot-Umgebung kann die Bibliothek gefunden werden.   
	```


---

## Erstellen vom vollständigen Verzeichnisstruktur 

```md
/
├── boot          → Enthält Bootloader-Dateien und Kernel-Images.
├── home          → Benutzerverzeichnisse.
├── root          → Das Verzeichnis für den Benutzer `root`. mode: 0750
├── mnt           → Einhängepunkt für temporäre Dateisysteme.
├── opt           → Optionale Softwarepakete.
├── srv           → Daten für vom System bereitgestellte Dienste.
├── etc
│   ├── opt       → Konfigurationsdateien für optionale Pakete.
│   └── sysconfig → Systemkonfigurationsdateien.
├── lib
│   └── firmware  → Gerätetreiber-Firmware-Dateien.
├── media
│   ├── floppy    → Einhängepunkt für Disketten.
│   └── cdrom     → Einhängepunkt für CD-ROMs.
├── usr
│   ├── include   → Standard-Header-Dateien für die Entwicklung.
│   ├── src       → Quellcode für Systemsoftware.
│   ├── lib
│   │   └── locale → Lokalisierungsdaten für Programme.
│   ├── share
│   │   ├── color     → Farbdefinitionsdateien.
│   │   ├── dict      → Wörterbuchdateien.
│   │   ├── doc       → Dokumentationsdateien.
│   │   ├── info      → GNU-Info-Dateien.
│   │   ├── locale    → Lokale Sprach- und Gebietsschemas.
│   │   ├── man       → Handbuchseiten.
│   │   │   ├── man1 … man8 → Handbuchseiten nach Abschnitt.
│   │   ├── misc      → Verschiedene gemeinsame Daten.
│   │   ├── terminfo  → Terminalfähigkeitsdaten.
│   │   └── zoneinfo  → Zeitzonendaten.
│   └── local
│       ├── include → Lokale Header-Dateien.
│       ├── src     → Lokaler Quellcode.
│       ├── bin     → Lokale ausführbare Programme.
│       ├── lib     → Lokale Bibliotheken.
│       ├── sbin    → Lokale Systemverwaltungs-Binaries.
│       └── share
│           ├── color     → Lokale Farbdefinitionsdateien.
│           ├── dict      → Lokale Wörterbuchdateien.
│           ├── doc       → Lokale Dokumentationsdateien.
│           ├── info      → Lokale GNU-Info-Dateien.
│           ├── locale    → Lokale Lokalisierungsdaten.
│           ├── man       → Lokale Handbuchseiten.
│           │   ├── man1 … man8 → Lokale Handbuchseiten nach Abschnitt.
│           ├── misc      → Lokale verschiedene gemeinsame Daten.
│           ├── terminfo  → Lokale Terminalfähigkeitsdaten.
│           └── zoneinfo  → Lokale Zeitzonendaten.
└── var
    ├── tmp       → Temporäre Verzeichnisse für alle Benutzer. (Symlink zu /tmp). mode: 1777 stick-bit gesetzt. 
    ├── run       → Laufzeitvariable Daten (Symlink zu /run).
        ├── lock  → Sperrdateien für Systemressourcen. (Symlink zu /run/lock).
    ├── cache     → Zwischengespeicherte Daten von Anwendungen.
    ├── local     → Lokale variable Daten.
    ├── log       → Systemprotokolldateien.
    ├── mail      → Mail-Spools der Benutzer.
    ├── opt       → Variable Daten für optionale Pakete.
    ├── spool     → Warteschlangen für Aufgaben (z.B. Druck, Mail).
    └── lib
        ├── color  → Farbbezogene variable Daten.
        ├── misc   → Verschiedene variable Daten.
        └── locate → Datenbank für den Befehl locate.
```

```bash
mkdir -pv /{boot,home,mnt,srv,opt}
mkdir -pv /etc/{opt,sysconfig}
mkdir -pv /lib/firmware
mkdir -pv /media/{floppy,cdrom}
mkdir -pv /usr/{,local/}{include,src}
mkdir -pv /usr/lib/locale
mkdir -pv /usr/local/{bin,lib,sbin}
mkdir -pv /usr/{,local/}share/{color,dict,doc,info,locale,man}
mkdir -pv /usr/{,local/}share/{misc,terminfo,zoneinfo}
mkdir -pv /usr/{,local/}share/man/man{1..8}
mkdir -pv /var/{cache,local,log,mail,opt,spool}
mkdir -pv /var/lib/{color,misc,locate}

ln -sfv /run /var/run
ln -sfv /run/lock /var/lock
```

**Ändere die Berechtigungen:**
```bash
install -dv -m 0750 /root
install -dv -m 1777 /tmp /var/tmp
```
> **<font color="#9bbb59">ERKLÄRUNG</font>:**
> ***`install`*** → *Wird hier verwendet, um Verzeichnisse anzulegen (ähnlich wie `mkdir -p`) und gleichzeitig Berechtigungen zu setzen.*
> ***`-d`*** → Erstellt ein Verzeichnis.
>**`-m 0750`** → *Setzt die Berechtigungen auf `rwxr-x---`: (Sicherstellen, dass nur `root` und die zugehörige Gruppe Zugriff auf `/root` hat.)*
>* **7 (rwx)** → Besitzer (`root`) kann lesen, schreiben, ausführen
>* **5 (r-x)** → Gruppe kann lesen und ausführen
>* **0 (---)** → Andere haben keinen Zugriff
>
>***`-m 1777`*** → *Setzt die Berechtigungen auf `rwxrwxrwt`:*
>	*(Temporäre Verzeichnisse, auf die jeder schreiben kann, aber niemand fremde Dateien löschen kann.)*
> * **7 (rwx)** → Besitzer kann lesen, schreiben, ausführen
> * **7 (rwx)** → Gruppe kann lesen, schreiben, ausführen
> * **7 (rwx)** → Andere können lesen, schreiben, ausführen
> * **t (sticky bit)** → Nur der Eigentümer einer Datei kann sie löschen, auch wenn andere Schreibrechte haben

---
 ## Erstellung von benötigte Dateien und Symlinks

### `/etc/mtab`
- Historisch sicherte Linux eine Liste von eingebundene Dateisysteme in `/etc/mtab`
- Moderne Betriebssysteme → `/proc`
- in LFS → Symlink zum `/proc/self/mounts`
```bash
ln -sv /proc/self/mounts /etc/mtab
```


### `/etc/hosts`
-  Die Datei **`/etc/hosts`** enthält eine lokale Tabelle, die **Hostnamen** (z. B. `localhost`, `myserver`) den **IP-Adressen** (z. B. `127.0.0.1`, `::1`) zuordnet.  
Sie dient also als **lokales Namensauflösungssystem**, bevor ein DNS-Server befragt wird.
- In manchen Testsuites  und in einer Perl-Konfigurationsdatei ist `/etc/hosts` benötigt.
```bash
cat > /etc/hosts << EOF
127.0.0.1 localhost $(hostname)
::1       localhost
EOF
```
> **<font color="#9bbb59">ERKLÄRUNG</font>:**
>**`127.0.0.1`** → *IPv4-Loopback-Adresse (dein eigener Rechner)*
>**`::1`** → *IPv6-Loopback-Adresse*


### Der Benutzer `root` Anmeldungsfähig machen:

Dafür sollte es entsprechende Einträge in `/etc/passwd` und `/etc/group` geben.
```bash
cat > /etc/passwd << "EOF"
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/dev/null:/usr/bin/false
daemon:x:6:6:Daemon User:/dev/null:/usr/bin/false
messagebus:x:18:18:D-Bus Message Daemon User:/run/dbus:/usr/bin/false
uuidd:x:80:80:UUID Generation Daemon User:/dev/null:/usr/bin/false
nobody:x:65534:65534:Unprivileged User:/dev/null:/usr/bin/false
EOF
```
> **<font color="#9bbb59">ERKLÄRUNG</font>:**
>***`/usr/bin/false`*** : *kein Shell*
>***`/dev/null`*** : *kein Home-Verzeichnis*
>
>***Daemon User***: Ein spezieller System-Benutzer, der in Unix/Linux Betriebssysteme, nur für  Hintergrund-Aufgaben zuständig ist.


 **Format der `/etc/passwd`-Datei**
Jede Zeile hat das Format:
`username:password_placeholder:UID:GID:Kommentar:Home-Verzeichnis:Login-Shell`
- **username** → Name des Benutzers
- **password_placeholder** → `x` → Passwort wird in `/etc/shadow` gespeichert
- **UID** → User ID (0 = root, >0 normale oder System-Benutzer)
- **GID** → Group ID
- **Kommentar** → Beschreibung oder vollständiger Name
- **Home-Verzeichnis** → Standardverzeichnis für diesen Benutzer
- **Login-Shell** → Shell beim Einloggen (`/bin/bash` = echte Shell, `/usr/bin/false` = kein Login möglich) 

```bash
cat > /etc/group << "EOF"
root:x:0:
bin:x:1:daemon
sys:x:2:
kmem:x:3:
tape:x:4:
tty:x:5:
daemon:x:6:
floppy:x:7:
disk:x:8:
lp:x:9:
dialout:x:10:
audio:x:11:
video:x:12:
utmp:x:13:
cdrom:x:15:
adm:x:16:
messagebus:x:18:
input:x:24:
mail:x:34:
kvm:x:61:
uuidd:x:80:
wheel:x:97:
users:x:999:
nogroup:x:65534:
EOF
```
> **<font color="#9bbb59">ERKLÄRUNG</font>:**
>***IDs von `root` und `bin`***: *Per **Linux Base System (LSB)** root ist die ID Number `0` und Der Benutzer `bin` ist die ID Nummer `1` gegeben. *
> ***IDs von `tty`***: *Häufig ist die ID Nummer `5` dem Benutzer `tty` gegeben. Zur Einbindung des `devpts` Dateisystem ist auch diesen ID Nummer benutzt.* 
>
>**`65534`**: *Vom Kernel für **unmapped users/groups** (NFS - Network File System, Separate Benutzer Namespaces - Container oder Isolierte Umgebungen, die nicht gemapped sind ) genutzt.*  
*`nobody`/`nogroup` → lokale Zuordnung, kein Login, minimalrechte. Programme sollten sich nicht darauf verlassen.*
 
 **Format der `/etc/group`**
`gruppenname:passwort:GID:Mitglieder`
- **gruppenname** → Name der Gruppe
- **passwort** → normalerweise `x` (echtes Passwort wird in `/etc/gshadow` gespeichert)
- **GID** → Gruppen-ID (numerisch, z. B. 0 für root)
- **Mitglieder** → Komma-getrennte Liste von Benutzern, die zu dieser Gruppe gehören (optional)

- Manche Prüfungen benötigen den Benutzer `tester`

```bash
echo "tester:x:101:101::/home/tester:/bin/bash" >> /etc/passwd
echo "tester:x:101:" >> /etc/group
install -o tester -d /home/tester
```

- Log-Dateien für Anmeldungen
```bash
touch /var/log/{btmp,lastlog,faillog,wtmp}
chgrp -v utmp /var/log/lastlog
chmod -v 664 /var/log/lastlog
chmod -v 600 /var/log/btmp
```
> **<font color="#9bbb59">ERKLÄRUNG</font>:**
>***`utmp`***: *Systemgruppe für **Login-Tracking** (Speichert aktive Sessions, z. B. `who`, `w`).*

>**<font color="#938953">HINWEIS zum Jahr 2038:</font>**
> - Die Logdateien `utmp`, `wtmp`, `btmp` und `lastlog` verwenden **32-Bit-Integer** für Zeitstempel.
> - Diese werden **nach dem Jahr 2038** ("Year 2038 problem") zu einem **fundamentalen Problem** führen, da der maximal darstellbare Wert überschritten wird.
> - Viele Softwarepakete stellen die Nutzung bereits ein oder planen dies. Es wird empfohlen, diese Dateien als **veraltet (deprecated)** zu betrachten.


 **📝 Zusammenfassung der Logdateien**

| **Protokolldatei**     | **Zweck**                                                       | **Inhalt**                                                                                        |
| ---------------------- | --------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| **`/var/log/wtmp`**    | Zeichnet **alle An- und Abmeldungen** auf.                      | Historie aller Benutzer-Logins und Logouts.                                                       |
| **`/var/log/lastlog`** | Speichert, **wann sich jeder Benutzer zuletzt** angemeldet hat. | Letzte erfolgreiche Anmeldezeit jedes Benutzers.                                                  |
| **`/var/log/faillog`** | Zeichnet **fehlgeschlagene Anmeldeversuche** auf.               | Informationen über gescheiterte Logins pro Benutzer (oft zusammen mit `/var/log/btmp` verwendet). |
| **`/var/log/btmp`**    | Zeichnet **fehlerhafte (schlechte) Anmeldeversuche** auf.       | Alle fehlgeschlagenen Login-Versuche.                                                             |
| **`/run/utmp`**        | Speichert Informationen über **aktuell angemeldete** Benutzer.  | Wird **dynamisch** während der Boot-Skripte erstellt.                                             |

