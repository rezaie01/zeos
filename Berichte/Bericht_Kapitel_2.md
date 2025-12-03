## Was ich gelernt habe?

1. **Core Command Execution & Help** – Ich habe gelernt, dass Kernbefehle wie `make` und `install` nicht einfach "magisch" funktionieren, sondern exakte Abhängigkeiten und Syntax benötigen, die ich durch das Lesen von Quell-READMEs und `make help` selbst ermitteln kann. Dieses tiefgehende Verständnis der Build-Prozesse hebt mich von Anwendern ab, die nur Paketmanager nutzen.

2. **Directory Navigation** – Ich verstehe, dass das Wechseln zwischen Host- und LFS-Partitionen (`$LFS`) und das Isolieren von Quellverzeichnissen essentiell sind, um eine saubere Bausystematik zu gewährleisten. Diese Disziplin zeigt, dass ich in komplexen Umgebungen strukturiert und fehlerfrei arbeiten kann.

3. **fdisk** **/** **lsblk** **(Partitioning)** – Ich habe gelernt, physische Datenträger zu analysieren und Partitionen spezifisch für Linux-Dateisysteme (z. B. ext4) und Swap-Speicher vorzubereiten. Das Verständnis der grundlegenden Festplattenverwaltung ist eine essentielle Systemadministrations-Grundlage.

4. **Permissions (****chmod****) &** **ls -l** – Die manuelle Zuweisung von Zugriffsrechten, wie `chmod 755`, ist notwendig, um die Sicherheit und Funktionsfähigkeit des Dateisystems (z. B. auf `/dev`) zu gewährleisten, bevor andere Prozesse starten. Dieses Wissen ermöglicht mir, Sicherheitskonzepte aktiv umzusetzen.

5. **File Ownership `chown`) &** **`umask`** – Durch die strikte Trennung der Benutzerrechte (z. B. das Setzen der `umask` auf `022` und die Zuweisung des `lfs`-Benutzers) habe ich die Wichtigkeit der Isolation im Build-Prozess verstanden. Ich kann garantieren, dass erstellte Dateien die korrekten Standardrechte erhalten.
