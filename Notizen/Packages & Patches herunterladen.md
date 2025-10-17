# 1.  Das `/mnt/sources` Verzeichnis erstellen:
	1. Erstelle: `mkdir -v $LFS/sources`
	2. Ändere die Berechtigungen als `schreibbar` `sticky`:
> ** #Sticky_Berechtigung**: Nur das Eigentümer darf Dateien löschen. 
>**chmod Flagge**: t
```bash
sudo chmod -v a+wt $LFS/sources
```
> #chmod_Flagge:
> a: beeinflusst alle Benutzer (Eigentümer, Gruppe, Andere)
> +: hinzufüge Berechtigungen
> w: schreiben
> t: macht `sticky`, wird meistens für die Verzeichnisse verwendet.


# 2. Die Packages & Patches herunterladen:
1. Lade die Liste von allen `wget https://www.linuxfromscratch.org/lfs/downloads/stable/wget-list`
2. Lade die Packages herunter: 
```bash
sudo wget -i wget-list -P $LFS/sources
```
>-i --input-file=Datei
>-P --directory-prefix=Prefix
3. Überprüfe die checksums:
	1. Hole die MD5-Prüfsummen `wget https://www.linuxfromscratch.org/lfs/downloads/stable/md5sums`
	2. Führe MD5-Prüfungen durch
```sh
pushd $LFS/sources
md5sum -c md5sums | grep -v 'OK$'
popd
```
>>`pushd` => Wechselt kurzfristig in ein anderes Verzeichnis => `popd` kehrt zurück
>
>>**md5sum**: Prüft die Summe & `-c` bezeichnet die Summendatei
>
>>`grep -v 'OK$'`: Filtert nur die Zeilen, die keine `OK` am Ende haben. 
>>`-v` prüft gegen der Bedingung
4. [[Vollständige Tabelle von allen benötigten Packages & Patches]]



   