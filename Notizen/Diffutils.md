
## Was ist Diffutils?

Diffutils ist eine Sammlung von Werkzeugen zum **Vergleichen von Dateien und Verzeichnissen** unter Unix-ähnlichen Systemen. Die Hauptkomponenten sind:

| Tool                                                                        | Funktion                                                           |
| --------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| [diff](https://www.gnu.org/software/diffutils/manual/diffutils.html#diff)   | Zeigt Unterschiede zwischen Textdateien zeilenweise an             |
| [cmp](https://www.gnu.org/software/diffutils/manual/diffutils.html#cmp)     | Vergleicht Dateien byteweise (auch für Binärdateien geeignet)      |
| [sdiff](https://www.gnu.org/software/diffutils/manual/diffutils.html#sdiff) | Zeigt zwei Dateien nebeneinander mit Markierungen für Unterschiede |
| [patch](https://www.gnu.org/software/patch/)                                | Wendet Änderungen aus diff-Ergebnissen auf Originaldateien an      |
Diese Tools sind besonders nützlich für:

- Versionskontrolle von Code
- Konfigurationsmanagement
- Automatisierte Tests
- Nachverfolgung von Änderungen in Dokumenten

---

## Installation von Diffutils

### Vorbereitung

```
./configure --prefix=/usr \
 --host=$LFS_TGT \
 gl_cv_func_strcasecmp_works=y \
 --build=$(./build-aux/config.guess)
```

> **<font color="#9bbb59">ERKLÄRUNG</font>:**
> **`./configure`**: Dieses Skript bereitet das Paket für die Kompilierung vor. Es prüft Abhängigkeiten und Systemmerkmale und erstellt die Makefiles.* * **`--prefix=/usr`**: Legt fest, dass die Software im Verzeichnis `/usr` des LFS-Systems installiert wird.*
>     
> **`--host=$LFS_TGT`**: Gibt das Zielsystem für die Cross-Kompilierung an. `$LFS_TGT` ist eine Variable, die die Zielarchitektur definiert (z.B. `x86_64-lfs-linux-gnu`).*
>     
> **`gl_cv_func_strcasecmp_works=y`**: Gibt das Ergebnis einer Überprüfung für die `strcasecmp`-Funktion an. Diese Überprüfung erfordert das Ausführen eines kompilierten C-Programms, was während der Cross-Kompilierung nicht möglich ist. Wir geben das Ergebnis (`y`, da die Funktion in Glibc-2.42 funktioniert) direkt an, damit die Konfiguration fortgesetzt werden kann.*
>     
> **`--build=$(./build-aux/config.guess)`**: Gibt das Bausystem explizit an. Zusammen mit `--host` aktiviert dies den Cross-Kompilierungsmodus.*
>     

---

### Kompilierung

```
make
```

---

### Installation

```
make DESTDIR=$LFS install
```
