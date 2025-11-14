[Sed Guide](https://roadmap.sh/ai/guide/die-umfassende-anleitung-zu-sed-textverarbeitung-im-terminal-und-in-bashskripten "Sed - guide")


--- 
# 🧰 `sed` Cheatsheet

| **Befehl / Syntax** | **Beschreibung** | **Beispiel** |
|----------------------|------------------|---------------|
| `sed 's/alt/neu/' datei` | Ersetzt **erstes Vorkommen** von „alt“ pro Zeile durch „neu“. | `sed 's/foo/bar/' text.txt` |
| `sed 's/alt/neu/g' datei` | Ersetzt **alle Vorkommen** von „alt“ pro Zeile. | `sed 's/foo/bar/g' text.txt` |
| `sed -i 's/alt/neu/g' datei` | Ersetzt direkt **in der Datei** (ohne Ausgabe auf stdout). | `sed -i 's/Hello/Hi/g' greetings.txt` |
| `sed -n 'p' datei` | Gibt jede Zeile aus (äquivalent zu `cat datei`). | `sed -n 'p' text.txt` |
| `sed -n '5p' datei` | Gibt nur **Zeile 5** aus. | `sed -n '5p' text.txt` |
| `sed -n '5,10p' datei` | Gibt Zeilen **5 bis 10** aus. | `sed -n '5,10p' text.txt` |
| `sed '/Muster/d' datei` | Löscht Zeilen, die **Muster** enthalten. | `sed '/error/d' logfile` |
| `sed '/Muster/p' datei` | Gibt nur Zeilen aus, die **Muster** enthalten. | `sed -n '/TODO/p' notes.txt` |
| `sed 's/[0-9]/#/g' datei` | Ersetzt **alle Ziffern** durch `#`. | `sed 's/[0-9]/#/g' data.txt` |
| `sed 's/^/# /' datei` | Fügt `# ` am **Zeilenanfang** hinzu. | `sed 's/^/# /' script.sh` |
| `sed 's/$/;/' datei` | Fügt `;` ans **Zeilenende** hinzu. | `sed 's/$/;/' lines.txt` |
| `sed '1,3d' datei` | Löscht **Zeilen 1 bis 3**. | `sed '1,3d' text.txt` |
| `sed 's/foo/bar/2' datei` | Ersetzt nur das **zweite Vorkommen** von „foo“. | `sed 's/foo/bar/2' text.txt` |
| `sed '/Muster/!d' datei` | Behalte nur Zeilen mit **Muster**. | `sed '/error/!d' logfile` |
| `sed -e 's/foo/bar/' -e 's/baz/qux/' datei` | Führt **mehrere Befehle** hintereinander aus. | `sed -e 's/a/A/' -e 's/b/B/' file` |
| `sed '/^#/d' datei` | Löscht **Kommentarzeilen** (beginnen mit `#`). | `sed '/^#/d' config` |
| `sed 's/ *$//' datei` | Entfernt **Leerzeichen am Zeilenende**. | `sed 's/ *$//' code.txt` |
| `sed 's/\(foo\)/\U\1/g' datei` | Wandelt „foo“ in **Großbuchstaben** um. | `sed 's/\(foo\)/\U\1/g' file` |
| `sed 's/\(.\)/\1\n/g'` | Gibt jeden Buchstaben in **eigener Zeile** aus. | `echo "abc" \| sed 's/\(.\)/\1\n/g'` |

---

### 📘 Nützliche Optionen
| **Option** | **Beschreibung** |
|-------------|------------------|
| `-n` | Unterdrückt automatische Ausgabe (nützlich mit `p`). |
| `-i` | Bearbeitet Dateien direkt („in-place“). |
| `-e` | Erlaubt mehrere Befehle in einer Zeile. |
| `-r` oder `-E` | Aktiviert erweiterte reguläre Ausdrücke. |

---

### 💡 Tipps
- Verwende `\` zum Escapen von Sonderzeichen (`\/`, `\(`, `\)` usw.).  
- Backup beim In-Place-Edit: `sed -i.bak 's/foo/bar/g' file`  
- Teste zuerst ohne `-i`, um unbeabsichtigte Änderungen zu vermeiden.
