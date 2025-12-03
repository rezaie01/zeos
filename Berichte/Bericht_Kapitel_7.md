
## Was ich gelernt habe?

1. **Chroot-Umgebungen zur Reparatur** – Das Betreten der `chroot`-Umgebung ist der zentrale Wendepunkt, der die vollständige Isolation vom Host herstellt. Ich beherrsche diese Technik, die grundlegend für Systemwartung und Upgrades ist.  
2. **Kern-Identitätsdateien (`/etc/passwd` etc.)** – Ich erstelle die grundlegenden Identitätsdateien manuell und definiere essentielle Gruppen wie `tty` mit GID 5. Ich verstehe, wie Benutzer- und Gruppen-IDs auf der niedrigsten Ebene im System verankert sind.  
3. **Lokale Host-Auflösung (`/etc/hosts`, `resolv.conf`)** – Ich erstelle die rudimentären Netzwerkdateien `/etc/hosts` und `/etc/resolv.conf`, die für die korrekte Funktion von Testsuiten und der Netzwerkerkennung notwendig sind.