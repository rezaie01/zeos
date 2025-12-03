
*Letzte Aktualisierung: 2025-10-17 16:56*
## Kernel-Version
![[kernel-version-host.png]]  
*Abb. 1: Kernel-Version ermittelt mit `uname -r`*

![[kernel-detail-host.png]]  
*Abb. 2: Detaillierte Kernel-Informationen via `uname -a`*

---

## Distributionsdetails
**Verwendete Distribution**:  
![[Distro-Name.png]]  
*Abb. 3: Ausgabe des Distributionsnamens im Terminal*

![[lsb_release_ergebnis.png]]  
*Abb. 4: Vollständige Systeminformationen mit `lsb_release -a`*

---

## Hardware-Konfiguration
![[Screenshot-2025-10-17-163032.png]]
*Abb. 5: Hardware-Übersicht Teil 1 VirtualBox aktueller Zustand*

![[Screenshot-2025-10-17-163053.png]]
*Abb. 5: Hardware-Übersicht Teil 2 VirtualBox aktueller Zustand*`*

---

## Abschlussprüfung
![[kapitel-3-überprüft.png]]  
*Abb. 7: Erfolgsbestätigung für Kapitel 4 (Alle Pakete & Patches erfolgreich herunterladen und überprüfen*

***

## Was ich gelernt habe?
1. **Backup-Strategien (`tar`, Wiederherstellung)** – Ich habe die Wichtigkeit erkannt, `tar` für das Erstellen von komprimierten Backups zu nutzen, insbesondere nachdem kritische temporäre Tools gebaut wurden. Das ist die Grundlage für eine zuverlässige Wiederherstellungsstrategie.  
2. **`wget` (Webprüfung)** – Ich nutze `wget` nicht nur zum Herunterladen der Quellpakete, sondern auch zur Verifizierung ihrer Integrität mittels MD5-Prüfsummen, bevor ich sie verwende. Ich garantiere damit die Unverfälschtheit der verwendeten Quellen.
