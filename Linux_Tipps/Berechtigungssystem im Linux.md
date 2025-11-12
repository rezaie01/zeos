
### 1️⃣ Dateitypen (d, -, l, b, c, s, p)
- `-` → reguläre Datei 
- `d` → Verzeichnis 
- `l` → symbolischer Link
- `b` → Blockgerät (z. B. Festplatte)
- `c` → Zeichen-Gerät (z. B. Terminal)
- `s` → Socket
- `p` → benannte Pipe (FIFO)

---

### 2️⃣ Benutzer, Gruppe, Andere (u, g, o)

- **u (user)** → Eigentümer der Datei
    
- **g (group)** → Gruppe, der die Datei zugeordnet ist
    
- **o (others)** → alle anderen Benutzer
    

---

### 3️⃣ Berechtigungen: Lesen, Schreiben, Ausführen (r, w, x)

- **r (read)** → Datei lesen / Verzeichnisinhalt auflisten
    
- **w (write)** → Datei ändern / Dateien im Verzeichnis erstellen oder löschen
    
- **x (execute)** → Datei ausführen / Verzeichnis betreten (`cd`)
    

---

### 4️⃣ Symbolische vs. oktale Darstellung

- **Symbolisch**: `rwxr-xr--`
    
- **Oktal**: `chmod 754 file`
    
    - Jede Position: User/Group/Other
        
    - Werte: r=4, w=2, x=1 → Summe = Oktalzahl
        

---

### 5️⃣ `chmod` (Ändern von Berechtigungen)

- Ändert Rechte einer Datei oder eines Verzeichnisses
    
- **Symbolisch:** `chmod u+x file`
    
- **Oktal:** `chmod 755 file`
    

---

### 6️⃣ `chown` (Ändern des Eigentümers)

- Ändert den **Besitzer einer Datei**
    
- Beispiel: `chown user file`
    
- Kann auch die Gruppe ändern: `chown user:group file`
    

---

### 7️⃣ `chgrp` (Ändern der Gruppe)

- Ändert **nur die Gruppe** einer Datei
    
- Beispiel: `chgrp staff file`
    

---

### 8️⃣ `umask` (Standard-Maskierung)

- Definiert, welche Rechte **bei der Erstellung von Dateien/Verzeichnissen entfernt** werden
    
- Beispiel: `umask 022`
    
    - Dateien: 666-022 → 644
        
    - Verzeichnisse: 777-022 → 755
        

---

### 9️⃣ SUID (Set User ID)

- Auf **ausführbaren Dateien** gesetzt
    
- Führt das Programm **mit Rechten des Besitzers** aus
    
- Beispiel: `chmod u+s /usr/bin/passwd`
    
- Symbolisch in `ls -l`: `rws`
    

---

### 🔟 SGID (Set Group ID)

- Auf **Dateien:** wie SUID, aber für Gruppenrechte
    
- Auf **Verzeichnissen:** neue Dateien übernehmen **automatisch die Gruppen-ID des Verzeichnisses**
    
- Symbolisch: `rwxr-sr-x`
    

---

### 1️⃣1️⃣ Sticky Bit (`t`)

- Auf **Verzeichnissen** gesetzt
    
- Nur **Besitzer oder root** darf Dateien löschen/umbenennen
    
- Beispiel: `/tmp` → `drwxrwxrwt`
    

---

### 1️⃣2️⃣ Effekt von Berechtigungen auf Dateien vs. Verzeichnisse

|Recht|Datei|Verzeichnis|
|---|---|---|
|r|Datei lesen|Inhalte auflisten|
|w|Datei schreiben|Dateien erstellen/löschen|
|x|Datei ausführen|Verzeichnis betreten (cd)|

---

### 1️⃣3️⃣ Löschen von Dateien vs. Rechte auf Verzeichnissen

- **Löschen hängt nur von Verzeichnisrechten ab** (w+x)
    
- Rechte auf der Datei selbst **spielen keine Rolle**
    

---

### 1️⃣4️⃣ Zugriffssteuerungsliste (ACL)

- Feinere Rechte als `u/g/o`
    
- Beispiel: `setfacl -m u:alice:r file` → Alice hat Lesezugriff
    

---

### 1️⃣5️⃣ Standard-ACLs

- Werden auf **neue Dateien/Verzeichnisse** in einem Verzeichnis vererbt
    
- Beispiel: `setfacl -d -m g:staff:rw dir` → neue Dateien bekommen automatisch rw für staff
    

---

### 1️⃣6️⃣ Vererbung von Rechten bei Verzeichnissen

- SGID auf Verzeichnis → neue Dateien übernehmen **Gruppen-ID**
    
- Standard-ACLs → Rechte werden vererbt
    

---

### 1️⃣7️⃣ Kombination von Rechten in Symbolisch + Oktal

- Symbolisch: `chmod u+rwx,g+rx,o-r file`
    
- Oktal: `chmod 750 file`
    

---

### 1️⃣8️⃣ `ls -l` Ausgabe und Interpretation

Beispiel:

```
-rwsr-sr-t 1 root staff 1234 Nov 5 16:00 file
```

- `-` → Datei
    
- `rws` → owner: rw + SUID
    
- `r-s` → group: r + SGID
    
- `r-t` → others: r + Sticky Bit
    
- `1` → Anzahl Hardlinks
    
- `root` → Besitzer
    
- `staff` → Gruppe
    
- `1234` → Größe
    
- `Nov 5 16:00` → Datum
    
- `file` → Name
    

---

### 1️⃣9️⃣ Maskenberechnung (z. B. 022, 027)

- `umask 022` → Dateien 644, Verzeichnisse 755
    
- `umask 027` → Dateien 640, Verzeichnisse 750
    
- Formel: **Standardrechte - umask = effektive Rechte**
    

---

### 2️⃣0️⃣ Sonderrechte auf Netzwerk- oder Gerätedateien

- **Blockgeräte (b) und Zeichengeräte (c)** → spezielle Rechte für Lese/Schreibzugriff
- SUID/SGID kann für Programme auf Geräten sinnvoll sein
- ACLs und Sticky Bit gelten genauso


# fragen: 
- wer darf löschen: