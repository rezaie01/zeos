**Details zum Paket:**
- **Geschätzte Bauzeit:** 0.6 SBU
- **Erforderlicher Speicherplatz:** 295 MB
### 7.9.1. Installation von Perl

1. Vorbereitung zur Kompilierung:

Es wird der Configure-Befehl verwendet, um Perl vorzubereiten:


```bash
sh Configure -des \
    -D prefix=/usr \
    -D vendorprefix=/usr \
    -D useshrplib \
    -D privlib=/usr/lib/perl5/5.42/core_perl \
    -D archlib=/usr/lib/perl5/5.42/core_perl \
    -D sitelib=/usr/lib/perl5/5.42/site_perl \
    -D sitearch=/usr/lib/perl5/5.42/site_perl \
    -D vendorlib=/usr/lib/perl5/5.42/vendor_perl \
    -D vendorarch=/usr/lib/perl5/5.42/vendor_perl
```
**Bedeutung der Configure-Optionen:**
- `-des`: Eine Kombination aus drei Optionen:
     - `-d`: Verwendet Standardwerte für alle Elemente.
    - `-e`: Stellt die Erledigung aller Aufgaben sicher.
    - `-s`: Unterdrückt nicht-essentielle Ausgabe.
- `-D vendorprefix=/usr`: Stellt sicher, dass Perl weiß, wohin es seine Perl-Module installieren soll.
- `-D useshrplib`: Erlaubt die Erstellung von Perl-Modulen als freigegebene Bibliothek (`shared library`) anstelle einer statischen Bibliothek (`static library`).
- `-D privlib`, `-D archlib`, `-D sitelib`, ...: Diese Einstellungen definieren, wo Perl nach installierten Modulen sucht. Sie ermöglichen das Aufrüsten von Perl auf neue Patch-Levels (das letzte durch Punkte getrennte Teil in der vollen Versionszeichenkette wie 5.42.0), ohne alle Module neu installieren zu müssen.
    
**2. Kompilierung des Pakets:**


```bash
make
```

**3. Installation des Pakets:**


```bash
make install
```