## 4.2 Erstellen eines eingeschränkten Verzeichnislayouts im LFS-Dateisystem
![[Pasted image 20251019165030.png]]

---

## 4.3 Hinzufügen des LFS-Benutzers![[4.3_Hinzufügen_des_LFS-Benutzers.png]]

---

### **Vergeben der Besitzrechte**![[4.3_Vergeben_der_Besitzrechte.png]]

---

## 4.4 Einrichtung der Umgebung![[4.4_Einrichtung_der_Umgebung.png]]


***


## Was ich gelernt habe?
1. **Piping & Umleitung** – Ich setze Umleitungen mit dem `EOF`-Begrenzer ein, um Konfigurationsdateien wie `.bash_profile` sauber und programmatisch zu erstellen. Diese Technik ist ein fundamentales Skripting-Werkzeug zur Automatisierung von Konfigurationsschritten.  
2. **Objekte erstellen und verlinken (`mkdir`/`ln -s`)** – Ich habe die initiale FHS-Verzeichnisstruktur (`/usr/bin`, `/etc`, etc.) manuell mit `mkdir` und `ln -s` erstellt, um die Pakete in ihren finalen Pfad zu installieren. Ich verstehe dadurch die logische Hierarchie eines Linux-Dateisystems von Grund auf.  
3. **Produktivitätsfunktionen der Shell (Konfiguration)** – Die Konfiguration von Umgebungsvariablen wie `LC_ALL=POSIX` und die PATH-Anpassung in `.bashrc` sind notwendig, um eine stabile Build-Umgebung zu schaffen. Mein Wissen über diese Variablen zeigt ein tiefes Verständnis des Shell-Verhaltens.  
4. **Benutzer- & Gruppenverwaltung (`useradd`/`usermod`)** – Ich habe den unprivilegierten `lfs`-Benutzer mit `useradd` erstellt. Ich beherrsche die Benutzerverwaltung, indem ich Benutzer mit korrekten IDs und Startverzeichnissen für spezifische Build-Zwecke erstelle.