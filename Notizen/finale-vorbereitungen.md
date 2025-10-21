# Kapitel 4 – finale Vorbereitungen

---

## 4.2 Erstellen eines eingeschränkten Verzeichnislayouts im LFS-Dateisystem

Im LFS-Wurzelverzeichnis sollen nur essenzielle Ordner angelegt werden, um die später benötigten temporären Tools sinnvoll installieren zu können. 

```bash
mkdir -pv $LFS/{etc,var} $LFS/usr/{bin,lib,sbin}

for i in bin lib sbin; do
  ln -sv usr/$i $LFS/$i
done

case $(uname -m) in
  x86_64) mkdir -pv $LFS/lib64 ;;
esac

mkdir -pv $LFS/tools
```
> **<font color="#9bbb59">ERKLÄRUNG</font>:**
>*`mkdir -p`: Legt erforderliche übergeordnete Verzeichnisse bei Bedarf an (z.B. via `-p`-Option).*
>
>*`$LFS/{etc,var}`: (**brace expansion**) -> Es erweitert sich zu: `$LFS/usr/bin $LFS/usr/lib $LFS/usr/sbin`.*
>
>*`ln -sv a b`: Erstellt symbolische Links (`-s`)(keine Hardlinks) von `b` nach `a`, z.B. verweist `/bin` auf `/usr/bin`.  `-v` -> ausführlich*
>
>*`case $(uname -m) in  x86_64) mkdir -pv $LFS/lib64; ;; esac`: Erstellt bei 64-Bit-Systemen (x86_64) das Verzeichnis `$LFS/lib64`.*  
>
>*`mkdir -pv $LFS/tools`: für spätere Installation von Cross-Compilern und temporären Tools.*


### **Theoretischer Hintergrund: FHS und Verzeichnisstruktur**  
Die [[Filesystem Hierarchy Standard (FHS)]] definiert die Anordnung von Systemdateien und Programmen. Moderne Systeme nutzen Symlinks (z.B. `/bin` → `/usr/bin`), um historisch bedingte Trennung zu vereinheitlichen. LFS folgt dem FHS-Standard, um Kompatibilität und Übersichtlichkeit zu gewährleisten.

>**<font color="#938953">HINWEIS:</font>**
>*Das Verzeichnis `/usr/lib64` darf nicht erstellt werden. Falls vorhanden, deutet dies auf einen Fehler hin, der besonders beim Cross-Compiling schwer auffindbare Probleme verursachen kann.*

---

## 4.3 Hinzufügen des LFS-Benutzers

**Risiko beim Bauen als root**: Vermeiden, um Host-Systemschäden durch Fehler zu verhindern. Unprivilegierten Benutzer verwenden.  

### Anlegen einer neuen Gruppe und eines neuen Benutzers

**Befehle:**
```bash
groupadd lfs
useradd -s /bin/bash -g lfs -m -k /dev/null lfs
```
> **<font color="#9bbb59">ERKLÄRUNG</font>:**
>*`groupadd lfs` (neue Gruppe für Berechtigungsmanagement).*
>
>*`useradd`: `-g` -> Gruppe auf ,,lfs'' setzen. `-m` -> Home-Verzeichnis erstellen. 
>`-k /dev/null` -> Normalerweise werden Dateien aus `/etc/skel` bei der Usererstellung ins Homedir kopiert; hier verhindert `/dev/null` (leer) die Nutzung bestehender Dateien.*

- **useradd -s /bin/bash -g lfs -m -k /dev/null lfs**
  - **-s /bin/bash**: Legt Bash als Standard-Shell für den Benutzer „lfs” fest. So landet der Benutzer nach dem Login direkt in Bash, nicht z.B. in sh oder dash.
  - **-g lfs**: Setzt die Primärgruppe des Benutzers auf „lfs”.
  - **-m**: Erstellt automatisch ein Home-Verzeichnis, typischerweise `/home/lfs`, falls noch nicht vorhanden.
  - **-k /dev/null**: Normalerweise werden Dateien aus dem Vorlagenverzeichnis (`/etc/skel`) bei der Usererstellung ins Homedir kopiert; hier wird verhindert, dass bestehende Dateien genutzt werden (weil `/dev/null` leer ist).
  - Das letzte Argument `lfs` ist der Benutzername.

Um sich **direkt als `lfs` anzumelden** oder mit **`su - lfs`** von einem Nicht-root-Account zu wechseln, ist ein Passwort erforderlich. 
Befehl: 
```bash
passwd lfs
```

**Vergeben der Besitzrechte:**
Das ist notwendig, weil dieser Benutzer später in sie schreiben können muss, um dort Programme und Dateien zu erstellen oder zu modifizieren.
```bash
chown -v lfs $LFS/{usr{,/*},var,etc,tools}
case $(uname -m) in
  x86_64) chown -v lfs $LFS/lib64 ;;
esac
```

**Wechsel in den neuen Account:**
```bash
su - lfs
```
> **<font color="#9bbb59">ERKLÄRUNG</font>:**
>*`su - lfs`: Startet eine Login-Shell als Benutzer lfs. Das `-` sorgt dafür, dass die Umgebung komplett neu geladen wird (Profil-Skripte werden ausgewertet, Umgebung ist „sauber”).*

---

## 4.4 Einrichtung der Umgebung

### Ziel  
Erstelle eine Umgebung für Benutzer „lfs“, die:  
- **keine Umgebungsvariablen der Host-Distribution** verwendet,  
- Pfade und Compiler-Einstellungen für reproduzierbare Builds festlegt,  
- neue Programme aus `$LFS/tools` priorisiert.

---

### **`~/.bash_profile`**

Beim SSH, TTY und `su - lfs` die `.bash_profile` eines Benutzers gelesen wird.

```bash
cat > ~/.bash_profile << "EOF"
exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
EOF
```
> **<font color="#9bbb59">ERKLÄRUNG</font>:**
>*`cat > DATEI << "EOF"` erstellt eine Datei (`>`) oder ersetzt sie. Mit `>>`  würde der Inhalt hinzufügt, und der Input endet, sobald eine Zeile nur `EOF` enthält.*
>
*`exec`: ersetzt die aktuelle Shell vollständig durch den angegebenen Befehl — hier `env`. bzw. das, was env startet.*
>
>*`env -i VARAIBLE=wert BEFEHL`: Starte BEFEHL mit gesetzten VARIABLEN. `-i` startet mit leerer Umgebung.* 
>*`PS1='\u:\w\$ '` → definiert den neuen Prompt. `\u` (Benutzer) & `\w` (aktuelle Arbeitsverzeichnis) => (z. B. `lfs:/Home/lfs$ `).*





---

### **~/.bashrc**

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
> **<font color="#9bbb59">ERKLÄRUNG</font>:**
>*`cat > ~/.bashrc << "EOF"`: Erstellt oder ersetzt die Datei `~/.bashrc` mit dem folgenden Inhalt, bis eine Zeile nur `EOF` enthält.*
>
>*`set +h` Deaktiviert Hashing, um sicherzustellen, dass Befehle neu gesucht werden.*
>
>*`umask 022` Setzt Standardberechtigungen für neue Dateien/Verzeichnisse (rwxr-xr-x).* Schau [[umask]] an. 
>
>*`LC_ALL=POSIX` Setzt die Locale auf POSIX für konsistente Ausgabe und Verhalten.*
>
>*`LFS_TGT=$(uname -m)-lfs-linux-gnu` Legt das Zielsystem für den Build fest.*
>
>*`PATH=/usr/bin` Setzt PATH, inkl. Standardbefehle.*
>*`if [ ! -L /bin ]; then PATH=/bin:$PATH; fi` Fügt `/bin` hinzu, wenn es kein Symlink ist.*
>*`PATH=$LFS/tools/bin:$PATH` Priorisiert die LFS-Tools im PATH. Damit beim Bau von LFS **immer die LFS-Tools verwendet werden**, nicht die Systemversionen.*
>
>*`CONFIG_SITE=$LFS/usr/share/config.site` Setzt Konfigurationsdatei für Builds. `config.site` enthält **vordefinierte Konfigurationsoptionen**, die **automatisch bei `./configure` übernommen** werden, z. B. Compiler-Flags, Pfade oder Feature-Einstellungen.*
>
>*`export LFS LC_ALL LFS_TGT PATH CONFIG_SITE` Exportiert alle wichtigen Variablen in die Umgebung.*



---

#### Um paralleles Kompilieren zu ermöglichen (bei Mehrkernsystemen):

```bash
cat >> ~/.bashrc << "EOF"
export MAKEFLAGS=-j$(nproc)
EOF
```
- **MAKEFLAGS=-j$(nproc)**: Setzt die Anzahl paralleler Prozesse bei Kompilierung auf die Anzahl der logischen Kerne (`nproc` gibt diese Zahl zurück; z.B. 8 bei einem 8-Kern-Prozessor).

**Vorsicht:** Niemals `-j` ohne Zahl! Sonst kann das System instabil werden, weil Make *unendlich* viele Prozesse starten kann.

---

#### Anwendung der neuen Shell-Umgebung:
```bash
source ~/.bash_profile
```
- **source**: Liest und lädt die Umgebungsvariable aus der angegebenen Datei in die aktuelle Shell.

---

#### Zusatz: Umgang mit /etc/bash.bashrc

Einige kommerzielle Distributionen nutzen `/etc/bash.bashrc` für eigene Systemvorgaben; diese Datei kann Variable oder Einstellungen enthalten, die den Bauprozess stören können. Falls vorhanden, sollte sie temporär umbenannt werden:

```bash
[ ! -e /etc/bash.bashrc ] || mv -v /etc/bash.bashrc /etc/bash.bashrc.NOUSE
```

---

## 4.5 SBUs (Standard Build Units)

- **Definition:** Relative Zeiteinheit für Paketbau, basierend auf der Zeit für Binutils 1. Durchgang.  
- **Beispiel:** 10 min für Binutils → 4.5 SBU ≈ 45 min.  
- **Hinweise:** Richtwert; Zeiten variieren nach System, Kernel, Parallelisierung. SBU-Angaben gelten unter bestimmten Kompilierbedingungen (z.B. `-j4`).  
- **Messung:** `time { ./configure ... && make && make install; }` → „real“ = 1 SBU.  
- **Nutzen:** Vorhersage für andere Pakete ohne stundenlange Berechnungen.

## 4.6 Test Suites

- **Zweck:** Prüfen, ob ein Tool erwartungsgemäß funktioniert (Sanity Check).  
- **Limit:** Nicht alle Fehler werden erkannt; Kernpakete (GCC, binutils, glibc) besonders wichtig.  
- **Cross-Compiler:** Tests oft nicht auf Host ausführbar, werden übersprungen.  
- **PTYs:** Benötigt für viele Tests; fehlt `devpts`, gibt Fehlermeldungen.  
- **Fehlertoleranz:** Bekannte, harmlose Fehler → LFS-Logs prüfen: [Build-Logs 12.4](https://www.linuxfromscratch.org/lfs/build-logs/12.4/)

## Praktische Analogien

- **SBUs:** Zeitmaßstab wie Kochzeiten.  
- **User/Gruppen:** Abteilungen/Mitarbeiter; Zugriff nach Bereich, Schreiben nur auf eigene Daten.