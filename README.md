# OverTheWire Bandit Writeups

[![Status](https://img.shields.io/badge/status-incomplete-yellow)](https://github.com/)
[![Focus](https://img.shields.io/badge/focus-Linux%20%7C%20Cybersecurity-blue)](https://github.com/)
[![Level](https://img.shields.io/badge/levels-spoilerfree-orange)](https://github.com/)

Dieses Repository dokumentiert meine Lösungen zum [Bandit Wargame](https://overthewire.org/wargames/bandit/) von [OverTheWire](https://overthewire.org).

**Alle Passwörter und konkrete Antworten habe ich versucht weggelassen um niemanden zu spoilern.**

## Inhaltsangabe
- [OverTheWire Bandit Writeups](#overthewire-bandit-writeups)
  - [Inhaltsangabe](#inhaltsangabe)
  - [Motivation und Ziel](#motivation-und-ziel)
  - [Inhalt](#inhalt)
  - [Wargames](#wargames)
    - [Bandit 0](#bandit-0)
    - [Bandit 1](#bandit-1)
    - [Bandit 2](#bandit-2)
    - [Bandit 3](#bandit-3)
    - [Bandit 4](#bandit-4)
    - [Bandit 5](#bandit-5)
    - [Bandit 6](#bandit-6)
    - [Bandit 7](#bandit-7)
    - [Bandit 8](#bandit-8)
    - [Bandit 9](#bandit-9)
    - [Bandit 10](#bandit-10)
    - [Bandit 11](#bandit-11)
    - [Bandit 12](#bandit-12)
    - [Bandit 13](#bandit-13)
    - [Bandit 14](#bandit-14)
    - [Bandit 15](#bandit-15)
    - [Bandit 16](#bandit-16)
    - [... (coming soon)](#-coming-soon)

## Motivation und Ziel

Ich möchte lernen wie man:

- verschiedene Shell Befehle unter Linux verwendet.
- Ergebnisse klar und strukturiert dokumentiert.

## Inhalt

Jedes Bandit-Level mit:

- kurzer Problembeschreibung.
- verwendeten Befehlen.
- einer knappen Erklärung warum das funktioniert hat.
- (manchmal) weitere interessante Learnings.

Dokumentiert und Erklärt werden nur 'neue' Befehle, also alle, die nicht bei einem vorherigen Level verwendet wurden.

## Wargames

Im Folgenden sind meine Lösungsansätze aufgelistet.

### Bandit 0
---

**Problem**  
Verbinden mit einem Server über SSH.

**Lösung**  
```bash
ssh user@host -p portnummer
```

**Erklärung**  
`ssh` als SSH client, `-p` um einen Port zu spezifizieren.

**Gelernt**  
Standartport für SSH ist 22, wenn der Port anders ist muss dieser mit `-p` spezifiziert werden.

### Bandit 1
---

**Problem**  
Anzeigen des Inhalts einer Datei im Terminal.

**Lösung**  
```bash
cat filename
```

**Erklärung**  
`cat` liest den Inhalt der spezifizierten Dateien, hängt alles aneinander und gibt es in `stdout` aus. 
Dieser Output wird am Ende des Befehls im Terminal angezeigt.

**Gelernt**  
- `stdin`, `stdout` und `stderr` (0, 1, 2) als File Descriptors
- Pipes `|` leiten `stdout` eines Befehls an `stdin` des nächsten Befehls weiter
- Redirects mit `< input` und `> output` (bzw. `>&` + File Descriptor)

### Bandit 2
---

**Problem**  
Anzeigen einer Datei `-` im Terminal.

**Lösung**  
```bash
cat ./-
# oder auch
cat < -
```

**Erklärung**  
`cat` interpretiert einen Bindestrich als Aufforderung den auszugebenden Inhalt von `stdin` zu nehmen.
`./` steht hierbei für den relativen Pfad auf den aktuellen Ordner, alles direkt Folgende wird als Datei bzw. Unterordner interpretiert.
`< filename` ist die Anweisung an `cat` den Input aus der folgenden Datei zu nehmen und erzwingt dadurch die Interpretation des Bindestrichs als Dateinamen.

### Bandit 3
---

**Problem**  
Anzeigen einer Datei mit Leerzeichen im Terminal.

**Lösung**  
```bash
cat 'dateiname mit leerzeichen'
```

**Erklärung**  
Im Terminal werden Leerzeichen als Trennzeichen zwischen Parametern interpretiert. 
Mit `'...'` werden Dateinamen und Ordnernamen trotz Leerzeichen als ein Argument interpretiert.
(Alternativ können Leerzeichen auch einzeln mit `\` escaped werden.)

### Bandit 4
---

**Problem**  
Anzeigen einer versteckten Datei im Terminal.

**Lösung**  
```bash
ls -a
```

**Erklärung**  
Dateien, deren Name mit einem Punkt beginnt, sind unter Linux versteckt. 
Mit `ls -a` bzw. `ls --all` werden alle, also auch versteckte, Dateien angezeigt.

### Bandit 5
---

**Problem**  
Anzeigen der einzigen (Menschen-) lesbaren Datei.

**Lösung**  
```bash
file ./*
```

**Erklärung**  
`file` gibt den Dateityp der angegebenen Datei aus.
`./*` meint dabei alle Dateien im aktuellen Ordner.
(`find -readable` listet nur Dateien auf, für die ein Benutzer Leserechte hat.)

**Alternative**  
'Einzige für menschen lesbare Datei' impliziert, dass die einzige Zeichenkette in allen Dateien das gesuchte Passwort ist -> `strings ./*`.

### Bandit 6
---

**Problem**  
Finden einer Datei im Ordner `dir/`, die menschenlesbar, 1033 Bytes groß und nicht ausführbar ist.

**Lösung**  
```bash
find dir/ -readable -type f -size 1033c \! -executable -exec file {} +
```

**Erklärung**  
`find dir` sucht nach Dateien mit den angegebenen Eigenschaften im angegebenen Ordner `dir`.
`-type f` spezifiziert den Dateityp auf 'file'.
`-readable` steht für Benutzer hat Leserechte.
`\!` steht für eine Negation des folgenden Arguments.
`-exec file {} +` führt auf der Liste der gefundenen Datei den Befehl `file` aus.
`-size ...c` steht für Größe in Character/Bytes.

**Gelernt**  
- Negation von Argumenten mit `\!`
- Ausführen von Befehlen auf die Ausgabe von `find`

### Bandit 7
---

**Problem**  
Finden einer Datei auf dem Server, die `user` und `group` gehört und 33 Bytes groß ist.

**Lösung**  
```bash
find / -type f -readable -user user -group group -size 33c 2>/dev/null
```

**Erklärung**  
`/` ist das Wurzelverzeichnis des Servers.
`stderr` nach `/dev/null` umleiten, damit die Ausgabe nur die gefundenen Dateien anzeigt, also alle Berechtigungsfehler verwirft.

### Bandit 8
---

**Problem**  
Finden einer Zeile mit bekanntem Wort.

**Lösung**  
```bash
grep 'word' < file
```

**Erklärung**  
`grep` gibt alle Zeilen aus die mit dem Argument übereinstimen.

### Bandit 9
---

**Problem**  
Finden der einzigen Zeile die nur einmal in der Datei vorkommt.

**Lösung**  
```bash
sort < file | uniq -u
```

**Erklärung**  
`uniq -u` gibt nur einzigartige Zeilen aus.
Vorher muss der Inhalt der Datei mit `sort` sortiert werden, da `uniq` nur bei aufeinander folgenden Zeilen feststellen kann, ob eine Zeile einzigartig ist.

### Bandit 10
---

**Problem**  
In einer Datei mit wenigen menschenlesbaren Zeilen nach einer Zeichenkette suchen, die auf mehrere '=' Zeichen folgt.

**Lösung**  
```bash
strings file | grep -E '={2,}'
```

**Erklärung**  
`strings` gibt alle Zeichenfolgen aus, die als Buchstaben interpretierbar sind.
`grep -E` nimmt alle Zeilen, die dem RegEx Ausdruck entsprechen.
`'={2,}'` sind alle Zeilen, die mind. 2 '=' Zeichen enthalten.

### Bandit 11
---

**Problem**  
Base64 codierte Datei decodieren.

**Lösung**  
```bash
base64 -d file
```

### Bandit 12
---

**Problem**  
Alle Zeichen einer Datei um 13 Stellen rotieren.

**Lösung**  
```bash
tr 'a-zA-Z' 'n-za-mN-ZA-M'
```

**Erklärung**  
`tr` für 'translate' übersetzt die Zeichen a-z in n-za-m (a + 13 Stellen = n) und macht das gleiche für die Großbuchstaben

**Gelernt**  
`tr` Mengenschreibweise:

| Syntax | Bedeutung |
|:---|:---|
| `a-z` | Bereich von `a` bis `z` |
| `[:alpha:]` | POSIX-Klasse |
| `\n` | Escape-Sequenz |
| `\123` | Oktale Angabe |
| `-` (Anfang/Ende/escaped) | `-` Zeichen |

### Bandit 13
---

**Problem**  
Ausgeben einer Datei, die mehrfach komprimiert wurde und als Hexdump vorliegt.

**Lösung**  
```bash
xxd -r file output
gunzip file.gz
bunzip2 file
tar -xf archive
```

**Erklärung**  
`xxd` Befehl für Hexdumps, `-r` um aus einem Hexdump die Datei wiederherzustellen.
`gunzip` um gzip Dateien, `.gz`, zu entpacken.
`bunzip2` um bzip2 Dateien (brauchen keinen Suffix) zu entpacken.
`tar -xf` um `tar` Archive zu entpacken.

### Bandit 14
---

**Problem**  
Herunterladen einer Datei vom Server und Verbinden mit einem Server über SSH ohne Passwort.

**Lösung**  
```bash
scp -P portnummer user@host:/pfad/zum/key . 
chmod 600 key
ssh user@host -p portnummer -i key
```

**Erklärung**  
`scp`, secure file copy, kopiert Dateien via SFTP über SSH.
`chmod` ändert die Berechtigungen auf `600`

**Gelernt**  
- `chmod` Berechtigungen können als drei Oktalzahlen für user, group und others angegeben werden
- Berechtigungen als Oktalzahl funktionieren als Summe von 4 (read) + 2 (write) + 1 (execute)

### Bandit 15

**Problem**  
Passwort an den Server unter `localhost:port` geben.

**Lösung**  
Als `bandit14`:
```bash
nc localhost port
```

**Erklärung**  
`nc` client Verbindung mit dem Server auf `localhost:port`.

### Bandit 16

**Problem**  
Passwort an einen Server unter `localhost:port` geben, der TLS Verschlüsselung nutzt.

**Lösung**  
Als `bandit14`
```bash
openssl s_client -connect localhost:portnummer
```

**Erklärung**  
`openssl s_client` ist ein generischer SSL/TLS client.

### ... (coming soon)