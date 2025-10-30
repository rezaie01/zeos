
## Einleitung

Das **File**-Paket enthält ein Dienstprogramm zur Bestimmung des Dateityps einer oder mehrerer angegebener Dateien. Es verwendet verschiedene Tests (Dateisystem-, Magic-Number- und Sprachtests), um den Typ einer Datei zu klassifizieren.

---

## Installation von File

Der `file`-Befehl auf dem Build-Host muss die gleiche Version haben wie der, den wir bauen, um die Signaturdatei zu erstellen. Führen Sie die folgenden Befehle aus, um eine temporäre Kopie des `file`-Befehls zu erstellen:

```
mkdir build
pushd build
 ../configure --disable-bzlib \
 --disable-libseccomp \
 --disable-xzlib \
 --disable-zlib
 make
popd
```

> **<font color="#9bbb59">ERKLÄRUNG</font>:** * **`mkdir build`**: Erstellt ein separates Verzeichnis `build` für die temporäre Kompilierung.* * **`pushd build`**: Wechselt in das `build`-Verzeichnis und legt das aktuelle Verzeichnis auf den Stack.*
> 
> - **`../configure --disable-*`**: Konfiguriert eine temporäre Version des `file`-Programms. Die `--disable-*`-Optionen verhindern, dass versucht wird, Bibliotheken vom Host-System zu verwenden (wie bzlib, libseccomp, xzlib, zlib), die möglicherweise zu Kompilierungsfehlern führen könnten, wenn die entsprechenden Header-Dateien fehlen.* * **`make`**: Kompiliert die temporäre Version des `file`-Programms.* * **`popd`**: Kehrt zum vorherigen Verzeichnis zurück (dem Hauptquellverzeichnis von File).*
> 

### Hilft zum Verstehen:
- **„Signature File“** = Datei mit Datei-Erkennungsmustern (`magic.mgc`).
- **Warum gleiche Version nötig:**  
    Weil der alte `file`-Befehl sonst nicht weiß, wie die neue Signaturdatei erzeugt werden soll → Bau würde fehlschlagen oder falsche Signaturen erzeugen.

---

Bereiten Sie File für die Kompilierung vor:

```
./configure --prefix=/usr --host=$LFS_TGT --build=$(./config.guess)
```

> **<font color="#9bbb59">ERKLÄRUNG</font>:** * **`./configure`**: Konfiguriert das eigentliche File-Paket für die Cross-Kompilierung.* * **`--prefix=/usr`**: Legt das Installationspräfix auf `/usr` im LFS-System fest.* * **`--host=$LFS_TGT`**: Gibt das Zielsystem für die Cross-Kompilierung an.* * **`--build=$(./config.guess)`**: Gibt das Bausystem explizit an und aktiviert den Cross-Kompilierungsmodus.*

---

Kompilieren Sie das Paket:

```
make FILE_COMPILE=$(pwd)/build/src/file
```

> **<font color="#9bbb59">ERKLÄRUNG</font>:** * **`make`**: Startet den Kompilierungsprozess für das File-Paket.* * **`FILE_COMPILE=$(pwd)/build/src/file`**: Weist den Build-Prozess an, die zuvor erstellte temporäre Version des `file`-Programms (`$(pwd)/build/src/file`) zu verwenden, um die Magic-Signaturdatei zu erstellen.*

---

Installieren Sie das Paket:

```
make DESTDIR=$LFS install
```

> **<font color="#9bbb59">ERKLÄRUNG</font>:** * **`make install`**: Installiert die kompilierten Dateien in das LFS-System.* * **`DESTDIR=$LFS`**: Gibt das Staging-Verzeichnis für die Installation an, sodass die Dateien unter `$LFS/usr` installiert werden.*

---

Entfernen Sie die libtool-Archivdatei, da sie für die Cross-Kompilierung schädlich ist:

```
rm -v $LFS/usr/lib/libmagic.la
```

> **<font color="#9bbb59">ERKLÄRUNG</font>:** * **`rm -v $LFS/usr/lib/libmagic.la`**: Löscht die libtool-Archivdatei (`.la`), da sie bei der Cross-Kompilierung Probleme verursachen kann.*
