GCC hängt von den Paketen **MPFR**, **GMP** und **MPC** ab. Da sie zusammen mit **GCC** gebaut werden, entpackt man diese Bibliotheken in das Quellverzeichnis von **GCC**.

**GMP** hilft GCC bei genauen Berechnungen mit großen Zahlen.  
**MPFR** sorgt für genaue Kommazahlen.  
**MPC** macht Berechnungen mit **komplexen Zahlen** (z. B. a + bi).

➡️ Sie helfen GCC, Mathe genau und ohne Fehler zu rechnen.

```bash
tar -xf ../mpfr-4.2.2.tar.xz
mv -v mpfr-4.2.2 mpfr
tar -xf ../gmp-6.3.0.tar.xz
mv -v gmp-6.3.0 gmp
tar -xf ../mpc-1.3.1.tar.gz
mv -v mpc-1.3.1 mpc
```

wenn die Maschine x86_64 ist, dann im Quellverzeichnis von `GCC` führt man dieses Kommando aus, um den `GCC` zu sagen, **wo GCC beim Kompilieren nach vorhandenen Bibliotheken suchen soll** und **wo er sie beim Linken erwarten soll**.
```bash
case $(uname -m) in x86_64) sed -e '/m64=/s/lib64/lib/' \ -i.orig gcc/config/i386/t-linux64 ;; esac
```
> **<font color="#9bbb59">ERKLÄRUNG</font>:**
>*`sed`*: *`sed` ist ein **Stream Editor** – ein Tool zum **automatischen Bearbeiten von Textdateien**. [[sed]]*
>
>*`/m64=/` → finde alle Zeilen, die m64= enthalten*
> 
> *`s/lib64/lib/` → ersetze in dieser Zeile das Wort lib64 durch lib*
> 
> *`-i.orig` → ändere die Datei direkt und mache vorher eine Sicherungskopie mit Endung `.orig`*
> 
> `*gcc/config/i386/t-linux64` → das ist die Datei, die geändert wird (im GCC-Quellcodeverzeichnis)*


```bash
../configure \
 --target=$LFS_TGT \
 --prefix=$LFS/tools \
 --with-glibc-version=2.42 \
 --with-sysroot=$LFS \
 --with-newlib \
 --without-headers \
 --enable-default-pie \
 --enable-default-ssp \
 --disable-nls \
 --disable-shared \
 --disable-multilib \
 --disable-threads \
 --disable-libatomic \
 --disable-libgomp \
 --disable-libquadmath \
 --disable-libssp \
 --disable-libvtv \
 --disable-libstdcxx \
 --enable-languages=c,c++
```
> **<font color="#9bbb59">ERKLÄRUNG</font>:**
`*--with-glibc-version=2.42`*: *Diese Option gibt die Version von Glibc an, die auf dem Zielsystem verwendet wird. Sie bezieht sich nicht auf die libc der Host-Distribution, da alles, was vom pass1-GCC kompiliert wird, in der chroot-Umgebung läuft, die von der libc der Host-Distribution isoliert ist.*  
>  
*`--with-newlib`*: *Da noch keine funktionierende C-Bibliothek verfügbar ist, stellt diese Option sicher, dass die Konstante `inhibit_libc` beim Bau von `libgcc` definiert wird. Dies verhindert die Kompilierung von Code, der libc-Unterstützung erfordert.*  
>  
*`--without-headers`*: *Beim Erstellen eines vollständigen Cross-Compilers benötigt GCC standardmäßig Header-Dateien, die mit dem Zielsystem kompatibel sind. Für unsere Zwecke sind diese Header nicht erforderlich. Diese Option verhindert, dass GCC nach ihnen sucht.*  
>  
*`--enable-default-pie` und `--enable-default-ssp`*:  
*Diese Optionen ermöglichen es GCC, Programme standardmäßig mit Sicherheitshärtungsfunktionen zu kompilieren (mehr Informationen dazu im Hinweis zu PIE und SSP in Kapitel 8). Sie sind in dieser Phase nicht zwingend erforderlich, da der Compiler nur temporäre ausführbare Dateien erzeugt. Es ist jedoch sinnvoll, die temporären Pakete so nah wie möglich an den endgültigen Versionen zu halten.*  
>  
*`--disable-shared`*: *Diese Option zwingt GCC dazu, seine internen Bibliotheken statisch zu verlinken. Dies ist notwendig, weil dynamische Bibliotheken `Glibc` erfordern, die auf dem Zielsystem noch nicht installiert ist.*  
>  
*`--disable-multilib`*: *Auf x86_64 unterstützt LFS keine Multilib-Konfiguration. Diese Option ist für x86 harmlos.*  
>  
*`--disable-threads`, `--disable-libatomic`, `--disable-libgomp`, `--disable-libquadmath`, `--disable-libssp`, `--disable-libvtv`, `--disable-libstdcxx`*:  
*Diese Optionen deaktivieren die Unterstützung für Threading, `libatomic`, `libgomp`, `libquadmath`, `libssp`, `libvtv` bzw. die C++-Standardbibliothek. Diese Funktionen können beim Erstellen eines Cross-Compilers fehlschlagen und sind für die Aufgabe, die temporäre libc zu cross-kompilieren, nicht erforderlich.*  
>  
*`--enable-languages=c,c++`*: *Diese Option stellt sicher, dass nur die C- und C++-Compiler gebaut werden. Dies sind die einzigen Sprachen, die derzeit benötigt werden.*


--- 

Nun geht es darum, **einen wichtigen Header** (`limits.h`) richtig zu setzen.  
Dieser Header ist notwendig, damit C-Programme die **Grenzen (Limits)** von Datentypen wie `int`, `char`, usw. kennen.

---

### 🔹 Was sind Header?

Header-Dateien (`*.h`) sind wie **Bibliotheksbeschreibungen**.  
Sie sagen dem Compiler: „Diese Funktionen, Konstanten und Typen gibt es.“

`limits.h` ist z. B. ein Standard-C-Header mit Informationen wie:

```c
#define INT_MAX 2147483647
#define CHAR_BIT 8
```

Diese Werte hängen vom System (z. B. 32-bit, 64-bit) ab.

---

### 🔹 Warum braucht LFS das?

Beim LFS-Bau im **Kapitel 5** existiert dein neues System (Root unter `$LFS`) noch **nicht vollständig**.  
Also gibt es dort auch **noch keine vollständigen System-Header**.  
Aber **GCC** braucht `limits.h`, um Programme zu kompilieren – auch Glibc später.

Darum erzeugt das Buch einen **temporären `limits.h`**, der alle nötigen Werte enthält.

---

### 🔹 Der Befehl im Detail

```bash
cd .. # gehe aus dem Build Ordner heraus.
cat gcc/limitx.h gcc/glimits.h gcc/limity.h > \
  `dirname $($LFS_TGT-gcc -print-libgcc-file-name)`/include/limits.h
```
> **<font color="#9bbb59">ERKLÄRUNG</font>:**
>*`cat gcc/limitx.h gcc/glimits.h gcc/limity.h`: Zeigt den Inhalt dieser drei Dateien hintereinander. Zusammen ergeben sie die komplette Version von `limits.h`.*
>
>*`$($LFS_TGT-gcc -print-libgcc-file-name)`:  Führt den Cross-Compiler (z. B. `x86_64-lfs-linux-gnu-gcc`) aus, um herauszufinden, **wo seine Bibliotheken liegen**.*
   *Beispielausgabe:*  *`/mnt/lfs/tools/lib/gcc/x86_64-lfs-linux-gnu/13.2.0/libgcc.a`*
>
   *`dirname ...`: Entfernt den Dateinamen, sodass nur der Pfad bleibt: `/mnt/lfs/tools/lib/gcc/x86_64-lfs-linux-gnu/13.2.0`*
>
>*`> .../include/limits.h`:  Leitet die Ausgabe von `cat` in eine neue Datei: `/mnt/lfs/tools/lib/gcc/x86_64-lfs-linux-gnu/13.2.0/include/limits.h`*
    

📦 Ergebnis:  
Eine vollständige `limits.h`-Datei wird im Compiler-Verzeichnis gespeichert.  
So kann dein Cross-Compiler korrekt weiterarbeiten.

---

# **Glossar – Wichtige Begriffe einfach erklärt**  

---
## **Compiler & Bibliotheken**  
- **Cross-Compiler**: Erstellt Code für ein anderes System (z. B. für LFS) als das, auf dem er läuft.  
- **Glibc**: Grundlegende C-Bibliothek für Systemfunktionen (z. B. Dateizugriff, Speicherverwaltung).  
- **libgcc**: Interne GCC-Bibliothek für low-Level-Operationen (z. B. Division auf 32-Bit-Systemen).  
- **GMP**: Rechnet exakt mit riesigen Zahlen – wird von GCC für mathematische Operationen genutzt.  
- **MPFR**: Ermöglicht präzise Gleitkommaberechnungen in GCC.  
- **MPC**: Verarbeitet komplexe Zahlen (z. B. *a + bi*) in GCC.  

---

## **Header & Systemdateien**  
- **Header-Dateien** (`*.h`): Enthalten Funktionsdeklarationen und Konstanten (z. B. `limits.h`).  
- **limits.h**: Definiert systemabhängige Grenzwerte wie `INT_MAX` (maximaler Integer-Wert).  

---

## **Sicherheitsfunktionen**  
- **PIE**: Platziert Programme zufällig im Speicher, um Angriffe zu erschweren.  
- **SSP**: Erkennt und blockiert Pufferüberläufe im Stack (Speicherbereich).  

---

## **Werkzeuge & Befehle**  
- **Stream Editor (sed)**: Automatisiert Textänderungen (z. B. Ersetzen von *lib64* durch *lib*).  
- **$LFS_TGT**: Umgebungsvariable für das Zielsystem (z. B. *x86_64-lfs-linux-gnu*).  

---

## **GCC-Konfigurationsoptionen**  
- **--disable-multilib**: Deaktiviert 32-Bit-Unterstützung auf 64-Bit-Systemen.  
- **--disable-shared**: Erzwingt statisches Verlinken (keine dynamischen Bibliotheken).  
- **--enable-default-pie/ssp**: Aktiviert Sicherheitsfeatures (PIE/SSP) standardmäßig.  
- **--enable-languages=c,c++**: Kompiliert nur C/C++-Compiler, um Zeit zu sparen.  
- **--with-glibc-version=2.42**: Legt die Glibc-Version für das Zielsystem fest.  
- **--without-headers**: Unterdrückt die Suche nach System-Headern (noch nicht vorhanden).