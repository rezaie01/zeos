***
## `printf` beim find

```bash
find . -type f -name '9.*.*' -printf '[[%f]]\n' 
```


| Platzhalter   | Bedeutung                                   |
| ------------- | ------------------------------------------- |
| `%p`          | Pfad                                        |
| `%f`          | Dateiname                                   |
| `%s`          | Größe (Bytes)                               |
| `%u`          | Besitzer                                    |
| `%g`          | Gruppe                                      |
| `%m`          | Permissions (numerisch)                     |
| `%M`          | Permissions (symbolisch, z. B. `rw-r--r--`) |
| `%TY %Tm %Td` | Datum (Jahr/Monat/Tag)                      |
| `\n`          | Zeilenumbruch                               |
| `\t`          | Tab                                         |
***
## Dateinamen sicher lesen (Leerzeichen-tauglich)
#printf #dateiNameMitLeerZeichen
- Nicht `ls` verwenden → bricht bei Leerzeichen.
- `printf` listet nicht selbst – die **Shell erweitert `./*`**.
- `printf '%s\0' ./*` gibt Dateinamen mit `\0` aus → sicher für `read`.

Sichere Schleife:

```bash
while IFS= read -r -d '' f; do
    [[ ! -s "$f" ]] && rm -iv -- "$f"
done < <(printf '%s\0' ./*)

```


***