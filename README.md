# Backend

Das Wanderlust-Backend ist ein Web-Service zur Verwaltung von Usern, Locations und Administratoren, sowie entsprechenden Abhängigkeiten. Es wird sowohl von der Android-App als auch vom Administations-Frontend verwendet. Ansprechbar ist es über eine REST-konforme Schnittstelle.

## Anleitung

Um das Bakend lokal auf einem Rechner auszuführen können die folgenden Schritte ausgeführt werden. Die Dokumentation geht dabei davon aus, dass alle Komponenten des Backends auf dem Rechner selbst ausgeführt werden. Um beispielsweise eine deployte Datenbank zu verwenden muss die entsprechende Konfigirationsdatei angepasst werden. Auch für ein Deployment in Production müssen Konfigurationen durchgeführt werden.

Bevor das Backend ausgeführt werden kann müssen folgende Abhängigkeiten installiert werden:

* Node.js
* NPM
* MongoDB

Daraufhin kann das Backend installiert werden:

1. Das Repository klonen.
2. `npm install` ausführen.

Anschließend kann das Backend gestartet werden:

1. Stelle sicher, dass die MongoDB-Instanz läuft. 
2. Führe `npm start` aus.

Dabei wird, falls noch nicht vorhanden, ein Administrator mit dem Benutzernamen _admin_ und dem Passwort _1234_ angelegt. Die API-Dokumentation befindet sich unter  [http://localhost:3000/docs](http://localhost:3000/docs).

## Architektur

Das Design des Backends orientiert sich an folgenden Eigenschaften, die erreicht werden sollen:

- REST-Konformität
- Sicherheit
- Skalierbarkeit

Bei dem Backend handelt es sich um eine Node.js-Anwendung. Für die Definition von Endpunkten wird Express.js eingesetzt. Als Datenbank kommt MongoDB zum Einsatz und zur Authentifizierung wird Passport.js verwendet. Weitere verwendete Technologien werden im Laufe dieser Dokumentation vorgestellt.

### Projektstruktur

```
├───config
├───node_modules
├───src
│   ├───middleware
│   │   └───auth
│   ├───models
│   ├───routes
│   │   ├───locations
│   │   └───users
│   └───schemas
└───uploads
    └───imgs
        ├───location
        └───user
```

Im Root-Verzeichnis befinden sich Konfigurationsdatein für Editor Config und ESLint, sowie die Swagger-Spezifikation, eine .gitignore-Datei, die package.json und package-lock.json.

Unter */config* werden alle Konfigurationsdateien abgelegt. Diese werden mit Ausnahme der *default.json* nach der Umgebung benannt, für die sie gelten sollen.

Unter */node_modules* installiert npm die in der package.json definierten Dependencies.

Im */src*-Verzeichnis befindet sich die tatsächliche Logik des Backends.

Unter */src/middleware* befindet sich die projektspezifische Middleware, also solche, die nicht über npm-Module installiert werden.

Unter */src/models* befinden sich die Mongoose-Models mit ihren Schemata. Unter */src/routes* werden die Enpunkte definiert. Im Verzeichnis */src/schemas* befinden sich Joi-Schemata für die Request-Validierung.

Im Repository befindet sich zwar kein Verzeichnis */uploads*, dieses wird aber angelegt, sobald Bilder hochgeladen werden. Hier werden alle Medien, die von Nutzern hochgeladen werden, gesichert.

### Schnittstellen

Die Schnittstellen der API werden als [OpenAPI-Spezifikation](https://github.com/OAI/OpenAPI-Specification) dokumentiert, die sich in der Datei *swagger.yml* im Root-Verzeichnis des Projekts befindet. Ein UI zu dieser Spezifikation ist unter dem Endpunkt */docs* zu erreichen, lokal also unter  [http://localhost:3000/docs](http://localhost:3000/docs). Hier können alle Informationen über die Endpoints eingesehen werden, sowie Requests ausprobiert werden. Um dieses UI bereitzustellen wird die Middleware [swagger-ui-express](https://github.com/scottie1984/swagger-ui-express) eingesetzt. Im Gegensatz zur offiziellen UI-Middleware kommt diese ohne manuell aufzulösende Abhängigkeiten aus. Da die sich die Spezifikation im Repository befindet enthält jede Version des Backends immer eine passende Dokumentation ihrer API.

| Endpoint                                 | Method | Description                              |
| ---------------------------------------- | ------ | ---------------------------------------- |
| `/users`                                 | GET    | Find users by criteria specified as query parameters. Admins can omit those criteria, users have to specify them. |
| `/users`                                 | POST   | Create user. Users can only create profiles linked to their own google account. |
| `/users/{userId}`                        | GET    | Get user. Users can only access their own profile. |
| `/users/{userId}`                        | PATCH  | Update user. Users can only update their own profile. |
| `/users/{userId}`                        | DELETE | Delete user. Users can only delete their own profile. |
| `/users/{userId}/friends`                | GET    | Get user's friends. Users can only access their own friends. |
| `/users/{userId}/friends/{friendId}`     | PUT    | Add friend. This will share the user's locations with the added friend. The friend does not automatically share their locations with the user. Users can only share their own locations. |
| `/users/{userId}/friends/{friendId}`     | DELETE | Remove friend.                           |
| `/users/{userId}/avatar`                 | PUT    | Upload avatar                            |
| `/users/{userId}/avatar`                 | DELETE | Remove Avatar                            |
| `/locations`                             | GET    | Get the locations of the users specified as query parameters |
| `/locations`                             | POST   | Add new location                         |
| `/locations/{id}`                        | PATCH  | Update location                          |
| `/locations/{id}`                        | DELETE | Delete Location                          |
| `/locations/{id}/images`                 | POST   | Upload location images                   |
| `/location/{locationId}/images/{imageId}` | DELETE | Delete Image                             |

Definiert werden die Schnittstellen mithilfe von Express.js. Die entsprechenden Routen befinden sich im Verzeichnis */src/routes*. Bei der Definition der Endpunkte wird großer Wert auf REST-Konformität gelegt. Die HTTP-Methoden werden dabei so genutzt wie es ihre Spezifikationen vorsehen:

**GET** wird verwendet um Ressourcen vom Backend anzufordern. Es werden dabei keine Ressourcen geändert.

**POST** wird verwendet um neue Ressourcen anzulegen. Dabei legt der Server fest, unter welcher URL die Ressource erreichbar sein wird. Die Daten werden über den Request-Body übermittelt. Wiederholtes Senden desselben Requests führen dabei jedes Mal zu einem anderen Ergebnis, da dieselbe Ressource unter verschiedenen IDs abgespeichert wird.

**PUT** hingegegen wird verwendet, wenn eine Ressource an der durch die URL definierte Stelle erzeugt werden soll. Damit können neue Ressourcen angelegt werden, aber auch komplette Ressourcen ausgetauscht werden.

**PATCH** kann verwendet werden, wenn Änderungen an einer Ressource vorgenommen werden sollen, ohne die gesamte Ressource auszutauschen. Dabei werden die Daten im Format [JSON Merge Patch](https://tools.ietf.org/html/rfc7386) übergeben. Dieses Format hat einige Nachteile gegenüber dem [JSON Patch](https://tools.ietf.org/html/rfc6902)-Format. So müssen beispielsweise um Daten zu Arrays hinzuzufügen extra Subroutes angelegt werden. Dafür ist bleibt der Request intuitiver und übersichtlicher. Der Content-Type-Header sollte auf *application/merge-patch+json* gesetzt werden um explizit auszudrücken, dass dieses Format verwendet wird. Dadurch wird auch die Möglichkeit offen gehalten in Zukunft andere Formate zu unterstützen, ohne die Backwards-Compability der API zu brechen.

**DELETE** wird verwendet um Ressourcen zu löschen.

### Modelle

Für die Datenhaltung kommt [MongoDB](https://www.mongodb.com/) zum Einsatz. Diese wird durch Mongoose in das Backend eingebunden, wodurch erweiterte Funktionalitäten zur Verfügung stehen und - besonders wichtig - Schemas definiert werden können. Im Backend gib es drei Datenmodelle: Admin, User und Location. Im Verzeichnis */src/models* werden diese Modelle definiert.

#### Admin

| field    | type   | required | default | ref  |
| -------- | ------ | -------- | ------- | ---- |
| username | String | true     | -       | -    |
| password | String | true     | -       | -    |

Eine Besonderheit der Admin-Collection ist, dass dort auch Passwörter gehalten werden. Dies kann nicht in Form von Klartext geschehen. Vor dem Speichern der Paswörter muss also ein Einweg-Hash erzeugt werden, gegen das dann andere Passwörter verifiziert werden können. Dies geschieht im _pre save_ Hook des Schemas. Zur Generierung des Salt-Wertes und des Hashes wir die Bibliothek [bcrypt](https://github.com/kelektiv/node.bcrypt.js) verwendet. Außerdem wird das Schema um eine Methode erweitert, mit welcher Passwörter validiert werden können. Auch hierfür kommt *bcrypt* zum Einsatz.

#### User

| field     | type       | required | default | ref      |
| --------- | ---------- | -------- | ------- | -------- |
| googleId  | String     | true     | -       | -        |
| firstName | String     | true     | -       | -        |
| lastName  | String     | true     | -       | -        |
| email     | String     | true     | -       | -        |
| locations | [ObjectId] | false    | -       | Location |
| friends   | [ObjectId] | false    | -       | User     |
| avatar    | String     | false    | -       | -        |

Da jede Location einem User zugeordnet ist und ein User mehrere Locations besitzen kann, muss darauf geachtet werden, dass beim Erzeugen der Location, die entsprechende ID im User-Dokument eingetragen wird und dass diese beim Löschen der Location wieder entfernt wird. Beim Hinzufügen eines Freundes muss darauf geachtet werden, dass die Freunde eines Users ausdrücken, welche anderen User ihre Orte für ihn freigegeben haben. Wenn man eine Gegenseitige Freundschaft anlegen will, müssen beide Freundeslisten erweitert werden. Um Freundschaften zu entfernen müssen beide Nutzer aus der jeweilig anderen Liste entfernt werden.

#### Location

| field       | type       | required | default | ref  |
| ----------- | ---------- | -------- | ------- | ---- |
| name        | String     | true     | -       | -    |
| user        | ObjectId   | true     | -       | User |
| description | String     | false    | ''      | -    |
| favorite    | Boolean    | false    | false   | -    |
| googleId    | String     | true     | -       | -    |
| coordinates | [Number]   | true     | -       | -    |
| city        | String     | true     | -       | -    |
| country     | String     | true     | -       | -    |
| images      | [ObjectId] | false    | -       | -    |
| categories  | [String]   | false    | -       | -    |

## Authentifizierung und Autorisierung

Die Endpunkte werden so gesichert, dass sie nur von registrierten Nutzern und Administratoren angesprochen werden können (Authentifizierung). Aber auch angemeldete Nutzer dürfen nicht jeden der Endpunkte mit beliebigen Parametern ansprechen (Autorisierung). Die gesamte Authentifzierung und Autorisierung geschieht in den Middleware-Funktionen, die sich unter */src/middleware/auth* befinden. Die Authentifizierung geschieht dabei mithilfe der Authentifizierungs-Middleware Passport.js. Darauf baut dann die Autorisierung auf.

Eine wichtige Middleware-Funktion ist dabei die Methode `init`. In dieser werden Passport.js und verschiedene Strategien initialisiert. Es wird außerdem ein Default-Admin angelegt, falls dieser noch nicht existiert. Dabei wird eine Strategie verwendet um Nutzer anhand ihres Google Tokens zu identifizieren. Dafür wird die Strategie [GoogleIdToken](https://github.com/jmreyes/passport-google-id-token) verwendet. Um Administratoren anhand von Nutzernamen und Passwort zu authentifizieren wird die Strategie [BasicAuthentication](https://github.com/jaredhanson/passport-http) verwendet. Für beide Strategien müssen `verify`-Funktionen implementiert werden. In dieser Funktion wird überprüft ob ein solcher Nutzer existiert und gegebenenfalls das Passwort überprüft. Anschließend wird dann dieser Nutzer auf ein User-Objekt gemapt, das dann an das Request-Objekt angehängt wird, sodass in folgender Middleware darauf zugegriffen werden kann.

Strategien, die dafür genutzt werden Endanwender zu authentifizieren können diese Nutzer auch auf Objekte mit denselben Eigenschaften mappen. Das user-Objekt muss aber auch Administratoren repräsentieren können, sowie Nutzer, die zwar einen gültigen Google-Account haben, sich aber noch nicht für Wanderlust registriert haben. Deshalb werden die User-/Admin-Dokumente aus der Datenbank um ein `type`-Attribut erweitert. Für User, die noch nicht in der Datenbank zu finden sind wird ein Objekt verwendet, das nur dieses `type`-Attribut sowie die googleId besitzt. Die drei verschiedenen User-Types ermöglichen in folgender Middleware die Autorisierung basierend auf den Type sowie der Eigenschaften des Users.

Folgende Typen werden dabei verwendet:

- user_candidate: Es handelt sich um einen validen Google User, allerdings wurde er noch nicht für Wanderlust registriert. Dieser type spielt nur für die Registrierung neuer Nutzer eine Rolle.
- user: Es handelt sich um einen validen und für Wanderlust registrierten Google User. Für alle Interaktionen der App mit der API ist dies der Fall, mit Ausnahme der Nutzer-Registrierung.
- admin: Es handelt sich um einen Administrator, der sich erfolgreich mit Nutzernamen und Passwort authentifiziert hat. Ein Administrator kann alle Endpunkte ansprechen und hat meehr Freiheiten, was die Wahl der einzelnen Parameter betrifft. API-Anfragen mit Administratoren-Rechten kommen normalerweise aus dem Frontend, können aber auch über die Swagger UI durchgeführt werden.

Um eine Route mithilfe der Authentifizierung zu schüzten, wird die Middleware `authenticate` verwendet. Diese stellt einen einfachen Wrapper um die `authenticate`-Middleware von Passport.js dar. Dabei werden Sessions deaktiviert, sodass dies nur an einer Stelle geschehen muss. Sessions werden deaktiviert, da sich das schwer mit REST vereinbaren lässt.

Die anderen Middleware-Funktionen im Verzeichnis */src/middleware/auth* können dann zur Autorisierung verwendet werden. Mit diesen kann festgelegt werden, welche User-Types Zugriff auf die Route bekommen sollen, sowie Anforderungen an die Request-Parameter definiert werden. Dabei wird immer ein Pfad angegeben unter dem der Parameter im Request-Objekt zu finden ist, sodass dieselbe Middleware verwendet werden kann um den Request-Body sowie Query-Parameter und URL-Parameter zu überprüfen.

Middleware, die verwendet wird um Request-Parameter zu überprüfen, sollte immer nach der Celebrate-Middleware verwendet werden, da diese zum einen die Request-Parameter noch verändern kann, zum anderen aber auch alle Parameter vorhanden sind, die von der Middleware erwartet werden.

## Datei-Uploads

## Notifications

## JavaScript Style Guide

Um konsistenten und hochwertigen Code zu schreiben verwendet das Backend (wie auch das Frontend) den [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript). Dieser beschreibt verschiedene Regeln, wie der Code geschrieben wird. Um die Einhaltung dieser Regeln zu erzwingen, verwenden wir [ESLint](https://github.com/eslint/eslint). Dieses kann mithilfe von `npm run lint` ausgeführt werden. Um zu verhindern, dass vor einem Commit vergessen wird, dieses Skript auszuführen (wodurch inkonsistenter Code im Repository landen kann) ist dieses Skript am Pre-Commit-Hook registriert worden. Das bedeutet, dass vor jedem commit die Einhaltung des Style Guides überprüft wird und nur im Erfolgsfall der tatsächliche Commit ausgeführt wird.

## Request-Validierung

Um fehlerhafte oder boshafte Requests an die API zu verhindern müssen alle Requests validiert werden. Fehlerhafte Query-Parameter, URL-Parameter oder Body-Parameter müssen zu einem `400` Status Code führen, oder falls möglich die Daten auf die richtige Form gebracht werden. Solch ein Whitelisting sorgt dafür, dass Request-Daten direkt an die MongoDB weitergegeben werden, ohne dass die Integrität unserer Daten gefährdet wird. Hierbei geht es nur um eine Validierung unabhängig vom Nutzer. Die Berechtigung des Nutzers wird in einer weiteren Middleware überprüft.

Für diese Validierung bedienen wir uns bei [Hapi](https://github.com/hapijs/hapi), einer von Walmart entwickelten Alternative zu Express. Dieses beinhaltet die Bibliothek [Joi](https://github.com/hapijs/joi), mit welcher Validierungsschemata beschrieben werden können und bei der Registrierung von Routes angegeben werden können. Um Joi in Express nutzen zu können, kommt die Middleware [celebrate](https://github.com/continuationlabs/celebrate) zum Einsatz.

## Konfiguration

Die Konfiguration der Anwendung geschieht über die Bibliothek [config](https://github.com/lorenwest/node-config). 

Das Backend lässt sich über Konfigurationsdateien im Verzeichnis `/config` konfigurieren. Dabei können für jede Environment eigene Konfigurationsdateien erstellt werden, die dann entsprechende Werte aus der Datei `default.json` überschreiben. Dabei muss nur darauf geachtet werden, den Namen der Konfiguration  entsprechend der gewünschten environment zu benennen. Sie wird dann verwendet, wenn dieser Environment-Name als `NODE_ENV`-Umgebungsvariable gesetzt wird. Da wir das Backend bisher noch nicht deployed haben, ist bisher nur die `default.json` vorhanden.

## Logging

Alle Requests, die vom Backend bearbeitet werden, werden geloggt. Dafür kommt die Middleware [morgan](https://github.com/expressjs/morgan) zum Einsatz. Das Format kann dabei über `logging.format` in der Konfigurationsdatei des Backends angepasst werden, sodass für verschiedene Stages verschiedene Formate definiert werden können. Morgan unterstützt dabei sowohl vordefinierte auch benutzerdefinierte Formate.

Zusätzlich werden auch bestimmte Informationen, wie der verwendete Port und die verwendete Datenbank geloggt, sowie verschiedene Server-Fehler. Dabei wird [chalk](https://github.com/chalk/chalk) verwendet, um die Logs farblich passend hervorzuheben.

## Ausblick

Während das Backend in seiner derzeitigen Form funktionstüchtig ist und auch Sicherheit und REST-Konformität gewährleistet, gibt es noch Ansatzpunkte, welche von technischer Seite, zu einer besseren Anwendung führen könnte.

### Authentifizierung über Authorization-Header

[RFC 7235](https://tools.ietf.org/html/rfc7235) definiert den Authorization-Header als den passenden Header um Credentials mit einem Request zu verschicken. Für Basic Authentication wird dieser Header in unserem Projekt auch verwendet, die Passport.js-Strategie, mit welcher wir die Google-Authentifizierung umsetzen, berücksichtigt allerdings den Authorization-Header nicht. Stattdessen muss ein seperater Custom Header verwendet werden. Wir haben deshalb die Bibliothek um die gewünschte Funktionalität erweitert, sodass sie auch Bearer Token, die sich im Authorization Header befinden, berücksichtigt. Der entsprechende [Pull Request](https://github.com/jmreyes/passport-google-id-token/pull/26) muss allerdings noch bestätigt werden. Sobald dies der Fall ist kann das Projekt entsprechend angepasst werden, sodass jede Form von Authentifizierung über den Authorization Header abläuft.

### Response-Validierung

Bisher wird Joi verwendet um Requests zu validieren und damit unerwünschte Requests zu unterbinden. Auf dieselbe Weise könnten auch Responses validiert werden, um beispielsweise zu verhindern, dass sensible Daten aufgrund eines Fehlers in einem Select-Statement oder durch eine ausgegebene Fehlermeldung preisgegeben werden. Inwiefern dies durch `celebrate`, der Middleware, die Joi in unser Projekt einbindet, unterstützt wird, müsste noch untersucht werden.

### File Store

Bisher speichern wir Bilder von Orten und Endanwendern auf dem Dateisystem des Servers ab, auf dem die Anwendung ausgeführt wird. Da angedacht ist mehrere Instanzen des Backends zu deployen, müssten die Bilder tatsächlich auf einem File Store Server abgespeichert werden. Hierfür gibt es meherere Alternativen.

### Deployment

Um das Backend in Produktion einzusetzen, muss es deployed werden. Um mit wechselnder Last zurecht zu kommen, müssten dynamisch mehrere Instanzen der Anwendung erzeugt werden. Dafür gibt es verschiedene Möglichkeiten, wenn man auf Cloud-Angebote zurückgreift, ist dies auch ohne großen Aufwand möglich. Da unsere Anwendung bis auf den oben schon genannten fehlenden File Storage zustandslos ist, lässt sie sich beliebig skalieren. Dafür müsste nur ein Load Balancer vorgeschaltet werden.

Um das Backend in Produktion einzusetzen müsste noch eine neue Konfigurationsdatei mit dem Namen production.json angelegt werden und auf dem Server die Umgebungsvariable `NODE_ENV=production` gesetzt werden.

### Google Token Validation

Jeder Request, der einen Google-Token mitschickt, wird über eine Schnittstelle von Google verifiziert. Um die Netzwerklast zu reduzieren und mögliche Beschränkungen der Google-Schnittstelle zu umgehen, könnten diese Anfragen gecached werden. Dafür müsste eine getGoogleCerts-Funktion an die Strategie übergeben werden, welche dieses Verhalten umsetzt.

# Administrations-Oberfläche

Die Administrationsoberfläche bietet Administratoren die Möglichkeit, Daten der Nutzer und Orte einzusehen und zu bearbeiten. So können beispielsweise Nutzer gelöscht werden, Freundschaften beendet werden oder Profile bearbeitet werden.

Es handelt sich dabei um eine Vue.js-Anwendung. Die Anwendung wurde mit dem [vue-cli](https://github.com/vuejs/vue-cli) erzeugt. Als Template wurde [webpack](https://github.com/vuejs-templates/webpack) verwendet. Informationen über die Projektstruktur können in der Dokumentation gefunden werden. Das Projekt wurde dann durch das Modul [bootstrap-vue](https://github.com/bootstrap-vue/bootstrap-vue) ergänzt.

```
├───build
├───config
├───node_modules
├───src
│   ├───assets
│   ├───components
│   └───router
└───static
```

Die Projektstruktur wird auf der Webseite des Templates [dokumentiert](https://vuejs-templates.github.io/webpack/structure.html).

Das Template befindet sich im Route-Verzeichnis und trägt den Namen *index.js*. Hier werden beispielsweise HTML-Header festgelegt. Die restlichen relevanten Dokumente befinden sich im */src*-Verzeichnis. Die zentrale Vue-Komponente heißt App.vue und befindet sich direkt im */src*-Verzeichnis. Sie verwendet den unter */src/router* definierten Router. Dieser verweist bisher nur auf eine einzige weitere Komponente, der Users-Komponente. Diese befindet sich im Verzeichnis */src/components*. Sie beschreibt eine Tabelle, mit allen Nutzern im System. Sie bietet die Möglichkeit User zu löschen, sowie Buttons, mit denen sich Modals öffnen lassen, über die dann Nutzerdaten bearbeitet werden können, sowie Freunde und Locations gelöscht werden können. Diese Modals befinden sich in eigenen Komponenten im selben Verzeichnis. Die Kommunikation zwischen diesen Komponenten und der Users-Komponente geschieht über Props und Events.

## API-Requests

Um mit dem Backend zu kommunizieren wird Axios verwendet. Entsprechende Controller befinden sich in der Datei */src/api.js*.

## JavaScript Style Guide

Im Frontend wird, wie im Backend auch, der Airbnb ...

## Ausblick

* Log-In
* Authentifizierung
* Nur geänderte Werte patchen
* Locations bearbeiten
* Bilder verwalten

# Mögliche Features

Freunde aus Kontaktliste vorschlagen