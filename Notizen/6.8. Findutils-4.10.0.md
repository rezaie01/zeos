
## 📜 Einleitung

Das **findutils**-Paket enthält Programme zum Suchen von Dateien in einem Dateisystem. Es stellt Werkzeuge bereit, um Verzeichnisbäume zu durchsuchen und eine Datenbank zu erstellen, zu pflegen und zu durchsuchen (oft schneller als die rekursive Suche, aber nur zuverlässig, wenn die Datenbank aktuell ist). Findutils liefert auch das `xargs`-Programm, mit dem ein angegebener Befehl auf jede durch eine Suche ausgewählte Datei angewendet werden kann.

---

## 📝 Zusammenfassung

Dieser Abschnitt beschreibt die Installation des **Findutils**-Pakets im temporären LFS-System. Der Prozess beinhaltet das Konfigurieren des Pakets für die Cross-Kompilierung und die Installation in das LFS-Zielverzeichnis, das Kompilieren der Quellen und schließlich die Installation der kompilierten Dateien.

---

## 🛠️ Installation von Findutils

### Vorbereitung

```
./configure --prefix=/usr \
 --localstatedir=/var/lib/locate \
 --host=$LFS_TGT \
 --build=$(build-aux/config.guess)
```

> **<font color="#9bbb59">ERKLÄRUNG</font>:** * **`./configure`**: Dieses Skript bereitet das Paket für die Kompilierung vor und prüft Systemabhängigkeiten.* * **`--prefix=/usr`**: Legt das Installationsverzeichnis auf `/usr` im LFS-System fest.* * **`--localstatedir=/var/lib/locate`**: Setzt den Speicherort für die `locate`-Datenbank gemäß dem Filesystem Hierarchy Standard (FHS) auf `/var/lib/locate`.* * **`--host=$LFS_TGT`**: Gibt das Zielsystem für die Cross-Kompilierung an (`$LFS_TGT` enthält die Zielarchitektur).*
> 
> - **`--build=$(build-aux/config.guess)`**: Gibt das Bausystem explizit an. Zusammen mit `--host` aktiviert dies den Cross-Kompilierungsmodus.*
>     

---

### Kompilierung

```
make
```

> **<font color="#9bbb59">ERKLÄRUNG</font>:** * **`make`**: Dieser Befehl kompiliert die Software gemäß den Anweisungen in den Makefiles, die durch das `./configure`-Skript erstellt wurden.*

---

### Installation

```
make DESTDIR=$LFS install
```

> **<font color="#9bbb59">ERKLÄRUNG</font>:** * **`make install`**: Dieser Befehl installiert die kompilierten Programme und zugehörigen Dateien.*
> 
> - **`DESTDIR=$LFS`**: Gibt das Staging-Verzeichnis für die Installation an. Die Dateien werden relativ zu `$LFS` (z.B. `/mnt/lfs`) installiert, sodass sie unter `$LFS/usr` und `$LFS/var/lib/locate` landen.*
>     
