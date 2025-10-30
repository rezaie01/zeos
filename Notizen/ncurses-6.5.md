# Analyse von LFS Kapitel 6.3: Ncurses-6.5-20250809

Dieser Abschnitt behandelt die Kompilierung des `ncurses`-Pakets. `ncurses` ist eine grundlegende Bibliothek, die von vielen Programmen (z.B. der `bash`-Shell) verwendet wird.

## 1. Paketübersicht

- **Ziel des Pakets:** `ncurses` (new curses) ist eine Programmbibliothek für textbasierte Benutzeroberflächen (TUIs), die unabhängig vom Terminaltyp funktionieren. Es bietet Funktionen zur Cursor-Steuerung, Farbänderung, Fenstererstellung und Tastatureingaben.
    
- **Verbindung zu meinem Lernziel:** Als zukünftiger Fachinformatiker für Systemintegration arbeitest du oft auf Servern ohne GUI. Wichtige Tools wie `htop`, `vim` oder `apt` nutzen `ncurses` für ihre textbasierten Oberflächen. Das Verständnis von `ncurses` erklärt, wie diese Admin-Tools funktionieren.
    
- **Hauptfunktionen:**
    
    - Abstraktion der Terminalsteuerung (unabhängig von "xterm", "vt100" etc.).
        
    - Erstellung von Fenstern, Menüs und Formularen im Terminal.
        
    - Verwaltung von Farben und Textattributen (fett, unterstrichen).
        
    - Effiziente Bildschirmaktualisierung (sendet nur die geänderten Teile an das Terminal).
        
- **Abhängigkeiten:** Benötigt `Gawk` (wurde in einem früheren LFS-Schritt kompiliert). Es ist selbst eine wichtige Abhängigkeit für Pakete wie `bash`, `less`, `vim` und viele mehr.
    

## 2. Kompilationsprozess

Der `ncurses`-Kompilationsprozess in LFS-Kapitel 6 ist ein klassisches **Cross-Compiling**-Beispiel mit einem "Henne-Ei-Problem", das wir manuell lösen.

**Das Problem:** Der `ncurses`-Build erfordert das Tool `tic`, um die `terminfo`-Datenbank (Terminal-Fähigkeiten) zu kompilieren. Der Build-Prozess erstellt `tic` für das _LFS-Zielsystem_, kann dieses aber nicht auf dem _Host-System_ ausführen, um die für den weiteren Build nötige Datenbank zu erzeugen.

**Die Lösung (in 2 Phasen):**

1. Wir kompilieren _zuerst nur das `tic`-Tool_ für unser _Host-System_.
  
2. Wir kompilieren _danach das gesamte `ncurses`-Paket_ für das _LFS-Zielsystem_ und teilen dem Konfigurationsskript mit, dass es das Host-Tool und die bereits erstellte Datenbank verwenden soll.
    

### Phase 1: Host-`tic` kompilieren

Wir bauen ein `tic`-Programm, das auf unserem Host-System läuft.

```
../configure --prefix=$LFS/tools AWK=gawk 
make -C include
make -C progs tic
install progs/tic $LFS/tools/bin
```

> **<font color="#9bbb59">ERKLÄRUNG:</font>**
> 
> `make`: Automatisiert Kompilationsprozesse basierend auf einem `Makefile`.
>     
> `-C include`: (Dies beantwortet deine 3. Frage). `-C` (Change Directory) weist `make` an, ins Verzeichnis `include` zu wechseln, um Header-Dateien vorzubereiten.
>     
> `-C progs tic`: `make` wechselt ins `progs`-Verzeichnis und baut nur das `tic`-Ziel, nicht das ganze Paket.
>     
> - **Ergebnis:** Wir haben eine Host-fähige `./progs/tic`-Anwendung. und installieren (`install`: Kopiert Dateien ins Zielverzeichnis) wir sie ins `$LFS/tools/bin` 
>     

---
### Phase 2.1: Vollständige Cross-Kompilierung von Ncurses

Jetzt folgt die Cross-Kompilierung des LFS-Zielpakets.

```
./configure --prefix=/usr \ # ins Quellverziechnis von ncurses
            --host=$LFS_TGT \
            --build=$(./config.guess) \
            --mandir=/usr/share/man \
            --with-manpage-format=normal \
            --with-shared \
            --without-normal \
            --with-cxx-shared \
            --without-debug \
            --without-ada \
            --disable-stripping \
            AWK=gawk
```
> **<font color="#9bbb59">ERKLÄRUNG:</font>** (Dies beantwortet deine 4. Frage)
> 
> `./configure`: Haupt-Konfigurationsskript. Prüft das System, findet Abhängigkeiten und erstellt das `Makefile`.
>     
> `--prefix=/usr`: Basis-Installationsverzeichnis (z.B. `/usr/bin`, `/usr/lib`) relativ zu `$LFS`.
>     
> `--host=$LFS_TGT`: **Wichtigster** Cross-Compile-Schalter. Definiert das Zielsystem (_host_) über die `$LFS_TGT`-Variable (z.B. `x86_64-lfs-linux-gnu`).
>     
> `--build=$(./config.guess)`: Definiert das Build-System (unser Host-Linux), erkannt durch `config.guess`.
>     
> `--with-shared`: Erstellt "shared libraries" (`.so`-Dateien) zur gemeinsamen Nutzung durch Programme.
>     
> `--without-normal`: Deaktiviert die Erstellung von nicht-wide-character Bibliotheken.
>     
> `--disable-stripping`: Deaktiviert das Entfernen von Debug-Symbolen (Stripping); LFS macht dies später global.
>     

```
make DESTDIR=$LFS install
```
> **<font color="#9bbb59">ERKLÄRUNG:</font>**
> 
> `make install`: Führt das "install"-Ziel des `Makefile` aus und kopiert kompilierte Dateien an ihren Bestimmungsort.
>     
> `DESTDIR=$LFS`: **Fundamental** für LFS. `DESTDIR` (Destination Directory) stellt allen Installationspfaden `$LFS` voran (z.B. `/usr/lib` wird zu `$LFS/usr/lib`). Dies verhindert eine Installation auf dem _Host-System_ und leitet alles korrekt in das LFS-Zielsystem.
>     

### Phase 2.2: Nach-Installations-Fixes

Zwei kleine Anpassungen sind nach der Installation nötig. (Beantwortet Q5).

```
ln -sv libncursesw.so $LFS/usr/lib/libncurses.so
```

> **<font color="#9bbb59">ERKLÄRUNG:</font>**
> ***Was passiert hier?** Wir erstellen einen symbolischen Link (Symlink).*
>     
> * `ln -sv`: `ln` erstellt Links. `-s` = symbolisch (Zeiger), `-v` = verbose (gesprächig).*
>     
> * `libncursesw.so`: Die Quelldatei, die echte "w"ide-character-Bibliothek in `$LFS/usr/lib/`.*
>     
> * `$LFS/usr/lib/libncurses.so`: Das Ziel; ein Link namens `libncurses.so`, der auf die `w`-Version zeigt.*
>     
> *-**Warum?** Sorgt für Kompatibilität. Wir haben nur die moderne `w`-Bibliothek (`libncursesw.so`) gebaut. Ältere Programme suchen oft nach der alten (`libncurses.so`). Der Symlink leitet Anfragen von der alten zur neuen Bibliothek um.*
>     

---

```
sed -e 's/^#if.*XOPEN.*$/#if 1/' \
    -i $LFS/usr/include/curses.h
```
> **<font color="#9bbb59">ERKLÄRUNG:</font>**
>  ***Was passiert hier?** Wir bearbeiten die `ncurses`-Header-Datei "im Flug".*
>   
>   *`sed`: Der **S**tream **Ed**itor, ein Werkzeug zur Textmanipulation.*
>     
> *`-e 's/FIND/REPLACE/'`: `-e` übergibt ein Skript, `s` steht für "substitute" (ersetzen).*
>     
>  *`^#if.*XOPEN.*$`: Der FIND-Teil (Regular Expression): Findet jede Zeile, die mit `#if` beginnt und `XOPEN` enthält.*
>     
>  *`#if 1`: Der REPLACE-Teil.*
>     
>  *`-i`: (in-place) Bearbeite die Datei direkt.*
>     
> *`$LFS/usr/include/curses.h`: Die Zieldatei, die wir bearbeiten.*
>     
> ***Warum?** Erzwingt die Aktivierung eines Code-Blocks im C-Präprozessor. Dieser Block deklariert zusätzliche "X/Open Curses"-Standardfunktionen (z.B. `wgetch`), die spätere Pakete (wie `gettext`) benötigen, um Kompilierungsfehler zu vermeiden.*
>     

## 3. Begriffe, Variablen & Konzepte

- **`tic` (Terminal Information Compiler):** (Beantwortet Q2) Übersetzt Terminal-Beschreibungen (Text) in die binäre `terminfo`-Datenbank, die `ncurses` schnell lesen kann.
    
- **`terminfo`:** Datenbank, die `ncurses` Terminal-Steuercodes mitteilt (z.B. `\e[H` = Cursor links oben).
    
- **Cross-Compiling:** Software auf einem System (_Build_, z.B. Host-Linux) kompilieren, die auf einem anderen System (_Host_, z.B. LFS-Ziel) laufen soll.
    
- **`$LFS_TGT`:** Umgebungsvariable mit dem "Triplet" (z.B. `x86_64-lfs-linux-gnu`) des Zielsystems; zentral für das LFS-Cross-Compiling.
    
- **`DESTDIR`:** Standard-`make`-Variable, die dem Installationspfad ein Stammverzeichnis voranstellt (z.B. `$LFS`). Verhindert Installation auf dem Host.
    
- **Shared Library (`.so`):** (Shared Object) Bibliotheksdatei, die zur Laufzeit von mehreren Programmen geladen und genutzt wird.
    
- **Wide-Character (`libncursesw.so`):** Version der Bibliothek, die Multi-Byte-Zeichen (z.B. Umlaute, Emojis) via UTF-8 korrekt verarbeiten kann.
    

## 4. Hinweise, Tipps & Fehlerquellen

> <font color="#938953">HINWEIS:</font>
> Das Henne-Ei-Problem mit tic ist ein klassisches Cross-Compiling-Szenario. Die LFS-Lösung (Host-Tool bauen, Ziel-Datenbank erstellen, Cross-Compile mit Verweis auf Host-Tool) ist ein Muster, das dir bei komplexeren Builds (wie GCC oder Glibc) wieder begegnen wird. Präge dir dieses Konzept gut ein!

> <font color="#938953">HINWEIS:</font>
> Die Unterscheidung zwischen --build (wo du kompilierst) und --host (wofür du kompilierst) ist beim Cross-Compiling von zentraler Bedeutung. Im normalen LFS-Setup (Kapitel 7+) sind build und host identisch, aber hier in Kapitel 6 sind sie verschieden.

> <font color="#c0504d">FEHLERQUELLE:</font>
> Wenn du make -j1 ignorierst und make -j4 (oder eine andere Zahl > 1) verwendest, wird der Build sehr wahrscheinlich mit kryptischen Fehlermeldungen fehlschlagen. Das ncurses-Makefile ist für parallele Builds nicht ausgelegt.

> <font color="#c0504d">FEHLERQUELLE:</font>
> Das Vergessen von DESTDIR=$LFS bei make install ist einer der häufigsten und gefährlichsten Fehler in LFS. Dies würde versuchen, die gerade kompilierte ncurses-Bibliothek in dein Host-System zu installieren (/usr/lib) und könnte dieses instabil machen.
> 
## 5. Zusammenfassung & Verständnis

- **Zusammenfassung:** Wir haben `ncurses` für LFS cross-kompiliert. Die Hürde war das `tic`-Tool: Es wird für das _Ziel_ gebaut, muss aber auf dem _Host_ laufen. Lösung: `tic` für den Host bauen, `terminfo`-Datenbank manuell erstellen und den Haupt-Build mit `ac_cv_...`-Variablen anpassen.
    
- **Verbindung zum LFS-System:** `ncurses` ist das Fundament für interaktive Terminal-Programme. Ohne `ncurses` könnten Pakete wie `bash`, `vim` oder `less` nicht kompiliert werden, da eine wichtige Abhängigkeit fehlt.
    
- **Analogie (Das IKEA-Problem):** Der `ncurses`-Build ist wie ein IKEA-Regal, für dessen Aufbau ein Inbusschlüssel (`tic`) benötigt wird. Das Problem: Der Inbusschlüssel liegt nur in Einzelteilen bei.
    
    - **Phase 1:** Wir bauen den Inbusschlüssel (`tic`) für unseren "Werkzeugkasten" (Host) zusammen.
        
    - **Phase 2:** Wir nutzen den Schlüssel, um die ersten Schrauben (die `terminfo`-Datenbank) anzuziehen.
        
    - **Phase 3:** Wir bauen das Regal (LFS-`ncurses`) fertig und teilen der Anleitung (`configure`) mit, wo der bereits gebaute Schlüssel (`ac_cv_path_...`) liegt.
        

Diese Aufschlüsselung soll dir helfen, die Befehle zu verstehen. Dieser Abschnitt ist ein perfektes Beispiel für die Problemlösung in der Systemintegration.

# meine Frage
1. Was ist `ncurses`?
2. Was ist `tic`? Und warum muss ich es vor Ncurses erstellen?
3. Was bedeutet es, wenn ich `make -C include` oder ein anderes Paket verwende?
4. Was bedeutet die Build-Konfiguration im Detail?
5. Was bedeutet das im Detail? Und wo ist eigentlich `libncursesw.so`? Ich konnte es nicht finden.
```bash
ln -sv libncursesw.so $LFS/usr/lib/libncurses.so
sed -e 's/^#if.*XOPEN.*$/#if 1/' \
-i $LFS/usr/include/curses.h
```