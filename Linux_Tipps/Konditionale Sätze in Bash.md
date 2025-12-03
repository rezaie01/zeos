
#bash #bashKonditionaleAusdrücke #bashCheatsheet
# **Bash `[` … `]` – Cheatsheet**

## **0. Namen der Konstrukte**

|Syntax|Name|
|---|---|
|`[ … ]`|**test command** / **single-bracket test** (POSIX)|
|`[[ … ]]`|**conditional command** / **double-bracket test** (Bash)|

---

# **1. Allgemeine Syntax**

```bash
[ AUSDRUCK ]
```

Abstand zwingend.

`[[ AUSDRUCK ]]` ist eine erweiterte, Bash-spezifische Version.

---

# **2. Quote-Probleme (kurz erklärt)**

`[ … ]` erfordert **immer Quotes** um Variablen (`"$var"`), sonst führen Leerzeichen, Wildcards oder leere Werte zu Syntaxfehlern.

Problem:

```bash
[ $file = "hi" ]   # bricht, wenn $file leer ist
```

Richtig:

```bash
[ "$file" = "hi" ]
```

`[[ … ]]` ist **sicherer**: Leerzeichen, Wildcards und leere Variablen machen keinen Syntaxfehler.

---

# **3. Datei-Tests**

|Ausdruck|Bedeutung|
|---|---|
|`-e FILE`|Datei existiert|
|`-f FILE`|reguläre Datei|
|`-d FILE`|Verzeichnis|
|`-L FILE`|symbolischer Link|
|`-h FILE`|symbolischer Link|
|`-r FILE`|lesbar|
|`-w FILE`|schreibbar|
|`-x FILE`|ausführbar|
|`-s FILE`|nicht leer|
|`-b FILE`|Blockdevice|
|`-c FILE`|Chardevice|
|`-p FILE`|Pipe|
|`-S FILE`|Socket|

---

# **4. String-Tests**

|Ausdruck|Bedeutung|
|---|---|
|`-z STRING`|Länge 0|
|`-n STRING`|Länge > 0|
|`STRING = STRING`|gleich|
|`STRING != STRING`|ungleich|
|`STRING < STRING`|lexikografisch kleiner|
|`STRING > STRING`|lexikografisch größer|

Beispiele:

```bash
[ -z "$var" ]
[ "$user" = "root" ]
```

---

# **5. Zahlen-Tests**

|Ausdruck|Bedeutung|
|---|---|
|`-eq`|gleich|
|`-ne`|ungleich|
|`-gt`|größer|
|`-ge`|größer/gleich|
|`-lt`|kleiner|
|`-le`|kleiner/gleich|

---

# **6. Kombinieren von Bedingungen**

|Operator|Funktion|
|---|---|
|`!`|NOT|
|`-a`|AND|
|`-o`|OR|

Beispiele:

```bash
[ -e file -a -w file ]
[ -z "$x" -o "$x" = "hi" ]
```

Hinweis: Für logische Verknüpfung moderner und sicherer:

```bash
[[ … && … ]]
[[ … || … ]]
```

---

# **7. `[[ … ]]` – erweiterte Tests**

Vorteile:

- Keine Quote-Probleme
    
- unterstützt `&&` / `||`
    
- unterstützt Regex mit `=~`
    

Beispiele:

```bash
[[ "$var" == foo* ]]
[[ $num -gt 3 && $num -lt 10 ]]
[[ "$str" =~ ^[0-9]+$ ]]
```

---

# **8. Beispielkombination**

```bash
if [[ -d /tmp && -w /tmp ]]; then
    echo "tmp ok"
fi
```
