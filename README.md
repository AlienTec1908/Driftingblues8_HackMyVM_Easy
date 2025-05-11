# Driftingblues8 - HackMyVM (Easy)

![Driftingblues8.png](Driftingblues8.png)

## Übersicht

*   **VM:** Driftingblues8
*   **Plattform:** [HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Driftingblues8)
*   **Schwierigkeit:** Easy
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 16. April 2023
*   **Original-Writeup:** https://alientec1908.github.io/Driftingblues8_HackMyVM_Easy/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel dieser "Easy"-Challenge war es, Root-Zugriff auf der Maschine "Driftingblues8" zu erlangen. Die Enumeration deckte einen Webserver (Apache) auf, der eine OpenEMR-Anwendung hostete. Mit `ffuf` wurde das Passwort (`.:.yarrak.:.31`) für den OpenEMR-Admin-Account (`admin`) durch Brute-Force gefunden. Ein bekannter Exploit für OpenEMR 5.0.1.3 (Exploit-DB 49998, CVE-2018-15139) wurde mit diesen Admin-Credentials verwendet, um eine PHP-Webshell hochzuladen und RCE zu erlangen. Dies führte zu einer initialen Shell als `www-data`. Als `www-data` wurde eine Backup-Datei der `/etc/shadow` (`shadow.backup`) gefunden und heruntergeladen. Mit `john` und `rockyou.txt` wurden aus dieser Datei die Passwörter für die Benutzer `clapton` (`dragonsblood`) und `root` (`.:.yarak.:.`) geknackt. Mittels `su` und diesen Passwörtern wurde zuerst zu `clapton` (User-Flag) und dann zu `root` (Root-Flag) eskaliert.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `gobuster`
*   `ffuf`
*   `hashcat` (Versuch, nicht der erfolgreiche Pfad)
*   `searchsploit`
*   `python` (für Exploit-Skript)
*   `nc` (netcat)
*   `find`
*   `wget`
*   `john` (John the Ripper)
*   `su`
*   Standard Linux-Befehle (`vi`, `grep`, `cat`, `ls`, `cd`, `id`, `pwd`, `which`, `export`, `stty`)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Driftingblues8" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Web Enumeration:**
    *   IP-Findung mittels `arp-scan` (Ziel: `192.168.2.111`, Hostname `driftingblues.hmv`).
    *   `nmap`-Scan identifizierte nur Port 80 (HTTP, Apache) mit einer OpenEMR-Login-Seite.
    *   `gobuster` listete eine typische OpenEMR-Verzeichnisstruktur auf.
    *   SQLi-Versuche mit `sqlmap` und LFI-Versuche (nicht erfolgreich dokumentiert) wurden unternommen.

2.  **Initial Access (als `www-data` via OpenEMR RCE):**
    *   Mittels `ffuf` wurde das Passwort für den OpenEMR-Benutzer `admin` auf der Login-Seite (`main_screen.php`) zu `.:.yarrak.:.31` brute-forced.
    *   `searchsploit` fand einen authentifizierten RCE-Exploit für OpenEMR 5.0.1.3 (Exploit-DB 49998, `49998.py`).
    *   Der Python-Exploit wurde mit den Admin-Credentials (`admin`:`.:.yarrak.:.31`) ausgeführt.
    *   Der Exploit lud eine PHP-Webshell (`shell.php`) in `/sites/default/images/` hoch.
    *   Über die Webshell wurde eine Bash-Reverse-Shell zu einem Netcat-Listener des Angreifers gestartet.
    *   Erfolgreicher Shell-Zugriff als `www-data`.

3.  **Privilege Escalation (von `www-data` zu `clapton` und `root` via Password Cracking):**
    *   Als `www-data` wurde eine Datei `shadow.backup` (eine Kopie von `/etc/shadow`) gefunden (vermutlich durch `linpeas.sh` oder manuelle Suche) und auf die Angreifer-Maschine heruntergeladen.
    *   `john --wordlist=/usr/share/wordlists/rockyou.txt shadow.backup` knackte die Passwörter:
        *   `clapton`:`dragonsblood`
        *   `root`:`.:.yarak.:.`
    *   In der `www-data`-Shell wurde `su clapton` mit dem Passwort `dragonsblood` ausgeführt, um zum Benutzer `clapton` zu wechseln. Die User-Flag wurde gelesen.
    *   Anschließend wurde `su root` mit dem Passwort `.:.yarak.:.` ausgeführt, um Root-Zugriff zu erlangen. Die Root-Flag wurde gelesen.

## Wichtige Schwachstellen und Konzepte

*   **Schwaches Web-Login-Passwort:** Das Passwort für den OpenEMR-Admin-Account konnte via Brute-Force (`ffuf`) erraten werden.
*   **Bekannte Webanwendungs-Schwachstelle (RCE):** Ausnutzung eines öffentlichen, authentifizierten RCE-Exploits für OpenEMR 5.0.1.3 (CVE-2018-15139).
*   **Information Disclosure / Unsicheres Backup:** Eine Kopie der `/etc/shadow`-Datei (`shadow.backup`) war auf dem System zugänglich und enthielt die Passwort-Hashes.
*   **Schwache System-Passwörter:** Die Passwörter für `clapton` und `root` konnten mit `rockyou.txt` geknackt werden.
*   **PHP Webshell / Reverse Shell:** Verwendet für initialen Zugriff und Codeausführung.

## Flags

*   **User Flag (`/home/clapton/user.txt`):** `96716B8151B1682C5285BC99DD4E95C2`
*   **Root Flag (`/root/root.txt`):** `E8E7040D825E1F345A617E0E6612444A`

## Tags

`HackMyVM`, `Driftingblues8`, `Easy`, `Web`, `Apache`, `OpenEMR`, `Brute-Force`, `ffuf`, `RCE`, `Exploit-DB`, `CVE-2018-15139`, `PHP Webshell`, `Shadow Backup`, `Password Cracking`, `John the Ripper`, `Privilege Escalation`, `Linux`
