# Laravel als API und Vue.js als Frontend

## Beschreibung

Dieses Repository enthält eine Anleitung, wie Laravel als API eingerichtet wird. Zusätzlich wird ein Vue.js Frontend eingerichtet, welches die API nutzt.

Gehe Schritt für Schritt vor, um die Anwendung zu installieren. Falls unklarheiten bestehen, kannst du auch direkt in der entsprechenden Datei nachschauen.

Dieses Repository wurde nach Anleitung aufgebaut und getestet. Es sollte keine Probleme geben, wenn du dich an die Anleitung hältst.

## Eigene Installation

1. Backend installieren: [Setup Guide Laravel](/wiki/01-Setup-Guide-Laravel-API.md)
2. Frontend installieren: [Setup Guide Vue.js Frontend](/wiki/02-Setup-Guide-Laravel-Vue-Frontend.md)

## Repository direkt verwenden

Um das Repository zu verwenden, musst du vorher noch folgendes ausführen:

```bash
# php dependencies installieren
composer install
```

```bash
# npm dependencies installieren
npm install
```

```bash
# .env Datei erstellen
cp .env.example .env
```

```bash
# Laravel Key generieren
php artisan key:generate
```

```bash
# sail starten
./vendor/bin/sail up
```

```bash
# Datenbank migrieren
./vendor/bin/sail artisan migrate
```

```bash
# Sail starten
./vendor/bin/sail up
```

```bash
# vite starten
npm run dev
```
