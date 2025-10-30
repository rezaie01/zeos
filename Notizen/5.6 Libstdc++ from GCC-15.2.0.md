## Was ist Libstdc++?
libstdc++ (ausgesprochen "lib standard C++") ist die Standard-C++-Bibliothek des GNU-Projekts. Sie wird hauptsächlich mit dem GCC-Compiler (GNU Compiler Collection) verwendet.

## Vorbereitung des Build-Verzeichnisses



```Bash
mkdir -v build
cd build
```

---

## Konfiguration



```Bash
../libstdc++-v3/configure \
 --host=$LFS_TGT \
 --build=$(../config.guess) \
 --prefix=/usr \
 --disable-multilib \
 --disable-nls \
 --disable-libstdcxx-pch \
 --with-gxx-include-dir=/tools/$LFS_TGT/include/c++/15.2.0
```

> **<font color="#9bbb59">ERKLÄRUNG:</font>**
> 
> - `../libstdc++-v3/configure`: Startet das Konfigurationsskript für die Libstdc++-Komponente, die sich im Unterverzeichnis `libstdc++-v3` relativ zum aktuellen `build`-Verzeichnis befindet.
>     
> - `\` : Dient nur als Zeilenumbruch, damit der Befehl übersichtlich bleibt.
>     
> - `--host=$LFS_TGT`: **Sehr wichtig** in einem Cross-Compiling-Szenario. Es gibt das Zielsystem an, auf dem die kompilierte Bibliothek laufen soll (das LFS-System). `$LFS_TGT` ist die Ziel-Triple-Architektur (z. B. `x86_64-lfs-linux-gnu`).
>     
> - `--build=$(../config.guess)`: Gibt das Hostsystem an, auf dem die Kompilierung durchgeführt wird (Ihr aktuelles Host-Linux). `$(../config.guess)` führt das `config.guess`-Skript aus, um die Architektur des Bau-Systems zu erraten.
>     
> - `--prefix=/usr`: Gibt das endgültige Installationspräfix auf dem Zielsystem an. Obwohl die Installation im Moment nach `$LFS/usr` erfolgt, wird es später im laufenden LFS-System als `/usr` erscheinen.
>     
> - `--disable-multilib`: Deaktiviert die Unterstützung für die Erstellung von Bibliotheken für mehrere ABI-Ziele/Architekturen (z. B. 32-Bit und 64-Bit), was für LFS nicht erforderlich ist.
>     
> - `--disable-nls`: Deaktiviert Native Language Support (NLS), d.h. die Unterstützung für übersetzte Meldungen. Dies reduziert die Größe und Komplexität und ist an dieser Stelle nicht erforderlich.
>     
> - `--disable-libstdcxx-pch`: Verhindert die Installation von präkompilierten Header-Dateien (Precompiled Headers - PCH), die in dieser Phase nicht benötigt werden.
>     
> - `--with-gxx-include-dir=/tools/$LFS_TGT/include/c++/15.2.0`: Legt das Installationsverzeichnis für die C++-Header-Dateien fest. Da wir cross-kompilieren, muss der Pfad explizit angegeben werden, damit er mit dem Suchpfad des Cross-C++-Compilers (`$LFS_TGT-g++`) übereinstimmt, der später die Header findet.
>     

---

### Kompilierung und Installation

```bash
make
```

```bash
make DESTDIR=$LFS install
```

---

## Aufräumen



```Bash
rm -v $LFS/usr/lib/lib{stdc++{,exp,fs},supc++}.la
```
> **<font color="#9bbb59">ERKLÄRUNG:</font>**
> - `$LFS/usr/lib/lib{...}.la`: Löscht die `libtool archive` (_`.la`_) Dateien für die installierten C++-Bibliotheken (`libstdc++`, `libstdcexp`, `libstdc++fs`, `libsupc++`).
>     
> - **Grund:** Diese `.la`-Dateien enthalten oft hartkodierte Pfade des **Bau**-Systems, die im Cross-Compiling-Szenario zu Problemen führen würden, wenn sie später im LFS-System verwendet werden. Sie werden für die weitere Cross-Kompilierung nicht benötigt und werden entfernt, um eine Beschädigung des Toolchains zu verhindern.