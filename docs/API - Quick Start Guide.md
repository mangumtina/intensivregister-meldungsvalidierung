# Schnittstellenbeschreibung des Intensivregisters - Quick Start Guide

## Dokument
Matti Mäkitalo <matti.maekitalo@prodyna.com>

## Zugriffsbeschränkungen

Alle Endpunkte mit "/public" im Pfad sind öffentlich nutzbar. Für alle anderen Endpunkte benötigt man ein AccessToken,
welches für einen spezifischen User mit bestimmten Rollen erstellt wurde. Diese Beschränkungen sind beim
DIVI-Intensivregisterteam erfragbar.

Dazu muss bei jedem API-Call ein HTTP-Header mit dem Namen "Authorization" und dem Wert "Bearer <ACCESS_TOKEN>" gesendet
werden. Beispiel:

```
curl --location --request GET \
  'https://www.intensivregister.de/api/intensivregister?page=0&size=10' \
  --header 'Content-Type: application/json' \
  --header 'Accept: application/json' \
  --header 'Authorization: Bearer <ACCESS_TOKEN>'
```

### Umgebungen und Freigaben

Das Intensivregister hat mehrere, komplett unabhängige Umgebungen.

In der Regel wird neuen API-Partnern zunächst Accounts und OICD-Client auf der UAT-Umgebung zugewiesen.

* `https://auth-uat.intensivregister.de/realms/ir-uat/protocol/openid-connect/token` ist der Access-Token-Endpunkt
* `https://uat.intensivregister.de/api/` ist der Basispfad der Serveranwendung.

Nach Test und Freigabe durch das RKI wird ein Account für Produktion eingerichtet. Die Pfade dort sind

* `https://auth.intensivregister.de/realms/intensivregister/protocol/openid-connect/token` ist der Access-Token-Endpunkt.
* `https://www.intensivregister.de/api/`  ist der Basispfad der Serveranwendung.

Zugewiesene Meldebereiche o.ä. auf UAT bedeuten nicht, dass ihren Nutzern der Meldebereich in Produktion ebenfalls
zugewiesen wird (oder umgekehrt) - die Umgebungen sind strikt voneinander getrennt.

### Besonderheiten auf UAT

Auf UAT läuft täglich ein Datengenerierungsskript zum Erzeugen von Testdaten. Wenn gewünscht, können wir dies für ihren
Meldebereich ausschalten.

Auf UAT werden keine eMails versendet (z.B. keine "Passwort zurücksetzen"-eMails). Falls Sie einen eMail-Inhalt
benötigen, wenden Sie sich bitte den den IT-Support (s. Ende).

## Login-Modi

### oAuth2-Standard-Flow vs. technischer Nutzer

Prinzipiell ist das System so ausgelegt, dass der Nutzer durch die Client-Applikation auf den Auth-Provider[^3]
weitergeleitet wird, sich dort einloggt und dann samt
des Access-Tokens zur aufrufenden Applikation zurückgeleitet wird. Die aufrufende Applikation kann dann mit dem Access-
Token im Namen des Nutzers Aktionen ausführen (Standard-oAuth2-Login-Flow). Dazu muss insbesondere die Redirect-URL der
Client-Applikation hinterlegt werden.

[^3]: `https://auth.intensivregister.de/realms/intensivregister`

Es ist möglich, dass ein technischer Nutzer eingerichtet wird, welcher als zusätzlicher Nutzer Schreibrecht auf einen
oder mehrere Meldebereiche erhält. Dann kann die Client-Applikation sich mit dessen Logindaten einloggen und hat dann
Zugriff auf die entsprechend konfigurierten Meldebereiche.

Wenn möglich wird der oAuth2-Standard-Loginflow präferiert - der Weg mit dem technischen Nutzer sollte nur dann gewählt
werden, wenn technische Gründe den Standard-Flow unmöglich machen (etwa: Meldungen werden im Batch-Verfahren ohne
Nutzer-Interaktion gesendet).

## Direktes Abholen des Access-Tokens (für den Fall "technischer Nutzer")

Das API-Token kann man sich mit einem OpenID-Connect-Client beim DIVI-Intensivregister keycloak abholen[^1].
Alternativ kann man auch folgenden HTTP-Call ausführen[^2]:


[^1]: `https://auth.intensivregister.de/realms/intensivregister/.well-known/openid-configuration`
[^2]: `https://tools.ietf.org/html/rfc6749#section-4.3`

```
curl --location --request POST \
 'https://auth.intensivregister.de/[...]/connect/token' \
 --header 'Content-Type: application/x-www-form-urlencoded' \
 --data-urlencode 'grant_type=password' \
 --data-urlencode 'client_id=<CLIENT_ID>' \
 --data-urlencode 'client_secret=<CLIENT_SECRET>'
 --data-urlencode 'username=<USERNAME>' \
 --data-urlencode 'password=<PASSWORD>' 
 ```

* CLIENT_ID und CLIENT_SECRET kann vom DIVI-Intensivregisterteam erfragt werden. Diese Werte sind pro Client-Applikation
  und geheimzuhalten.
* USERNAME und PASSWORD sind die Nutzercredentials des einzuloggenden, technischen Users.

Als Ergebnis erhält man einen JSON-formatierten Body mit folgenden Feldern (gekürzt):

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

## Intensivregister-API

Die Dokumentation des (semi-)öffentlichen Teils dieser Schnittstelle findet man unter

* `https://www.intensivregister.de/api/public/api-docs-ui` (UI) bzw.
* `https://www.intensivregister.de/api/public/api-docs` (OpenAPI Spec).

Darüber hinaus gibt es noch weitere Endpunkte, die nicht in der öffentlichen API-Spec beschrieben werden.

### Kompatibilität

Es gibt keine garantierte Rückwärtskompatibilität der API. Im Fall von notwendigen rückwärts inkompatiblen Changes (insb: ein alter Request ist ungültig im Kontext der neuen API) wird es aber i.d.R. eine Übergangsfrist geben. API-Changes werden per e-Mail bekanntgegeben.

Changes der API-Endpunkte, die nicht in der öffentlichen API-Spec beschrieben sind, werden nicht kommuniziert. Die Benutzung dieser Endpunkte ist möglich, wird aber nicht offiziell unterstützt.

### OpenAPI-Spezifikation

Es wird dringend angeraten, die Client-seitige API anhand der OpenAPI-Spec automatisch zu generieren[^4]. Das Intensivregister-Frontend nutzt diese Variante ebenfalls. Der Vorteil hierbei ist, dass Änderungen an der API einfacher nachzuvollziehen sind (da der clientseitige API-Code nur neu generiert werden muss). Dies ist für alle gängigen Programmiersprachen möglich.

[^4]: `https://github.com/OpenAPITools/openapi-generator`

### Fachliche Anwendungsfälle

#### Meldebereiche des Nutzers abfragen

Die Meldebereiche, die für den Nutzer selber freigegeben sind, sind mit dem Endpunkt ```GET /stammdaten/meldebereich``` abfragbar.

#### Letzte Meldung eines Meldebereichs abfragen.

Die letzte (aktive/freigegebene) Meldung eines Meldebereichs ist mit dem Endpunkt ```GET /stammdaten/meldebereich/{meldebereichId}/letzte-meldung``` abfragbar.

#### Meldungen senden

Zum Abgeben von Meldungen ist der Endpunkt ```POST /api/meldungen``` relevant, zum Aktualisieren
```PUT /api/meldungen/{meldungId}``` [^5]. Notwendige technische Informationen zum Benutzen sind: ein Access-Token (s.o.), die ID des Meldebereichs, für den man melden will (s.u.) und eine zufällig generierte UUID für die neue Meldung (bzw. die ID der zu aktualisierenden Meldung).

[^5]: Ist die ID der Meldung bereits in Benutzung durch einen anderen Meldebereich wird die neue Meldung abgelehnt (Status-Code 403).

Die Beschreibung der zu sendenden Struktur findet sich in der OpenAPI-Spezifikation [^6]

[^6]: `https://www.intensivregister.de/api/public/swagger-ui/index.html?configUrl=
      /api/public/api-docs/swagger-config#/meldungs-controller/meldeKapazitaet`. 

Folgende Hinweise für einige Felder:

* id: eine selbst generierte UUID (für neue Meldungen) oder die ID der zu aktualisierenden Meldung
* bettenmeldungen: muss NULL sein oder weggelassen werden
* api_version: muss "V2" sein (V1 wird nicht mehr akzeptiert)
* quellsystem: dieser Wert kann weggelassen werden. Dieser wird serverseitig mit dem Namen des API-Clients
  überschrieben.
* lastUpdateTimestamp, lastUpdateIdmUserId, creationIdmUserId, creationTimestamp: werden alle serverseitig gesetzt und
  können weggelassen werden
* aktiv: kann weggelassen werden.

Das Attribut ```kapazitaeten``` beschreibt den aktuellen Zustand der Betten und ist zwingend zu füllen.

NULL-Werte (bzw. nicht gesendet) sind (bis auf unten aufgeführte Felder)
zugelassen und bedeuten, dass der Anwender nicht um den Zustand des entsprechenden Feldes weiß. Die Ausprägungen von ```versorgungskategorie```, ```bettenVerfuegbarkeit``` und ```betriebssituation``` sind in der OpenAPI-Spec hinterlegt.

Die DIVI-Intensivregister-Verordnung[^Verordnung] definiert die Meldepflicht für verschiedene Datenfelder. Technisch verpflichtende Eingabefelder sind:

* ```id```
* ```meldebereich.id``` (das restliche Meldebereich-Objekt kann leer gelassen werden)
* ```kapazitaeten.intensivBetten```
* ```kapazitaeten.intensivBettenBelegt```
* ```faelleCovidGenesen```
* ```faelleCovidVerstorben```

[^Verordnung]: `https://www.buzer.de/gesetz/13878/index.htm`

Hinweis:
Ist die Betriebssituation ```KEINE_ANGABE``` oder ```REGULAERER_BETRIEB```, so sollen die vier Felder für
Betriebseinschränkungen (betriebseinschraenkungPersonal, betriebseinschraenkungRaum,
betriebseinschraenkungBeatmungsgeraet, betriebseinschraenkungVerbrauchsmaterial) NULL sein. In anderen Fällen können
beliebig viele dieser Betriebseinschränkungsgründe gesetzt werden.

Für eine genaue Datenfeld-Definition wenden Sie sich bitte an das RKI.

#### Aktuelle Ankündigung zu API Änderungen erweiterte COVID-19-Fälle vom 22.11.2022

Auf vielfachen Wunsch und daraus resultierender Vorgabe aus dem Bundesministerium für Gesundheit implementieren wir aktuell eine erweiterte Abfrage zu den COVID-19-Fällen, die bald differenziert nach mit bzw. ohne intensivmedizinisch relevante COVID-19-Manifestation abgefragt werden.
Im Anhang finden Sie einen Mockup zur neuen geplanten Abfrage sowie die anzupassenden Plausibilitätsregeln - beides ein fortgeschrittener Arbeitsstand.
Diese Umstellung wird sowohl für Erwachsenen- als auch Kinder-Meldebereiche erfolgen.
Zudem wird die Abfrage für Schwangere reduziert auf die Frage "Anzahl aktuell schwangerer und frisch entbundener COVID-19-Patient*innen auf Ihrer ITS" bzw. "anzahlSchwangere".

Geplant ist ein Live-Gang zum 04. Januar 2023, eventuell aber auch früher im Dezember. Eine (vorübergehende) Abwärtskompatibilität bzw. einen übergangsweisen Parallelbetrieb werden wir sicher stellen.

Den aktuellen Stand der in Umsetzung befindlichen API finden Sie auf unserer Testumgebung (Anpassung des Meldungsobjektes / POST / meldungen):
https://uat.intensivregister.de/api/public/swagger-ui/index.html?configUrl=/api/public/api-docs/swagger-config#/meldungs-controller/meldeKapazitaet
Konkret kommt das Objekt "covid19StatusV3" hinzu.

In den nächsten Wochen werden wir an der API zudem noch weitere Methoden, vor allem zur Ausleitung der neu erfassten Informationen, anpassen:
POST public/intensivregister, GET /public/reporting/laendertabelle, GET  /public/data-api/krankenhaus-standorte-meldungen.csv. Dies sollte Sie allerdings nicht betreffen.

Nach Umstellung auf die neue Übermittlung durch alle API-Partner:innen werden dann obsolete Felder entfernt (bspw. "anzahlSchwangereBeatmet" oder "faelleCovidOhneSymptomatik")

#### Meldungsfreigabe

Um dem RKI eine fachliche Validierung der gesendeten Meldungen zu ermöglichen inkl. eines Parallelbetriebs wurde
folgendes Verfahren ermöglicht:

* mit der Kommunikation von Client-ID/-Secret sowie Nutzername/Passwort ist es für die entsprechende Umgebung möglich,
  Meldungen abzugeben.
* Ohne weitere Konfiguration werden die Meldungen zwar angenommen und gespeichert, aber werden nicht für inhaltliche
  Zwecke ausgewertet und in der Oberfläche nicht angezeigt. Die Meldungen sind "nicht freigegeben"
  und im Meldungsobjekt ist der Wert des Feldes "aktiv" auf 0.
* Nachdem das RKI die Qualität überprüft hat, wird ihr Client durch das IT-Team "freigegeben". Alle nachfolgenden
  Meldungen werden dadurch bei Meldungseingang freigegeben. Alte Meldungen bleiben im Zustand "nicht freigegeben".

#### Hinweise:

Die Meldebereich-IDs sind technische Schlüssel (UUIDs) und haben keine fachliche Bewandnis.

Der Nutzeraccount des Access-Tokens muss für das Schreiben auf den Meldebereich freigeschaltet werden, entweder, weil der Nutzeraccount Owner des Meldebereichs ist oder weil das Entwicklerteam eine separate Schreibberechtigung auf den Meldebereich konfiguriert hat. Es ist insbesondere nicht möglich, für beliebige Meldebereiche Meldungen abzugeben.

### Validierungen

Meldungen werden vor der Speicherung oder Aktualisierung zunächst validiert/plausibilisiert. Diese Validierungen können sich auf einzelne Felder beziehen (Beispiel: der Wert für ```faelleCovidAktuell``` muss im Bereich 0 <= faelleCovidAktuell <= 999 liegen), oder auf mehrere Felder (Beispiel: die Anzahl der aktuellen COVID-19-Fälle muss kleinergleich der Gesamtzahl der belegten Betten sein).

Es gibt auch Plausibilitätsregeln über die aktuelle Meldung hinweg. So wird z.B. geprüft, dass ```faelleCovidGenesen``` sowie ```faelleCovidVerstorben``` von der letzten Meldung zur neuen Meldung nicht abnimmt. Beim Bearbeiten von alten Meldungen (nur über die manuelle Maske auf intensivregister.de möglich) sind diese Plausibilitätsregeln über die aktuelle Meldung hinweg ausgeschaltet.

Schlägt die Validierung fehl, wird ein HTTP-Response mit dem Status Code 400 (Bad Request) zurückgegeben, welcher die Validierungsfehler als ```errors``` enthält. Für jedes in der fehlgeschlagenen Validierung beteiligte Feld wird ein separater ```error``` zurückgegeben.

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

Für die Meldung-Erfassen-Endpunkte (neu und update) gibt es ein separates Dokument mit den Plausibilisierungsregeln, abzufragen beim RKI.

## Kontakt

Für Einrichtung von Zugängen wenden Sie sich bitte an den Helpdesk: `intensivregister-hilfe@rki.de`

Für technische Fragen zu diesem Dokument/der Schnittstelle wenden Sie sich bitte an das Support-Team:
`Tech-support-IntensivRegister@prodyna.com`. 

Das RKI ist erreichbar unter `intensivregister@rki.de`
