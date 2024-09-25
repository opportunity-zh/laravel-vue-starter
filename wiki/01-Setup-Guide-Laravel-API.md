# Backend Setup Guide

Dieser Guide geht schrittweise durch die Installation und Konfiguration des Backends mit Laravel und MySQL.

Das Ziel ist es, eine lokale Entwicklungsumgebung aufzusetzen, die es ermöglicht, die Backend-Logik zu entwickeln und zu testen.

## Voraussetzungen

-   Docker, Docker Compose (https://docs.docker.com/get-docker/)
-   Composer sollte über den Terminal verfügbar sein (https://getcomposer.org/download/). Kann auch über Docker installiert werden, aber es empfiehlt sich eine lokale Installation für die Entwicklung zu haben.
-   Empfehlenswert: PHP 8.0 oder höher (https://www.php.net/downloads)
-   Empfehlenswert: Postman (https://www.postman.com/downloads/)


## Installation

Für ein besseres zusammenarbeiten installieren wir Laravel in einem Docker Container. Für Laravel gibt es ein offizielles Erweiterungspaket namens Sail, das die Installation und Verwaltung von Laravel in einem Docker-Container vereinfacht. Sail kann auch nachträglich installiert werden, aber es ist einfacher, es direkt bei der Installation zu verwenden. Es installiert Laravel mit MySQL und Redis und konfiguriert alles für die Verwendung mit Docker.

Dokumentation: https://laravel.com/docs/11.x/sail

### 1. Laravel mit Docker (Sail) installieren
Laravel mit Sail Docs: https://laravel.com/docs/11.x/installation#docker-installation-using-sail
Wir verwenden den CURL-Befehl für Linux, um Sail zu installieren. Führe den folgenden Befehl im Terminal aus:

```bash
# Ersetze example-app durch den Namen deines Projekts, z.B. mini-twitter. Dieser Name wird auch für den Docker-Container verwendet und ein Ordner mit diesem Namen wird erstellt.
curl -s https://laravel.build/example-app | bash
```

Wechsle in das Projektverzeichnis:

```bash
# ersetze example-app durch den Namen deines Projekts (z.B. mini-twitter)
cd example-app
```

### 2. phpMyAdmin hinzufügen
Wir möchten noch phpMyAdmin hinzufügen, um die Datenbank zu verwalten. Füge die folgende Zeile in die Datei docker-compose.yml ein. Achte darauf, dass die Einrückung (Tab) korrekt ist. Füge die Zeile unterhalb von MySQL hinzu:

```yaml
phpmyadmin:
    image: phpmyadmin/phpmyadmin
    ports:
        - 8080:80
    links:
        - mysql
    environment:
        PMA_HOST: mysql
        PMA_PORT: 3306
    depends_on:
        mysql:
            condition: service_healthy
    networks:
        - sail
```

### 3. Docker Container starten

#### Befehl Sail alias erstellen
Überspringe diesen Schritt, wenn du den Alias bereits erstellt hast.
[Sail alias erstellen](/wiki/Sail-Befehl-Bash.md)

#### Befehl sail up ausführen

Jetzt können wir Sail verwenden, um die Docker-Container zu starten:

```bash
# Dieser Befehl startet "docker compose up" und bindet die Ports für die Verwendung in der lokalen Entwicklungsumgebung
sail up
```

Jetzt sollte unter http://localhost:80 die Laravel-Startseite erscheinen. Unter http://localhost:8080 kann phpMyAdmin aufgerufen werden. Womöglich ist eine Error-Seite unter http://localhost:80 zu sehen.

#### Permissions Problem

Bei Linux könnte es sein, dass die Berechtigungen für die Dateien nicht korrekt sind. Das könnte an Docker Desktop für Linux liegen. Lese die Error-Meldung, siehst du im Pfad "/var/www/html/storage/..." und den Fehler "Permission denied", dann sind die Berechtigungen nicht korrekt. Um die Berechtigungen zu korrigieren, führe den folgenden Befehl aus:

```bash
sudo chmod -R 777 storage
```

Öffne den Browser nochmals und lade die Seite neu. Die Error-Seite sollte verschwinden und eine neue Meldung erscheinen, dass die Datenbank nicht gefunden wurde. Das ist in Ordnung, da wir noch keine Datenbank erstellt haben, bzw. die Datenbank Konfigurationen noch nicht migriert wurden.sailt Ports können auftreten, wenn die Ports bereits von anderen Anwendungen verwendet werden. In diesem Fall diese Applikationen beenden und die Ports wieder freigeben.

Schaue die Error-Meldungen an und versuche die Ports herauszufinden, die bereits verwendet werden. Mit dem Befehl lsof können wir laufende Prozesse und die Ports herausfinden, die sie verwenden:

```bash
sudo netstat -tulpen | grep :<PORT>
```

Finde den Prozess, der den Port verwendet übertrage die PID in den foglenden Befehl, um den Prozess zu stoppen. Die PID befindet sich in der letzten Spalte, die Nummer vor dem Namen des Prozesses.

```bash
sudo kill <PID>
```
## API aktivieren/installieren

Laravel 11 wird im Standard Konfiguration klein gehalten, gewisse Funktionen und Datein müssen wir selber aktivieren/hinzufügen. Um die API route Datei zu aktivieren, schreibe folgenden command in die Konsole:

```bash
php artisan install:api
```

***Version Info: Mit diesem Command wird ab Laravel 11 auch gerade die API-Route aktiviert und Sanctum installiert, bei früheren Versionen muss Sanctum noch manuell installiert werden.***

Die Datei `routes/api.php` wird erstellt und die API-Route wird aktiviert.

Die Datei `routes/web.php` ist wird nur für Blade-Views verwendet und wird nicht benötigt. (Optional: Kann ignoriert werden, wenn das Frontend mit Vue.js oder React.js decoupled von Laravel entwickelt wird.)

### API routes aktivieren

Prüfe ob folgende Zeile in in der Datei `bootstrap/app.php` bei `withRouting` hinterlegt ist:

```php
    ->withRouting(
        web: __DIR__ . '/../routes/web.php',
        api: __DIR__ . '/../routes/api.php', // to enable api routes
        commands: __DIR__ . '/../routes/console.php',
        health: '/up',
    )
```
***Version Info: Mit Laravel 11 wird die API-Route automatisch aktiviert, bei früheren Versionen muss die API-Route manuell aktiviert werden.***

### Test API-Route erstellen

Für das Testen der API-Route erstellen wir eine einfache Route, die eine JSON-Antwort zurückgibt. Füge folgende Zeilen in die Datei `routes/api.php` ein, lasse die anderen Zeilen unverändert (Request und Facades\Route sollten bereits importiert sein):

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

// TODO: remove this on public release, only for testing!
Route::get('/test', function () {
    return response()->json(['message' => 'Hello World!'], 200);
});
```

#### Laravel Command um alle Routes zu sehen

Wir können diesen Artisan Command verwenden, um alle Routes zu sehen.Praktisch um für die Übersicht zu behalten, welche Routes wir schon aktiv haben, andere Erweiterungen oder Module könnten schon eigene Routes registriert haben.

```bash
sail artisan route:list
```

Die /api/test Route sollte in der Liste erscheinen.

Rufe die Route auf http://localhost/api/test auf und prüfe, ob die Antwort `{"message":"Hello World!"}` ist.

### Sanctum Konfiguration

Sanctum ist ein einfaches Paket zur Authentifizierung von APIs. Es verwendet Tokens, um Benutzer zu authentifizieren. Dokumentation: https://laravel.com/docs/11.x/sanctum

#### Sanctum Konfiguration

Mit Sanctum können wir uns entscheiden, ob wir Cookies oder Tokens verwenden möchten. Für dieses Projekt verwenden wir die Session-Cookies Authentifizierung. Das bedeutet, dass wir uns mit der API über Cookies authentifizieren. 

***Alternative Info: Die andere Methode wäre, dass wir uns mit einem Token authentifizieren, der in der Datenbank gespeichert wird und wir bei jeder Anfrage mitsenden.***

1. Füge folgende Zeilen in deine .env-Datei hinzu, es sollte schon ein Berreich `SESSION_` vorhanden sein:

```bash
# for sanctum to work with localhost or your custom domain
SESSION_DOMAIN=localhost
SESSION_SECURE_COOKIE=true
SANCTUM_STATEFUL_DOMAINS="localhost"
```

2. Manchmal wird der Konfigurationscache nicht aktualisiert, wenn wir Änderungen an der Konfiguration vornehmen. Um eine Aktualisierung zu erzwingen, führe den folgenden Befehl aus:

```bash
sail artisan config:clear
```


3. Jetzt werden die Cors-Konfigurationen angepasst. Cors (Cross-Origin Resource Sharing) ist ein Mechanismus, der es Webseiten ermöglicht, Ressourcen von einer anderen Domain zu laden. Wenn diese Domains nicht übereinstimmen, wird die Anfrage blockiert. 
Laravel Docs: https://laravel.com/docs/11.x/sanctum#spa-configuration
Mozilla Docs, was sind cors: https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS

Zuerst müssen wir die Cors-Konfigurationen für uns Entwickler veröffentlichen:

```bash
# Veröffentliche die Cors-Konfigurationsdatei
php artisan config:publish cors
```

4. Öffne die Datei `config/cors.php` und prüfe die folgende Zeile "paths":

```php
# lässt Anfragen von der API zu und die CSRF-Cookie-Route
'paths' => ['api/*', 'sanctum/csrf-cookie'],
```

Die `sanctum/csrf-cookie` Route ist wichtig, damit die CSRF-Cookies gesetzt werden können.

Zusätzlich möchten wir hier auch unsere Frontend-Domain hinzufügen, damit die Anfragen von der Frontend-Domain akzeptiert werden. Füge die Frontend-Domain in die `allowed_origins` hinzu:

```php
'allowed_origins' => [env('FRONTEND_URL', 'localhost:5173')],
```
In der .env-Datei fügen wir die Frontend-Domain hinzu, am besten direkt unterhalb der `APP_URL`:

```bash
FRONTEND_URL=localhost
```

5. Jetzt müssen wir noch die Middleware für Sanctum konfigurieren. Öffne die Datei `bootstrap/app.php` und füge die `$middleware->group('api', ...)` Middleware hinzu (sie muss in die `withMiddleware` Funktion hinzugefügt werden):

```php
use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__ . '/../routes/web.php',
        api: __DIR__ . '/../routes/api.php',
        commands: __DIR__ . '/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        /*
            for session cookie authentication we need to add this here.
            src: https://laravel.com/docs/11.x/middleware#manually-managing-laravels-default-middleware-groups
        */
        $middleware->group('api', [
            \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
            // 'throttle:api',
            // \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();

```
Jetzt ist Sanctum konfiguriert und wir können die Authentifizierung mit Cookies verwenden. Wir können auch die API-Route testen, um sicherzustellen, dass die Authentifizierung funktioniert. (Die Routes mit der Middleware `auth:sanctum` sind geschützt und benötigen eine Authentifizierung und sollten eine 401 Unauthorized zurückgeben, wenn keine Authentifizierung vorhanden ist.)


## Login und Registriung API einrichten
Für die Registrierung und das Login müssen wir die Authentifizierung konfigurieren. Das können wir mit einer eigen Logik machen oder mit dem Laravel-Paket Fortify. Fortify ist ein Paket, das die Authentifizierung von Benutzern vereinfacht. Es bietet verschiedene Funktionen, um die Authentifizierung zu konfigurieren. Dokumentation: https://laravel.com/docs/11.x/fortify

### Fortify Installation

```bash
# Installiere Fortify
composer require laravel/fortify
```

Installiere den Fortify Service Provider:

```bash
php artisan fortify:install
```

#### Fortify Konfiguration

Da wir nur die API verwenden, können wir die Blade-Views deaktivieren und den Prefix für die API-Routes ändern. Öffne die Datei `config/fortify.php` und ändere die folgenden Zeilen:

```php
// find an change prefix to /api
'prefix' => '/api',

// find and change views to false
'views' => false,
```


### Migrations

Unter `database/migrations` befinden sich die Migrations-Dateien, die die Tabellen für die Datenbank erstellen oder ändern. Damit die Konfigurationen in die Datenbank migriert werden, führe den folgenden Befehl aus:

```bash
sail artisan migrate
```
***Info: Weil wir Sail (Docker) verwenden, müssen wir es mit Sail ausführen, da wir diesen Befehl innerhalb des Docker-Containers ausführen möchtenn. Anderenfalls würde der Befehl `php artisan migrate` funktionieren, wenn wir Laravel lokal installiert haben.***

Prüfe nun auf http://localhost, ob die Laravel-Startseite erscheint. Wenn ja, dann ist Laravel erfolgreich installiert und konfiguriert.



## Postman für das Testen der API einrichten

Wir erstellen in Postman eine Collection, um die API-Route zu testen. Die Collection enthält die API-Route, die wir erstellt haben. Die Collection kann exportiert und geteilt werden, um anderen Entwicklern zu helfen, die API zu testen.

1. Öffne Postman und erstelle eine neue Collection.
2. Füge eine neue Request hinzu und gib der Request einen Namen.
3. Wähle die Methode GET aus und füge die URL `http://localhost/api/test` hinzu.
4. Führe die Request aus und prüfe, ob die Antwort `{"message":"Hello World!"}` ist.

Super! Deine API-Route funktioniert und du kannst nun mit der Entwicklung beginnen.

### Postman Registrierung und Login erstellen

Fortify nimmt uns die Arbeit ab, die Authentifizierung zu konfigurieren. Wir verwenden die bereits vorhandenen Fortify routes `/api/register` und `/api/login` für die Registrierung und das Login.

#### CSRF-Token

Laravel verwendet CSRF-Token, um Cross-Site Request Forgery (CSRF) zu verhindern. Wir müssen den CSRF-Token in Postman bzw. in unserer Applikation speichern und bei jeder Anfrage mitsenden die in den Routes mit Sanctum geschützt sind.

1. Erstelle eine neue Enviroment Variable in Postman für deine backend_url: `http://localhost`.
2. Erstelle eine neue Environment Variable in Postman und füge `xsrf-token` hinzu.
3. Speichere die Environment Variable.
4. Setze die neue Enviroment Config für die Collection.
   ![Postman Environment](./media/postman-environment.png)

5. Füge folgendes bei deinem Postman Request unter Pre-request Script hinzu:

```javascript
pm.sendRequest(
    {
        url: `${pm.environment.get("backend_url")}/sanctum/csrf-cookie`,
        method: "GET",
    },
    function (error, response, { cookies }) {
        if (!error) {
            pm.environment.set("xsrf-token", cookies.get("XSRF-TOKEN"));
        }
    }
);
```

6. Wichtig, bei jedem Request der Sanctum benötigt, muss der CSRF-Token mitgesendet werden. Füge folgendes unter Headers hinzu:

```
X-XSRF-TOKEN: {{xsrf-token}}
```

7. Zusätzlich mit dieser Architektur in Laravel müssen wir noch den Referer Header hinzufügen.

```
Referer: {{backend_url}}
```

![Postman Login](./media/postman-login.png)

Wichtig, ohne Doppelpunkte übernehmen!

#### Registrierung

1. Erstelle eine neue Request in Postman und füge die URL `{{backend_url}}/api/register` hinzu (die Environment Variable wird automatisch in den gescheiften Klammern ersetzt).
2. Wähle die Methode POST aus und füge die Body-Parameter `name`, `email`, `password` und `password_confirmation` hinzu.
3. Im untermenü Headers füge `Accept` und `application/json` hinzu. Damit wir JSON-Daten als Antwort erhalten.
4. Schaue auch, dass der CSRF-Token mitgesendet wird.
5. Senden den Request ab und prüfe, was die Antwort ist. Sollte einfach aktuell ein 201 Created sein.

Prüfe in der Datenbank, ob der Benutzer erstellt wurde.
Geh in phpMyAdmin `http://localhost:8080` und prüfe die Datenbank `laravel` und die Tabelle `users`. Der Benutzer sollte erstellt worden sein.

#### Login

1. Erstelle eine neue Request in Postman und füge die URL `{{backend_url}}/api/login` hinzu.
2. Wähle die Methode POST aus und füge die Body-Parameter `email` und `password` hinzu.
3. Im untermenü Headers füge `Accept` und `application/json` hinzu. Damit wir JSON-Daten als Antwort erhalten.
4. Schaue auch, dass der CSRF-Token mitgesendet wird.
5. Senden den Request ab und prüfe, was die Antwort ist. Sollte einfach aktuell ein 200 OK sein.

## Vue Setup

[Setup Guide Vue.js Frontend](/wiki/02-Setup-Guide-Laravel-Vue-Frontend.md)
