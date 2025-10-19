# Finaler Plan für LFS 12.4 - Kapitel 4

## ⚒️ Kapitel 4: Vorbereitung der Build-Umgebung

### Ziel: LFS-Benutzer, Verzeichnisstruktur und eine saubere Arbeitsumgebung einrichten.

---

### 1. LFS-Benutzer und Gruppe erstellen

Ein dedizierter `lfs`-Benutzer isoliert den Build-Prozess vom Host-System und minimiert Risiken.

- [x] **1.1. Gruppe `lfs` erstellen (als `root`):**
    ```bash
    sudo groupadd lfs
    ```

- [x] **1.2. Benutzer `lfs` erstellen (als `root`):**
    *   Dieser Befehl setzt die primäre Gruppe (`-g`), erstellt ein Home-Verzeichnis (`-m`) und verhindert das Kopieren von Konfigurationsdateien vom Host (`-k /dev/null`).
    ```bash
    sudo useradd -s /bin/bash -g lfs -m -k /dev/null lfs
    ```

- [x] **1.3. Passwort für `lfs`-Benutzer setzen (als `root`):**
    *   Dies ist notwendig, um dich später mit `su - lfs` als dieser Benutzer anzumelden.
    ```bash
    sudo passwd lfs
    ```

- [x] **1.4. Überprüfung (als `root`):**
    *   Prüfe, ob Benutzer und Gruppe korrekt angelegt wurden.
    ```bash
    id lfs
    ```
    *   *Erwartete Ausgabe:* `uid=... (lfs) gid=... (lfs) groups=... (lfs)`

---

### 2. Verzeichnisstruktur und Berechtigungen einrichten

Wir erstellen die Arbeitsverzeichnisse und weisen sie dem `lfs`-Benutzer zu.

- [x] **2.1. LFS-Mountpoint als Variable setzen (optional, aber empfohlen):**
    *   Definiere die `$LFS` Variable in deiner Host-Shell, um Tippfehler zu vermeiden.
    ```bash
    export LFS=/mnt/lfs
    ```

- [x] **2.2. Verzeichnisse erstellen (als `root`):**
    *   `$LFS/sources`: Für die heruntergeladenen Quellcode-Archive.
    *   `$LFS/tools`: Für die temporäre Cross-Toolchain.
    ```bash
    sudo mkdir -pv $LFS/sources
    sudo mkdir -pv $LFS/tools
    ```

- [x] **2.3. Eigentümerschaft zuweisen (als `root`):**
    *   Der `lfs`-Benutzer muss der Eigentümer dieser Verzeichnisse sein.
    ```bash
    sudo chown -v lfs:lfs $LFS/sources
    sudo chown -v lfs:lfs $LFS/tools
    ```

- [x] **2.4. "Sticky Bit" für das `sources`-Verzeichnis setzen (als `root`):**
    *   **Wichtiger Schritt:** Dies stellt sicher, dass nur der Eigentümer einer Datei diese löschen kann, auch wenn andere Schreibrechte im Verzeichnis haben.
    ```bash
    sudo chmod -v a+wt $LFS/sources
    ```

- [x] **2.5. Überprüfung (als `root`):**
    *   Prüfe die Eigentümer und Berechtigungen der Verzeichnisse.
    ```bash
    ls -ld $LFS/sources $LFS/tools
    ```
    *   *Erwartete Ausgabe:* `drwxr-xr-x lfs lfs ... /mnt/lfs/tools` und `drwxrwxrwt lfs lfs ... /mnt/lfs/sources` (beachte das `t` am Ende der `sources`-Berechtigungen).

---

### 3. Quellpakete herunterladen und verifizieren

Alle für LFS 12.4 benötigten Pakete werden heruntergeladen und ihre Integrität geprüft.

- [x] **3.1. Zum `lfs`-Benutzer wechseln:**
    ```bash
    su - lfs
    ```
    *   *Hinweis:* Das `-` ist wichtig, da es eine Login-Shell startet und das Home-Verzeichnis des `lfs`-Benutzers lädt.

- [x] **3.2. In das `sources`-Verzeichnis wechseln (als `lfs`-Benutzer):**
    ```bash
    cd $LFS/sources
    ```

- [x] **3.3. Download-Listen herunterladen (als `lfs`-Benutzer):**
    *   Diese Dateien enthalten die URLs und Prüfsummen aller benötigten Pakete.
    ```bash
    wget https://www.linuxfromscratch.org/lfs/downloads/12.1/wget-list
    wget https://www.linuxfromscratch.org/lfs/downloads/12.1/md5sums
    ```
    *   **Achtung:** Passe die Versionsnummer (hier `12.1`) an die exakte Version deines LFS-Buches an, falls sie abweicht! Für LFS 12.1 ist dies korrekt.

- [x] **3.4. Alle Pakete herunterladen (als `lfs`-Benutzer):**
    ```bash
    wget --input-file=wget-list --continue --directory-prefix=$LFS/sources
    ```

- [x] **3.5. Integrität der Downloads prüfen (als `lfs`-Benutzer):**
    ```bash
    md5sum -c md5sums
    ```
    *   **Ergebnis:** Alle Zeilen müssen mit `: OK` enden. Wenn eine Datei `FAILED` anzeigt, lösche sie und führe den `wget`-Befehl erneut aus.

---

### 4. Build-Umgebung für den `lfs`-Benutzer konfigurieren

Dies ist ein **entscheidender Schritt**, um eine saubere und vom Host-System isolierte Build-Umgebung zu schaffen.

- [ ] **4.1. `.bash_profile` für den `lfs`-Benutzer erstellen (als `lfs`-Benutzer):**
    *   Diese Datei sorgt dafür, dass bei jeder Anmeldung eine saubere Shell gestartet wird.
```bash
cat > ~/.bash_profile << "EOF"
exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
EOF
```

- [ ] **4.2. `.bashrc` für den `lfs`-Benutzer erstellen (als `lfs`-Benutzer):**
    *   Diese Datei setzt die für den LFS-Build-Prozess notwendigen Umgebungsvariablen.
```bash
cat > ~/.bashrc << "EOF"
set +h
umask 022
LFS=/mnt/lfs
LC_ALL=POSIX
LFS_TGT=$(uname -m)-lfs-linux-gnu
PATH=/usr/bin
if [ ! -L /bin ]; then PATH=/bin:$PATH; fi
PATH=$LFS/tools/bin:$PATH
CONFIG_SITE=$LFS/usr/share/config.site
export LFS LC_ALL LFS_TGT PATH CONFIG_SITE
EOF
```
    *   *Anmerkung:* Diese Konfiguration ist exakt auf LFS 12.1 abgestimmt. Sie stellt sicher, dass die temporären Werkzeuge in `$LFS/tools/bin` zuerst gefunden werden und die Lokalisierungseinstellungen (`LC_ALL=POSIX`) keine Build-Fehler verursachen.

- [ ] **4.3. Symbolischen Link für `/tools` erstellen (als `root`):**
    *   Verlasse zuerst die `lfs`-Shell mit `exit`.
    ```bash
    exit
    ```
    *   Erstelle dann den Link als `root`.
    ```bash
    sudo ln -sv $LFS/tools /
    ```
    *   *Erwartete Ausgabe:* `'/tools' -> '/mnt/lfs/tools'`

---

### 5. 📸 Meilenstein: "Kapitel 4 abgeschlossen"

Die Vorbereitung ist abgeschlossen. Es ist ein guter Zeitpunkt, den Status zu dokumentieren.

- [ ] **5.1. Umgebungsvariablen aktivieren und prüfen:**
    *   Melde dich erneut als `lfs`-Benutzer an, damit die neuen `.bash_profile` und `.bashrc` geladen werden.
    ```bash
    su - lfs
    ```
    *   Führe die folgenden Befehle aus und mache Screenshots oder kopiere die Ausgaben in deine Notizen.
    ```bash
    echo $LFS
    echo $LC_ALL
    echo $LFS_TGT
    echo $PATH
    ```
    *   *Erwartete Ausgaben:* `/mnt/lfs`, `POSIX`, `x86_64-lfs-linux-gnu` (oder ähnlich), und `/mnt/lfs/tools/bin` sollte am Anfang des `PATH` stehen.

- [ ] **5.2. Dokumentation sichern:**
    *   Speichere die Screenshots und Terminal-Ausgaben in deinem Obsidian Vault.
    *   Wenn du eine VM nutzt, ist **jetzt** der perfekte Zeitpunkt für einen Snapshot mit dem Namen "LFS Chapter 4 Complete".