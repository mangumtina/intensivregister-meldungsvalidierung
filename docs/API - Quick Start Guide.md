# Schnittstellenbeschreibung des Intensivregisters - Quick Start Guide

## Dokument
Matti Mäkitalo <matti.maekitalo@prodyna.com>

# Getting Started
Sie bekommen Zugangsdaten, mit denen Sie zunächst ein "Access Token" bei unserer Authentifizierungssoftware [^Keycloak] erstellen.
Dieses Access-Token muss in jedem HTTP-Request im Header namens "Authorization" samt des Prefix "Bearer " (Leerzeichen beachten!) mitgesendet werden.

## Beispiel:
1. Access-Token abfragen
```
curl --location --request POST \
 'https://auth.intensivregister.de/[...]/connect/token' \
 --header 'Content-Type: application/x-www-form-urlencoded' \
 --data-urlencode 'grant_type=client_credentials' \
 --data-urlencode 'client_id=<CLIENT_ID>' \
 --data-urlencode 'client_secret=<CLIENT_SECRET>'
 ```

Die Antwort ist ein JSON-Dokument, welches u.a. den Eintrag "access_token" enthält. Dieses bitte extrahieren. 
Die Gültigkeitsdauer des Tokens ist ebenfalls in der Antwort enthalten (alternativ kann auch das Token entpackt werden 
- es ist ein normales JWT).

2. Beispiel-Request an das Intensivregister
```
curl --location --request GET \
  'https://www.intensivregister.de/api/stammdaten/meldebereich' \
  --header 'Accept: application/json' \
  --header 'Authorization: Bearer ACCESS_TOKEN'
```

Das Ergebnis ist ein JSON-Dokument mit der Menge der Meldebereiche, die Ihnen zugewiesen ist.

[^Keycloak] Wir nutzen Keycloak, ein OpenID-Connect fähige Lösung.

# Allgemeines

## Umgebungen und Freigaben

Das Intensivregister hat mehrere, komplett unabhängige Umgebungen.

In der Regel wird neuen API-Partnern zunächst Accounts und OICD-Client auf der UAT-Umgebung zugewiesen.

* `https://auth-uat.intensivregister.de/ir-uat/protocol/openid-connect/token` ist der Access-Token-Endpunkt
* `https://uat.intensivregister.de/api/` ist der Basispfad der Serveranwendung.

Nach Test und Freigabe durch das RKI wird ein Account für Produktion eingerichtet. Die Pfade dort sind

* `https://auth.intensivregister.de/intensivregister/protocol/openid-connect/token` ist der Access-Token-Endpunkt.
* `https://www.intensivregister.de/api/`  ist der Basispfad der Serveranwendung.

Zugewiesene Meldebereiche o.ä. auf UAT bedeuten nicht, dass ihren Nutzern der Meldebereich in Produktion ebenfalls
zugewiesen wird (oder umgekehrt) - die Umgebungen sind strikt voneinander getrennt. 
Auch IDs u.ä. können voneinander abweichen.

## Besonderheiten auf UAT

Auf UAT läuft täglich ein Datengenerierungsskript zum Erzeugen von Testdaten. Wenn gewünscht, können wir dies für ihren
Meldebereich ausschalten.

Auf UAT werden keine eMails versendet (z.B. keine "Passwort zurücksetzen"-eMails). Falls Sie einen eMail-Inhalt
benötigen, wenden Sie sich bitte den den IT-Support (s. Ende).

## Login-Modi
Wir stellen derzeit um von "password based credentials" auf "client credentials". 
Im Falle von "password based credentials" bekommen Sie folgende Informationen:

* client-id
* client-secret
* username
* passwort

Der Request zum Abholen des Access-Tokens sieht dann folgendermaßen aus:
```
curl --location --request POST \
 <URL des Token-Endpunktes, s.o.> \
 --header 'Content-Type: application/x-www-form-urlencoded' \
 --data-urlencode 'grant_type=password' \
 --data-urlencode 'client_id=<CLIENT_ID>' \
 --data-urlencode 'client_secret=<CLIENT_SECRET>'
 --data-urlencode 'username=<USERNAME>' \
 --data-urlencode 'password=<PASSWORD>' 
 ```

Im Fall von "client credentials" entfallen username/passwort, und der grant_type ist "client_credentials". Beispiel:
```
curl --location --request POST \
 'https://auth.intensivregister.de/[...]/connect/token' \
 --header 'Content-Type: application/x-www-form-urlencoded' \
 --data-urlencode 'grant_type=client_credentials' \
 --data-urlencode 'client_id=<CLIENT_ID>' \
 --data-urlencode 'client_secret=<CLIENT_SECRET>'
 ```


Als Ergebnis erhält man jeweils einen JSON-formatierten Body mit folgenden Feldern (gekürzt):

```json
{
  "access_token": "eyJhb...Yx2w",
  "expires_in": 900,
  "refresh_expires_in": 1800,
  "refresh_token": "eyJh...eJo",
  "token_type": "bearer",
  "not-before-policy": 0,
  "session_state": "a3c4...b3d4",
  "scope": "profile email"
}
 ```

Den Wert im Feld access_token muss in folgenden API-Calls an das Intensivregister mitgesendet werden.

## oAuth2
Wir gehen davon aus, dass sie ihre eigene Nutzerverwaltung haben und vertrauen darauf, dass Sie 
eine ordentliche Zuordnung der ihnen zugeordneten Meldebereiche zu ihren Nutzern regeln.

Alternativ können Sie auch die Intensivregister-Logins für ihre Nutzer verwenden - ihre Nutzer würden sich dann 
"bei ihnen" mit den Intensivregister-Logins anmelden und ihre Software hat ein Access-Tokens, mit denen sie dann
"im Namen" des Nutzers Requests stellen können. Wenn dieser Anwendungsfall für sie in Frage kommt wenden Sie sich
bitte an den IT-Support.

## Intensivregister-API

Die Dokumentation des (semi-)öffentlichen Teils dieser Schnittstelle findet man unter

* `https://www.intensivregister.de/api/public/api-docs-ui` (UI) bzw.
* `https://www.intensivregister.de/api/public/api-docs` (OpenAPI Spec).

Darüber hinaus gibt es noch weitere Endpunkte, die nicht in der öffentlichen API-Spec beschrieben werden.

### Kompabilität

Wir versuchen die API "backwards compatible" zu entwickeln bzw. garantieren eine Übergangsphase von einigen 
Monaten, in denen alte Requests technisch gültig sind.

Voraussetzung ist allerdings, dass sie beim Verarbeiten der Antworten "ignore unknowns" aktivieren, da wir
jederzeit beginnen könnten, in den Antworten neue Felder zu senden.

#### Beispiel:
Alte Antwort für eine Meldung
```
{
  "meldung_id": "abcd",
  "kapazitaeten": { ... }
}
```

wenn wir den Fragebogen erweitern, erweitern wir i.d.R. das Meldungsobjekt um ein neues Sub-Feld. 

Hier beispielhaft mit "neuaufnahmen"
```
{
  "meldung_id": "abcd",
  "kapazitaeten": { ... },
  "neuaufnahmen": {
    "erstaufnahmen": null,
    "verlegungen": null
  }
}
```

Selbst wenn sie die API ohne das ihnen unbekannte Sub-Feld aufrufen, wird die Antwort dieses Feld (mit dem Wert null)
enthalten. Wenn ihr API-Client kein "ignore unknowns" kann wird sie in diesem Fall einen Fehler werfen (da die Antwort
ein ihr unbekanntes Feld enthält).

Changes der API-Endpunkte, die nicht in der öffentlichen API-Spec beschrieben sind, werden nicht kommuniziert. Die Benutzung dieser Endpunkte ist möglich, wird aber nicht offiziell unterstützt.

## OpenAPI-Spezifikation

Es wird dringend angeraten, die Client-seitige API anhand der OpenAPI-Spec automatisch zu generieren[^4]. Das Intensivregister-Frontend nutzt diese Variante ebenfalls. Der Vorteil hierbei ist, dass Änderungen an der API einfacher nachzuvollziehen sind (da der clientseitige API-Code nur neu generiert werden muss). Dies ist für alle gängigen Programmiersprachen möglich.

[^4]: `https://github.com/OpenAPITools/openapi-generator`

# Fachliche Anwendungsfälle

## Meldebereiche des Nutzers abfragen

Die Meldebereiche, die für den Nutzer selber freigegeben sind, sind mit dem Endpunkt ```GET /stammdaten/meldebereich``` abfragbar.

## Letzte Meldung eines Meldebereichs abfragen.

Die letzte (aktive/freigegebene) Meldung eines Meldebereichs ist mit dem Endpunkt ```GET /stammdaten/meldebereich/{meldebereichId}/letzte-meldung``` abfragbar.

## Meldungen senden

Zum Abgeben von Meldungen ist der Endpunkt ```POST /meldungen``` relevant, zum aktualisieren
```PUT /meldungen/{meldungId}``` [^5]. Notwendige technische Informationen zum Benutzen sind: ein Access-Token (s.o.)
, die ID des Meldebereichs, für den man melden will (s.u.) und eine zufällig generierte UUID für die neue Meldung (bzw.
die ID der zu aktualisierenden Meldung).

[^5]: Ist die ID der Meldung bereits in Benutzung durch einen anderen Meldebereich wird die neue Meldung abgelehnt (Status-Code 403).

Die Beschreibung der zu sendenden Struktur findet sich in der OpenAPI-Spezifikation [^6]

[^6]: `https://www.intensivregister.de/api/public/swagger-ui/index.html?configUrl=
      /api/public/api-docs/swagger-config#/meldungs-controller/meldeKapazitaet`. 

Folgende Hinweise für einige Felder:

* id: eine selbst generierte UUID (für neue Meldungen) oder die ID der zu aktualisierenden Meldung
* api_version: muss "V2" sein (V1 wird nicht mehr akzeptiert)

NULL-Werte (bzw. nicht gesendet) sind (bis auf unten aufgeführte Felder)
zugelassen und bedeuten, dass der Anwender nicht um den Zustand des entsprechenden Feldes weiß. Die Ausprägungen von ```versorgungskategorie```, ```bettenVerfuegbarkeit``` und ```betriebssituation``` sind in der OpenAPI-Spec hinterlegt.

Die DIVI-Intensivregister-Verordnung[^Verordnung] definiert die Meldepflicht für verschiedene Datenfelder. Technisch verpflichtende Eingabefelder sind:

* ```id```
* ```meldebereich.id```
* ```kapazitaeten.intensivBetten```
* ```kapazitaeten.intensivBettenBelegt```

[^Verordnung]: `https://www.buzer.de/gesetz/13878/index.htm`

### Hinweis:
Ist die Betriebssituation ```KEINE_ANGABE``` oder ```REGULAERER_BETRIEB```, so sollen die vier Felder für
Betriebseinschränkungen (betriebseinschraenkungPersonal, betriebseinschraenkungRaum,
betriebseinschraenkungBeatmungsgeraet, betriebseinschraenkungVerbrauchsmaterial) NULL sein. In anderen Fällen können
beliebig viele dieser Betriebseinschränkungsgründe gesetzt werden.

Für eine genaue Datenfeld-Definition wenden Sie sich bitte an das RKI.

# Meldungsfreigabe

Um dem RKI eine fachliche Validierung der gesendeten Meldungen zu ermöglichen inkl. eines Parallelbetriebs wurde
folgendes Verfahren ermöglicht:

* mit der Kommunikation von Client-ID/-Secret sowie Nutzername/Passwort ist es für die entsprechende Umgebung möglich,
  Meldungen abzugeben.
* Ohne weitere Konfiguration werden die Meldungen zwar angenommen und gespeichert, aber werden nicht für inhaltliche
  Zwecke ausgewertet und in der Oberfläche nicht angezeigt. Die Meldungen sind "nicht freigegeben"
  und im Meldungsobjekt ist der Wert des Feldes "aktiv" auf 0.
* Nachdem das RKI die Qualität überprüft hat, wird ihr Client durch das IT-Team "freigegeben". Alle nachfolgenden
  Meldungen werden dadurch bei Meldungseingang freigegeben. Alte Meldungen bleiben im Zustand "nicht freigegeben".

Auf UAT wird ihr Client i.d.R. sofort freigegeben; obige Hinweise betreffen nur die Produktionsumgebung.

### Validierungen

Meldungen werden vor der Speicherung oder Aktualisierung zunächst validiert/plausibilisiert. Diese Validierungen können sich auf einzelne Felder beziehen (Beispiel: der Wert für ```faelleCovidAktuell``` muss im Bereich 0 <= faelleCovidAktuell <= 999 liegen), oder auf mehrere Felder (Beispiel: die Anzahl der aktuellen COVID-19-Fälle muss kleinergleich der Gesamtzahl der belegten Betten sein).

Schlägt die Validierung fehl, wird ein HTTP-Response mit dem Status Code 400 (Bad Request) zurückgegeben, welcher die
Validierungsfehler als ```errors``` enthält. Für jedes in der fehlgeschlagenen Validierung beteiligte Feld wird ein
separater ```error``` zurückgegeben.

Ein ```error``` besteht aus einem ```errorCode```, der den Fehler identifiziert, einem ```propertyPath```, welcher das beteiligte Feld benennt und einer ```errorMessage```, welche den Fehler beschreibt.

Ein Beispiel-Response für einen Request mit einer negativen Anzahl verstorbener und mehr aktuellen COVID-19-Fällen als Gesamtzahl der belegten Betten:

```json
{
  "statusCode": 400,
  "timestamp": "2020-05-15T16:24:19.186772",
  "errors": [
    {
      "errorCode": "VALIDATIONERROR",
      "propertyPath": "faelleCovidVerstorben",
      "errormessage": "must be greater than or equal to 0"
    },
    {
      "errorCode": "VALIDATIONERROR",
      "propertyPath": "intensivBettenBelegt",
      "errormessage": "faelleCovidAktuell must be less or equal to intensivBettenBelegt."
    },
    {
      "errorCode": "VALIDATIONERROR",
      "propertyPath": "faelleCovidAktuell",
      "errormessage": "faelleCovidAktuell must be less or equal to intensivBettenBelegt."
    }
  ]
}
 ``` 

Für die Meldung-Erfassen-Endpunkte gibt es ein separates Dokument mit den Plausibilisierungsregeln, zu finden unter 
`https://github.com/Intensivregister/intensivregister-meldungsvalidierung``

## Kontakt

Für Einrichtung von Zugängen wenden Sie sich bitte an den Helpdesk: `intensivregister-hilfe@rki.de`

Für technische Fragen zu diesem Dokument/der Schnittstelle wenden Sie sich bitte an das Support-Team:
`Tech-support-IntensivRegister@prodyna.com`. 

Das RKI ist erreichbar unter `intensivregister@rki.de`
