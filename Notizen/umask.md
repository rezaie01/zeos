## Umask: Konzept, Praxis und Bedeutung

Ein grundlegendes **Sicherheits- und Organisationsmerkmal** von Linux/Unix ist das Berechtigungssystem für Dateien und Verzeichnisse (siehe auch: `ls -l`).

### Was ist die umask?

- Die **umask** legt die „Negativmaske“ für Datei- und Verzeichnisrechte fest, d.h. sie gibt an, welche Rechte für *neu* erstellte Dateien und Verzeichnisse weggefiltert werden.
- **Standardwerte**:
  - Für **Dateien**: Maximale Rechte sind 666 (rw-rw-rw-), *kein execute!*
  - Für **Verzeichnisse**: Maximale Rechte sind 777 (rwxrwxrwx)
- **umask 022** bedeutet:
  - Dateien: 666-022 = 644 (rw-r--r--)
  - Verzeichnisse: 777-022 = 755 (rwxr-xr-x)
- Damit dürfen neu erstellte Dateien nur noch vom Besitzer geschrieben werden, alle dürfen sie lesen; neue Verzeichnisse sind für jeden betretbar, aber nur für Besitzer beschreibbar.

#### Typische umask-Werte

| umask | Rechte neue Verzeichnisse | Rechte neue Dateien | Kommentar                         |
|-------|--------------------------|--------------------|-----------------------------------|
| 022   | 755 (rwxr-xr-x)          | 644 (rw-r--r--)    | Standard auf den meisten Systemen |
| 002   | 775 (rwxrwxr-x)          | 664 (rw-rw-r--)    | Gruppen dürfen schreiben          |
| 077   | 700 (rwx------)          | 600 (rw-------)    | Sehr restriktiv, nur Besitzer     |

**Warum empfiehlt LFS „022“?**
- Es ist ein Kompromiss zwischen Sicherheit (niemand außer dem Besitzer kann schreiben) und Benutzbarkeit (andere können Dateien und Verzeichnisse sehen/lesen, aber nicht verändern).

**Setzen der umask in LFS:**
```bash
umask 022
```
- Diese Zeile steht sowohl im Host als auch im neu einzurichtenden Bash-Umfeld.

**Abfrage und Kontrolle:**
```bash
umask   # Gibt aktuelle Maske als Zahl zurück (bzw. mit -S symbolisch)
```

#### Beispiel für die Auswirkungen einer umask:

```bash
umask 022
touch testfile
ls -l testfile
# => -rw-r--r--
mkdir testdir
ls -ld testdir
# => drwxr-xr-x
```

---