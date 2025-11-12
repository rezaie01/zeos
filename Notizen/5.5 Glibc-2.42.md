#glibc #linux-system #c-library #compilation-process #system-integration #FHS #LSB #dynamic-linker #cross-compiling #lfs #system-administration #kernel-interface #package-management
## Paketübersicht: Glibc-2.42

- **Ziel des Pakets:** Glibc steht für **GNU C Library**. Es ist die _zentrale C-Bibliothek_ in fast jedem Linux-System (und vielen anderen Unix-artigen Systemen). Sie implementiert die grundlegenden Funktionen, die C-Programme benötigen, um mit dem Betriebssystemkern (Linux) zu interagieren. Ohne Glibc können die meisten Programme unter Linux nicht laufen. Sie ist essenziell für das Starten des Systems und das Ausführen fast aller User-Space-Anwendungen.
    
- **Verbindung zu meinem Lernziel:** Als angehender Fachinformatiker für Systemintegration ist das Verständnis von Glibc fundamental. Du lernst hier, wie Standardprogramme Systemaufrufe (System Calls) an den Kernel senden (z.B. Dateien öffnen, Speicher anfordern). Dieses Wissen hilft beim Debugging, beim Verstehen von Abhängigkeiten (wie `ldd` sie anzeigt) und bei der Systemadministration im Allgemeinen. Es ist das Bindeglied zwischen Anwendungen und dem Kernel.
    
- **Hauptfunktionen:**
    - Implementierung der Standard-C-Bibliotheksfunktionen (wie `printf`, `malloc`, `strcpy`, etc.).
    - Bereitstellung von Wrappern (Schnittstellen) für Linux-Systemaufrufe.
    - Unterstützung für Internationalisierung (i18n) und Lokalisierung (l10n).
    - Name Service Switch (NSS) für die Konfiguration, wie Systemdatenbanken (Benutzer, Gruppen, Hostnamen) nachgeschlagen werden.
    - Dynamischer Lader (`ld.so`), der Shared Libraries (gemeinsam genutzte Bibliotheken) lädt.
        
- **Abhängigkeiten:**
    - Benötigt: Linux API Headers (aus Schritt 5.4 3), Binutils (Pass 1, aus Schritt 5.2 4), GCC (Pass 1, aus Schritt 5.3 5).
    - Wird benötigt von: Fast allen anderen Paketen im LFS-System.
        

---

##  Kompilationsprozess: Glibc-2.42.

---
### Schritt 1: Symbolische Links für LSB erstellen

```bash
case $(uname -m) in
 i?86) ln -sfv ld-linux.so.2 $LFS/lib/ld-lsb.so.3
 ;;
 x86_64) ln -sfv ../lib/ld-linux-x86-64.so.2 $LFS/lib64
 ln -sfv ../lib/ld-linux-x86-64.so.2 $LFS/lib64/ld-lsb-x86-64.so.3
 ;;
esac
```
> **<font color="#9bbb59">ERKLÄRUNG:</font>**
*`case $(uname -m) in ... esac`: Dies ist eine Shell-Struktur, die Befehle basierend auf der Systemarchitektur ausführt. `uname -m` gibt die Architektur aus (z.B. `i686` oder `x86_64`).*
>
*`i?86)`: Führt den folgenden Befehl aus, wenn die Architektur `i386`, `i486`, `i586` oder `i686` ist (32-Bit x86).*
>
*`ln -sfv ld-linux.so.2 $LFS/lib/ld-lsb.so.3`: Erstellt einen symbolischen Link (`ln -s`) namens `ld-lsb.so.3` im Verzeichnis `$LFS/lib`. Er zeigt auf (`->`) `ld-linux.so.2`. `-f` erzwingt das Überschreiben, falls der Link existiert, `-v` zeigt an, was getan wird. Dies ist für die **LSB-Kompatibilität** (Linux Standard Base) erforderlich. Der LSB-Standard legt fest, wo bestimmte Dateien zu finden sein sollten, um die Kompatibilität zwischen verschiedenen Linux-Distributionen zu verbessern.*
>
*`ln -sfv ../lib/ld-linux-x86-64.so.2 $LFS/lib64`: Erstellt einen Link im `$LFS/lib64`-Verzeichnis, der auf den eigentlichen dynamischen Lader (`ld-linux-x86-64.so.2`) im `$LFS/lib`-Verzeichnis zeigt. Dies ist eine Glibc-interne Notwendigkeit für 64-Bit-Systeme.*
>
*`ln -sfv ../lib/ld-linux-x86-64.so.2 $LFS/lib64/ld-lsb-x86-64.so.3`:  Erstellt den LSB-Kompatibilitätslink für 64-Bit-Systeme, analog zur 32-Bit-Version.*

---
### Schritt 2: FHS-Patch anwenden
#### **Warum dieser Patch?** 
Er modifiziert Glibc-Quelldateien, damit Programme, die Laufzeitdaten speichern, dies in FHS-konformen Verzeichnissen tun (z.B. unter `/var/lib` statt `/var/db`). Der **Filesystem Hierarchy Standard (FHS)** definiert, wo welche Art von Dateien im System abgelegt werden sollen.

```bash
patch -Np1 -i ../glibc-2.42-fhs-1.patch
```
> **<font color="#9bbb59">ERKLÄRUNG:</font>**
> *`patch`: Das `patch`-Kommando wendet Änderungen, die in einer Patch-Datei beschrieben sind, auf Quelldateien an.*
> 
> *`-Np1`: `-N` stellt sicher, dass Patches, die vielleicht schon angewendet wurden, ignoriert werden. `-p1` weist `patch` an, die erste Verzeichnisebene aus den Pfadangaben in der Patch-Datei zu entfernen (z.B. wird `a/configure.ac` zu `configure.ac`). Dies ist eine sehr gängige Option.*
> 
> *`-i ../glibc-2.42-fhs-1.patch`: Gibt die Patch-Datei an (`-i` für Input), die angewendet werden soll. Sie befindet sich im übergeordneten Verzeichnis (`../`), dem `sources`-Verzeichnis.*

---

### Schritt 4: `configparms`-Datei erstellen

```
echo "rootsbindir=/usr/sbin" > configparms
```
> **<font color="#9bbb59">ERKLÄRUNG:</font>**
> *`echo "rootsbindir=/usr/sbin"`: Gibt den Text `rootsbindir=/usr/sbin` auf der Standardausgabe aus.*
> 
> *`>`: Dies ist der **Umleitungsoperator**. Er leitet die Standardausgabe des `echo`-Befehls in die Datei `configparms` um. Wenn die Datei existiert, wird sie überschrieben; wenn nicht, wird sie erstellt.*
> 
> *`configparms`: Der Dateiname. Das Glibc `configure`-Skript sucht speziell nach dieser Datei.*
> 
> ***Warum dieser Inhalt?** Die Zeile `rootsbindir=/usr/sbin` weist Glibc an, Programme, die nur von `root` ausgeführt werden sollten (wie `ldconfig`), im Verzeichnis `/usr/sbin` zu installieren, was dem FHS entspricht.*

---
### Schritt 5: Konfiguration ausführen

```bash
../configure \
 --prefix=/usr \
 --host=$LFS_TGT \
 --build=$(../scripts/config.guess) \
 --disable-nscd \
 libc_cv_slibdir=/usr/lib \
 --enable-kernel=5.4
```
> <font color="#9bbb59">ERKLÄRUNG:</font>
> *`--prefix=/usr`: Gibt an, wo die Software im Zielsystem installiert werden soll. Alle Dateien werden relativ zu diesem Präfix installiert (z.B. Bibliotheken in `/usr/lib`, Header in `/usr/include`).*
> 
> *`--build=$(../scripts/config.guess)`: Teilt `configure` mit, auf welchem System der Build-Prozess stattfindet. Wir führen das Skript `config.guess` aus (in `../scripts/`), um den Systemtyp automatisch zu ermitteln (z.B. `x86_64-pc-linux-gnu`). Das ist wichtig, um den Cross-Compile-Modus korrekt zu aktivieren.*
> 
> *`--disable-nscd`: Deaktiviert den Bau des **Name Service Cache Daemon (nscd)**. Dieser Daemon wird in modernen Systemen oft nicht mehr benötigt oder durch andere Mechanismen ersetzt.*
> 
> *`libc_cv_slibdir=/usr/lib`: Dies ist eine Variable, die direkt an `configure` übergeben wird. `libc_cv_` deutet auf eine "cached value" hin. Sie überschreibt die automatische Erkennung des Standardverzeichnisses für Shared Libraries (`slibdir`) und zwingt es, `/usr/lib` zu sein, auch auf 64-Bit-Systemen, wo sonst vielleicht `/usr/lib64` gewählt würde. LFS verwendet `/usr/lib` einheitlich.*
> 
> *`--enable-kernel=5.4`: Informiert Glibc über die minimale Kernel-Version, die unterstützt werden muss. LFS 12.4 verwendet Kernel 5.4 als Basislinie, auch wenn der finale Kernel neuer sein wird. Dies stellt sicher, dass Glibc keine Funktionen verwendet, die nur in neueren Kerneln verfügbar sind, aber aktiviert Optimierungen oder Workarounds, die für diese Kernel-Version relevant sind.*

---
### Schritt 6: Kompilieren


```bash
make
```
---
### Schritt 7: Installieren

```
make DESTDIR=$LFS install
```
> **<font color="#9bbb59">ERKLÄRUNG:</font>**
> *`DESTDIR=$LFS`: **Dies ist wieder entscheidend!** Es weist `make install` an, die Dateien nicht direkt in das `--prefix`-Verzeichnis (also `/usr` auf unserem Host-System!) zu installieren, sondern in ein **Staging-Verzeichnis**. Die Variable `$LFS` (z.B. `/mnt/lfs`) wird als temporäres Wurzelverzeichnis (`/`) verwendet. Eine Datei, die für `/usr/lib` bestimmt ist, wird also nach `$LFS/usr/lib` kopiert. Dies verhindert, dass wir unser laufendes Host-System überschreiben.*

### Schritt 8: `ldd`-Skript anpassen

```bash
sed '/RTLDLIST=/s@/usr@@g' -i $LFS/usr/bin/ldd
```
> **<font color="#9bbb59">ERKLÄRUNG:</font>**
> *`sed`: Der Stream-Editor `sed` wird verwendet, um Textdateien zu bearbeiten.*
> 
> *`'/RTLDLIST=/s@/usr@@g'`: Dies ist der `sed`-Befehl: Suche (`/RTLDLIST=/`) nach der Zeile, die `RTLDLIST=` enthält. Führe dann eine Ersetzung (`s`) aus: Ersetze (`s@...`) das erste Vorkommen von `/usr` (`@/usr@`) durch nichts (`@@`). Mache das global (`g`) in dieser Zeile (obwohl hier wahrscheinlich nur ein `/usr` vorkommt). Verwendet der Befehl `@` statt `/`.*
> 
> *`-i $LFS/usr/bin/ldd`: `-i` bedeutet "inplace", die Änderungen werden direkt in der Datei `$LFS/usr/bin/ldd` gespeichert.*
> 
> *Warum diese Anpassung? Das `ldd`-Skript enthält einen hartkodierten Pfad zum dynamischen Lader, der `/usr` enthält. Da wir im Chroot-Umfeld arbeiten werden, wo `$LFS` zu `/` wird, muss dieser absolute Pfad entfernt werden, damit der Lader korrekt gefunden wird.*

### Schritt 9: Sanity Checks (Überprüfung der Toolchain)

Das Buch führt hier mehrere Befehle aus, um sicherzustellen, dass der Cross-Compiler (`$LFS_TGT-gcc`) und der Linker (`$LFS_TGT-ld`) korrekt konfiguriert sind und die gerade installierte Glibc im `$LFS`-Verzeichnis verwenden.

#### **Wichtigkeit**: 
*Wenn diese Überprüfungen fehlschlagen, stimmt etwas Grundlegendes mit der Cross-Toolchain nicht, und man sollte die vorherigen Schritte überprüfen, bevor man weitermacht.}*

```bash
echo 'int main(){}' | $LFS_TGT-gcc -x c - -v -Wl,--verbose &> dummy.log
readelf -l a.out | grep ': /lib'
grep -E -o "$LFS/lib.*/S?crt[1in].*succeeded" dummy.log
grep -B3 "^ $LFS/usr/include" dummy.log
grep 'SEARCH.*/usr/lib' dummy.log |sed 's|; |\n|g'
grep "/lib.*/libc.so.6 " dummy.log
grep found dummy.log
rm -v a.out dummy.log
```
> **<font color="#9bbb59">ERKLÄRUNG:</font>**
> *Ziel der Checks: Diese Befehle kompilieren ein minimales C-Programm und analysieren die sehr ausführliche Ausgabe (`-v` für GCC, `-Wl,--verbose` für den Linker), um zu überprüfen, ob die Werkzeuge die richtigen Pfade innerhalb von `$LFS` verwenden.*
> 
> *`echo '...' | $LFS_TGT-gcc ... &> dummy.log`: Kompiliert ein einfaches C-Programm aus der Standardeingabe (`-x c -`) und leitet die gesamte Ausgabe (Standardausgabe und Standardfehlerausgabe) in die Datei `dummy.log` um.*
> 
> *`readelf -l a.out | grep ': /lib'`: Prüft, welchen **Programminterpreter** (dynamischen Lader) die kompilierte Datei `a.out` anfordert. Es sollte der Lader innerhalb von `$LFS` sein (z.B. `/lib64/ld-linux-x86-64.so.2`), ohne `$LFS` am Anfang des Pfades.*
> 
> *`grep ... "$LFS/lib.*/S?crt[1in].*succeeded" ...`: Überprüft, ob der Linker die korrekten **Startdateien** (`crt1.o`, `crti.o`, `crtn.o`) aus dem `$LFS/usr/lib`-Verzeichnis gefunden und verwendet hat.*
> 
> *`grep -B3 "^ $LFS/usr/include" ...`: Überprüft, ob der Compiler (`gcc`) nach **Header-Dateien** im korrekten Verzeichnis (`$LFS/usr/include`) sucht.*
> 
> *`grep 'SEARCH.*/usr/lib' ... | sed ...`: Überprüft die **Suchpfade des Linkers**. Er sollte die Bibliotheksverzeichnisse innerhalb von `$LFS` (`/usr/lib`, etc.) durchsuchen.*
> 
> *`grep "/lib.*/libc.so.6 " ...`: Überprüft, ob der Linker die korrekte **C-Bibliothek**  (`libc.so.6`) aus dem `$LFS/usr/lib`-Verzeichnis gefunden hat.*
> 
> *`grep found dummy.log`: Überprüft, ob der Compiler den korrekten **dynamischen Lader** (`ld-linux-*.so.2`) im `$LFS/usr/lib`-Verzeichnis gefunden hat.*
> 
> *`rm -v a.out dummy.log`: Löscht die temporären Dateien.*


---

## 3. Begriffe, Variablen & Konzepte

- **Glibc:** Siehe Paketübersicht. Die zentrale C-Bibliothek.
    
- **Cross-Compiling:** Der Prozess, Software auf einem System (Build-System, z.B. deine vorhandene Linux-Distribution) zu kompilieren, die für ein _anderes_ System (Host/Target-System, z.B. unser LFS-System) bestimmt ist. Dies ist notwendig, weil wir noch kein lauffähiges LFS-System haben, auf dem wir kompilieren könnten.
    
- **$LFS:** Eine **Umgebungsvariable**, die auf das Einhängepunkt (Mount Point) der LFS-Partition zeigt (z.B. `/mnt/lfs`). Sie dient als Wurzelverzeichnis (`/`) für das im Aufbau befindliche LFS-System.
    
    - _Bedeutung:_ Basisverzeichnis für das LFS-System.
        
    - _Verwendung:_ Wird in Pfadangaben verwendet, um sicherzustellen, dass Dateien im LFS-System und nicht im Host-System landen (`DESTDIR=$LFS`).
        
    - _Fehlt/Falsch:_ Wenn nicht gesetzt oder falsch, könnten Befehle versuchen, in das Host-System zu schreiben oder Dateien nicht zu finden, was zu Fehlern oder zur Zerstörung des Host-Systems führen kann.
        
- **$LFS_TGT:** Eine **Umgebungsvariable**, die das **Ziel-Triplet** (Target Triplet) für unsere Cross-Toolchain definiert (z.B. `x86_64-lfs-linux-gnu`). Ein Triplet beschreibt die CPU-Architektur, den Hersteller und das Betriebssystem.
    
    - _Bedeutung:_ Identifiziert das Zielsystem, für das die Cross-Werkzeuge (Compiler, Linker) Code erzeugen sollen.
        
    - _Verwendung:_ Wird an `configure`-Skripte übergeben (`--host=$LFS_TGT`), damit diese die richtigen Cross-Werkzeuge auswählen.
        
    - _Fehlt/Falsch:_ `configure` würde nicht wissen, dass es cross-kompilieren soll, oder würde die falschen Werkzeuge verwenden, was zu Build-Fehlern führt.
        
- **`configure`-Skript:** Ein Shell-Skript, das Teil vieler Quellcode-Pakete ist (oft generiert durch Autoconf). Es prüft das System auf vorhandene Werkzeuge, Bibliotheken und Features und erstellt daraus `Makefiles`, die an das spezifische System angepasst sind.
    
- **`Makefile`:** Eine Datei, die Anweisungen für das `make`-Programm enthält, wie die Software kompiliert und installiert werden soll.
    
- **`DESTDIR`:** Eine Standardvariable, die von `make install` verwendet wird. Sie stellt einen Pfad vor alle Installationsziele. Wenn `--prefix=/usr` und `DESTDIR=/mnt/lfs` gesetzt sind, wird eine Datei, die nach `/usr/bin/programm` installiert werden soll, tatsächlich nach `/mnt/lfs/usr/bin/programm` kopiert.
    
    - _Bedeutung:_ Staging-Verzeichnis für die Installation.
        
    - _Verwendung:_ Essentiell beim Bau eines neuen Systems oder bei der Paketerstellung, um das laufende System nicht zu verändern.
        
    - _Fehlt/Falsch:_ Wenn `DESTDIR` fehlt, würde `make install` versuchen, die Dateien direkt in die vom `--prefix` angegebenen Pfade zu installieren (z.B. `/usr`), was das Host-System überschreiben und zerstören würde!
        
- **FHS (Filesystem Hierarchy Standard):** Ein Standard, der die Verzeichnisstruktur und die Platzierung von Dateien in Linux-Systemen festlegt (z.B. `/bin`, `/usr/bin`, `/etc`, `/var`). Der Patch `glibc-2.42-fhs-1.patch` und die `configparms`-Datei helfen, Glibc FHS-konform zu machen.
    
- **Dynamischer Lader / Linker (ld.so):** Ein Teil von Glibc, der beim Start eines Programms dafür sorgt, dass alle benötigten Shared Libraries (wie `libc.so.6` selbst) gefunden, in den Speicher geladen und korrekt verlinkt werden. Die symbolischen Links (`ld-lsb.so.3` etc.) stellen sicher, dass Programme ihn unter standardisierten Namen finden können.
    
- **LSB (Linux Standard Base):** Ein Satz von Standards, der die Kompatibilität zwischen verschiedenen Linux-Distributionen verbessern soll. Die LSB-Links für `ld.so` sind Teil dieser Bemühungen.
    
- **Sanity Checks:** Überprüfungen nach einem wichtigen Schritt (wie der Glibc-Installation), um sicherzustellen, dass die Konfiguration korrekt ist und die nächsten Schritte wahrscheinlich erfolgreich sein werden.
    

---

## 4. Hinweise, Tipps & Fehlerquellen

> <font color="#938953">HINWEIS:</font>
> Der Unterschied zwischen `--prefix` und `DESTDIR` ist fundamental. `--prefix` definiert den *logischen* Pfad innerhalb des Zielsystems, während `DESTDIR` ein *temporäres* Verzeichnis ist, in das diese Struktur physisch kopiert wird. Verwechsle sie nicht!

> <font color="#938953">HINWEIS:</font>
> Wir kompilieren Glibc hier *immer noch* mit dem Cross-Compiler (`$LFS_TGT-gcc`). Das Ergebnis ist die C-Bibliothek, die später von *allen* Programmen im LFS-System verwendet wird. Es ist quasi das Herzstück des User-Space.

> <font color="#938953">HINWEIS:</font>
> Die Sanity Checks am Ende sind sehr wichtig. Wenn hier etwas schiefgeht, ist es viel einfacher, den Fehler jetzt zu finden und zu beheben, als später, wenn Dutzende weitere Pakete darauf aufgebaut wurden.

> <font color="#c0504d">FEHLERQUELLE:</font>
> Vergessen, ins `build`-Verzeichnis zu wechseln (`cd build`). Das Kompilieren direkt im Quellverzeichnis wird nicht empfohlen und kann zu Problemen führen.

> <font color="#c0504d">FEHLERQUELLE:</font>
> Die Umgebungsvariablen `$LFS` oder `$LFS_TGT` sind nicht gesetzt oder nicht korrekt exportiert (z.B. nach einem `su`). Das `configure`-Skript wird dann fehlschlagen, oft mit Meldungen wie "cannot find compiler" oder ähnlichem. Überprüfe immer mit `echo $LFS` und `echo $LFS_TGT`.

> <font color="#c0504d">FEHLERQUELLE:</font>
> Die Linux API Headers aus Schritt 5.4 wurden nicht korrekt nach `$LFS/usr/include` kopiert. Das `configure`-Skript von Glibc wird dann mit einer Fehlermeldung abbrechen, dass es die Kernel-Header nicht finden kann.

> <font color="#c0504d">FEHLERQUELLE:</font>
> Fehlende Abhängigkeiten auf dem *Host*-System (obwohl LFS versucht, unabhängig zu sein, werden in den frühen Phasen noch Host-Werkzeuge wie `patch`, `make` etc. benötigt). Stelle sicher, dass dein Host-System die in Kapitel 2 genannten Voraussetzungen erfüllt.


## 5. Zusammenfassung & Verständnis

- **Kurze Zusammenfassung:** Wir haben die GNU C Library (Glibc), die grundlegende Bibliothek für fast alle Programme, erfolgreich mit unserem Cross-Compiler gebaut. Anschließend haben wir sie mittels `DESTDIR` sicher in unser LFS-Zielverzeichnis (`$LFS`) installiert. Abschließende Tests haben bestätigt, dass unsere Cross-Toolchain korrekt konfiguriert ist, um diese Glibc zu verwenden.
    
- **Verbindung zum größeren Linux-System:** Glibc ist das Fundament des User-Space. Alle Programme, die wir in den nächsten Kapiteln bauen (Shell, Core-Utilities, etc.), werden gegen diese Glibc gelinkt. Ohne sie könnte kein einziges dieser Programme gestartet werden. Wir haben soeben die zentrale Schnittstelle zwischen Anwendungen und dem Linux-Kernel für unser LFS-System geschaffen.
    
- **Analogie:** 💡 Stell dir den Linux-Kernel als den Motor eines Autos vor. Glibc ist dann das Armaturenbrett, das Lenkrad, die Pedale und die Schaltung – die standardisierte Schnittstelle, die jeder Fahrer (Anwendung) benutzt, um mit dem Motor zu interagieren und das Auto zu steuern. Wir haben gerade dieses gesamte Kontrollsystem für unser LFS-Auto gebaut.
