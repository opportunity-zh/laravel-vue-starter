# Befehl Sail alias erstellen
Wir können nun Laravel mit Docker starten, anstatt "docker compose up" zu verwenden, verwenden wir "./vendor/bin/sail up". Dieser Befehl startet die Docker-Container und bindet die Ports für die Verwendung in der lokalen Entwicklungsumgebung. Damit wir nicht dauernd den ganzen Pfad eingeben müssen, können wir einmalig ein Alias erstellen. Öffne die Datei ~/.bashrc oder ~/.bash_aliases und füge folgende Zeile hinzu:

```bash
alias sail='bash vendor/bin/sail'
```

Führe den Befehl aus, um die Änderungen zu übernehmen:

```bash
source ~/.bashrc
```