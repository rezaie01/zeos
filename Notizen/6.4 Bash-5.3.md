## 2) Was ist Bash? 

Bash (Bourne Again Shell) ist die Standard-Shell in Linux-Systemen, ein Kommandozeileninterpreter zum Ausführen von Befehlen und Skripten. Im LFS-Kontext wird sie für die temporäre Build-Umgebung kompiliert.

---

## 3) **Vorbereiten zur Kompilierung**

Code (aus dem Buch):

```
./configure --prefix=/usr \
  --build=$(sh support/config.guess) \
  --host=$LFS_TGT \
  --without-bash-malloc
```
> **<font color="#9bbb59">ERKLÄRUNG</font>:**
>*`--build=$(sh support/config.guess)` — bestimmt die Build-Maschine automatisch (Script `config.guess` wird ausgeführt).* 
> 
> *`--without-bash-malloc` — **wichtig:** deaktiviert Bashs eigene `malloc`-Implementierung, da diese bekanntermaßen zu Segfaults führen kann; dadurch nutzt Bash die `malloc`-Funktionen der Glibc, die stabiler sind. (Wortlaut Buch: diese Option schaltet Bashs Memory-Allocation ab, weil sie Segmentation Faults verursachen kann.)*
    

---

## 4) **Kompilieren** 

```
make
```

---

## 5) **Installieren** 

```
make DESTDIR=$LFS install
ln -sv bash $LFS/bin/sh
```
> **<font color="#9bbb59">ERKLÄRUNG</font>:**
>*`ln -sv bash $LFS/bin/sh` — erzeugt einen symbolischen Link `/bin/sh` (im temporären LFS) der auf `bash` zeigt; viele Programme erwarten `/bin/sh`, daher stellt dieser Link die Kompatibilität sicher.*
    