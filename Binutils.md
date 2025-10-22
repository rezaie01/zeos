#binutils 
Sammlung von Programmen zum **Erstellen, Manipulieren und Analysieren von Binärdateien** (Objektdateien, Libraries, Executables).
## Kern-Tools:
- `as` → **Assembler**: wandelt Assembly-Code in Maschinencode um.
- `ld` → **Linker**: verbindet Objektdateien zu ausführbaren Programmen oder Bibliotheken.
- `objdump`, `nm`, `ar`, `ranlib` → **Analyse & Verwaltung** von Binärdateien und Archiven.
## **LFS-Relevanz:** Ohne Binutils können **Compiler wie GCC keine Programme bauen**, da Objekte nicht erzeugt oder verlinkt werden können.

---

# Bauen von `binutils`

```bash
mkdir -v build # in $LFS/sources/binutils.... Ordner
cd build
```

```bash
../configure --prefix=$LFS/tools \
 --with-sysroot=$LFS \
 --target=$LFS_TGT \
 --disable-nls \
 --enable-gprofng=no \ 
 --disable-werror\ 
 --enable-new-dtags \ 
 --enable-default-hash-style=gnu
```
> **ERKLÄRUNG**:
 *`--prefix=$LFS/tools` : Bestimmt das Installationsverzeichnis für Binutils. Hiermit
 installiert `make install` alle Binutils-Programme in `$LFS/tools/bin` . Damit bleiben sie vom späteren System getrennt.*
 >
 *`--with-sysroot=$LFS` : Teilt dem Build-System mit, dass die Zielsystembibliotheken (z.B.
 für Header-Suche) im Verzeichnis `$LFS` liegen sollen . Dies ist wichtig für Cross
Kompilierung, damit das System nach Abhängigkeiten im neuen LFS-System sucht.*
>
*`--target=$LFS_TGT` : Gibt an, dass Binutils einen Linker/Assembler für das Zielsystem
 `$LFS_TGT` bauen soll. Da `$LFS_TGT` (z.B. x86_64-lfs-linux-gnu ) vom von
 config.guess ermittelten Namen abweichen kann, passt dieser Schalter das Build-System
 an die Zielumgebung an .*
 >
 *`--disable-nls` : Deaktiviert die Unterstützung  für „Native Language Support“  Internationalisierung) in Binutils . Für die temporären Tools braucht man keine
 Übersetzungen, das spart Build-Aufwand.*
 >
 *`--enable-gprofng=no` : Verhindert den Bau des neuen Profilers „gprofng“ . Auch dieser
 wird für den temporären LFS-Compiler nicht benötigt.*
 >
 *`--disable-werror` : Setzt den Schalter, der normalerweise alle Warnungen des Compilers zu Fehlern macht, zurück. So bricht der Build nicht ab, wenn Warnungen (vom Host-Compiler) auftreten .*
 >
 *`--enable-new-dtags` : Erzwingt, dass der Linker moderne „runpath“-Tags für dynamische  Bibliothekspfade verwendet. Dies erleichtert das Debuggen dynamisch gelinkter Programme und umgeht Probleme in manchen Test-Suiten .*
 >
 *`--enable-default-hash-style=gnu` : Schaltet den Linker so, dass er standardmäßig
 nur den GNU-Hash-Tabellen-Stil für Shared Libraries erzeugt . Der klassische ELF-Hash wird weggelassen, da der Glibc-Lader auf LFS ohnehin nur GNU-Hashes verwendet.*

```bash
make
make install
```
> **ERKLÄRUNG**:
 *`make` : Kompiliert das Binutils-Paket. Es erzeugt u.a. Programme wie `ld` (Linker), (Assembler) und andere. Dabei kann es Warnungen und Fehlermeldungen geben; dank as `--disable-werror` stoppen Warnungen aber nicht mehr den Build. Bei Fehlern hilft es oft,  den Build-Log zu prüfen.*
>
 *`make install` : Installiert Binutils in das zuvor angegebene Prefix ( `$LFS/tools` ). Danach  liegen alle ausführbaren Binutils-Programme in `$LFS/tools/bin` und können genutzt  werden


Um die Bauzeit des gesamten Schritts in Standard Build Units (SBUs) zu messen. 1 SBU entspricht der Zeit für den ersten Binutils-Build (ca. 1 SBU auf dem eigenen System )

```bash
time { ../configure ... && make && make install; }
```


