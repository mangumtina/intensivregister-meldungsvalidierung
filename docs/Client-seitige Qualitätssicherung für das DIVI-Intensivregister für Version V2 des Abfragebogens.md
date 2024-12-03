# Client-seitige Qualitätssicherung für das DIVI-Intensivregister für Version V2 des Abfragebogens
*Verfasser: RKI-Team des DIVI-Intensivregisters (intensivregister@rki.de)*

Dieses Dokument fasst alle Plausibilitäts-Checks & Systemanforderungen zusammen, welche der Software-Hersteller beim Erfassen von Meldungen für das DIVI-IntensivRegister implementiert haben soll.

**Hinweise**:
* **Durchgestrichene Passagen** betreffen derzeit **pausierte**, d. h. nicht erfragte Variablen. Nach Ende der Pausierung treten die betreffenden Prüfregeln wieder in Kraft.
* Die Nummerierung der Prüfregeln ist historisch bedingt. Durch nachträgliche Änderungen, Hinzufügen oder Streichen von Plausibilitätsprüfungen kann daher ein Sprung in der Nummerierung auftreten.

# 1.Tägliche Meldung
## 1.1) Pflichtfelder zum Speichern der täglichen Meldung
Eine Meldung kann von technischer Seite gespeichert werden, wenn sie mindestens Angaben zu den folgenden Datenfeldern enthält:
* betreibbare Betten *(intensivBetten)*
* belegte Betten *(intensivBettenBelegt)*
* Anzahl der Intensivpatient*innen mit Beatmung (nicht-invasiv UND invasiv) *(patientenBeatmet)*
* Anzahl der Intensivpatient*innen mit ECMO-Behandlung (Pflichtfeld mit NULL) *(patientenEcmo)*

Für die anderen Datenfelder sind auch NULL-Werte zugelassen. Das Speichern der Meldung ist trotzdem möglich. Wenn eines dieser Felder leer gelassen wurde (NULL), sollte es einen Warnhinweis geben. Im Intensivregister Frontend ist der Warnhinweis so implementiert, dass er auf der Meldungszusammenfassungsseite angezeigt wird, wenn ein Datenfeld mit Zahlenwert aus einer vorherigen Meldung auf eine NULLmeldung wechselt. Danach werden keine weiteren Warnhinweise bei späteren Meldungen angezeigt, bis wieder erstmalig ein Datenfeld leer gelassen wird.
<del>**Bitte beachten Sie: Grundsätzlich besteht eine Meldepflicht, die über die technischen Pflichtfelder hinausgeht.</del> Die gesetzliche Grundlage bildet die Verordnung zur Krankenhauskapazitätssurveillance (https://www.gesetze-im-internet.de/khkapsurv/BJNR626200022.html). Alle Datenfelder sind demnach rechtlich verpflichtend für die Meldung, ausgenommen Datenfelder, welche explizit mit dem Label „(optional)“ versehen sind.**

#### Fall: Datenfeld Betriebseinschränkung
NULL-Werte sind auch hier zugelassen und bedeuten, dass der Anwender nicht um den Zustand des Feldes weiß.
* Bei Auswahl der Betriebssituation mit ‚KEINE_ANGABE‘ oder ‚REGULAERER_BETRIEB‘ sollen die vier Felder für Betriebseinschränkungen (betriebseinschraenkungPersonal, betriebseinschraenkungRaum, betriebseinschraenkungBeatmungsgeraet, betriebseinschraenkungVerbrauchsmaterial) NULL sein. 
* Bei Auswahl der Betriebssituation mit ‚TEILWEISE EINGESCHRÄNKT‘ oder ‚EINGESCHRÄNKT‘ sollen die vier Felder ‚Gründe der Betriebseinschränkung‘ angezeigt werden. Es können beliebig viele dieser Betriebseinschränkungsgründe gesetzt werden (Mehrfachauswahl), kein Ausfüllen (NULL-Wert) der vier Felder ist auch möglich.

## 1.2) Notwendige Datentypen für Datenfelder
Die Datentypen der gemeldeten Werte müssen mit den Datentypen übereinstimmen, die in der Dokumentation der Schnittstelle (https://www.intensivregister.de/api/public/api-docs-ui) als Schema hinterlegt sind.

## 1.3) Notwendige Zahlenintervalle für numerische Eingabewerte
Die gemeldeten Werte aller numerischer Datenfelder müssen ganze Zahlen sein in dem Intervall [0,999] (0 und 999 eingeschlossen).

## 1.4) Notwendige Prüfregeln auf Eingabeseite für die tägliche Meldung
Die Prüfregeln sind **immer** aktiv, selbst wenn kein Wert in einem für die Regel relevanten Feld eingegeben wurde. Die Prüfregel **muss** eingehalten werden, um eine Meldung speichern zu können.

Wenn ein für die Regel relevantes Datenfeld **nicht** ausgefüllt wurde, werden für die Prüfregel leere Felder als die Zahl „0“ interpretiert. *Beispiel:* Für intensivBettenBelegt wurde kein Wert eingetragen, patientenBeatmet = 2. Nach Regel 5A wird nun getestet auf 0 >= 2. Da dies nicht erfüllt ist, wird ein Speichern verhindert.

Sollte gegen eine Regel verstoßen werden, soll der User auf diesen Regelbruch hingewiesen werden (z. B. in Form einer Fehlermeldung oder durch ein Pop-up während der Eingabe). 

* **Regel 1A:** Alle Eingaben <= 300

* **Regel 1B:** intensivBettenBelegt < intensivBetten + 60 % intensivBetten

* **Regel 1C:** patientenBeatmet <= intensivBettenBelegt

* **Regel 1D:** patientenEcmo <= intensivBettenBelegt

* **Regel 1E:** patientenEcmo NULL, wenn Meldebereich kein ECMO hat

* **Regel 1F:** patientenEcmo <= 300 oder NULL

## 1.5) Warnmeldung bei stark veränderten Werten
Bei zu starker Abweichung von der letzten Meldung sollte die folgende Warnmeldung für den User erscheinen: "Sind sie sicher, dass die Eingabe bei [Label] [Wert] korrekt ist, da die Zahl sehr stark abweicht?". Diese Warnmeldungen sollten nur bei numerischen Datenfeldern angezeigt werden. Wie groß die Abweichung sein muss, damit eine Warnmeldung angezeigt wird, hängt von der Größenordnung der vorherigen Meldung ab:
* bei Größenordnung 0-9 vorheriger Meldungen: Warnmeldung nur bei Sprung von +- >= 7
* bei Größenordnung 10-19 vorheriger Meldungen: Warnmeldung bei Verdoppelung oder Reduktion auf ein Viertel
* bei Größenordnung ab 20 vorheriger Meldungen: Warnmeldung bei Verdoppelung oder Halbierung

Eine Warnung muss für jedes Datenfeld, das eine starke Abweichung aufzeigt, einzeln angezeigt werden. 

Die Integration der Warnmeldung setzt voraus, dass das Client-System die letzte Meldung des Meldebereichs gespeichert hat und abrufen kann.

## 1.6) Funktionalität: Automatisches Werte-Vorausfüllen
Beim „Meldung erfassen“ kann eine automatische Werte-Vorausfüll-Funktionalität genutzt werden, die alle Datenfelder automatisch mit den Daten der letzten Erfassung befüllt.
Dies soll den Meldenden das Melden erleichtern, da nur Werte, die sich seit der letzten Erfassung verändert haben, angepasst werden müssen. Die Funktionalität setzt voraus, dass das Client-System die letzte Meldung des Meldebereichs gespeichert hat und abrufen kann.

Auch im Korrektur-Modus einer schon erfolgten Meldung ist ein Vorausfüllen unabhängig vom Zeitpunkt der vorhergehenden Meldung möglich.

## 1.7) Funktionalität: Nachträgliche Werte-Korrektur von erfassten Meldungen
Eine Meldung kann rückwirkend bis zu 14 Tage nach der initialen Erfassung der Meldung von allen Meldenden eines Meldebereichs korrigiert werden.
Im Frontend des Intensivregisters ist die Korrekturfunktionalität zu finden unter dem Menüpunkt "Mein Krankenhaus-Standort/ Mein Meldebereich:/ Meldungshistorie des Meldebereichs (korrigierbar)".

# 2) Strukturmeldung
## 2.1) Pflichtfelder zum Speichern der Strukturmeldung
Eine Strukturmeldung kann von technischer Seite gespeichert werden, wenn sie mindestens Angaben zu den folgenden Datenfeldern enthält:
* Anzahl der planmäßig verfügbaren Intensivbetten Ihres Meldebereichs *(planBetten)*
* Anzahl der planmäßig verfügbaren Beatmungskapazitäten (invasiv UND nicht-invasiv) Ihres Meldebereichs *(planBeatmungskapazitaeten:)*
* Anzahl der planmäßig verfügbaren ECMO-Kapazitäten Ihres Meldebereichs *(planEcmoKapazitaeten)*
* Anzahl im Rahmen eines intensivmedizinischen Notfall-Szenarios innerhalb von 7 Tagen an Ihrem Standort betreibbare Betten *(planNotfallkapazitaet)*

## 2.2) Notwendige Datentypen für Datenfelder
Die Datentypen der gemeldeten Werte müssen mit den Datentypen übereinstimmen, die in der Dokumentation der Schnittstelle (https://www.intensivregister.de/api/public/api-docs-ui) als Schema hinterlegt sind.

## 2.3) Notwendige Zahlenintervalle für numerische Eingabewerte
Die gemeldeten Werte aller numerischer Datenfelder müssen ganze Zahlen sein in dem Intervall [0,999] (0 und 999 eingeschlossen).

## 2.4) Notwendige Prüfregeln auf Eingabeseite für die tägliche Meldung
Die Prüfregeln sind **immer** aktiv, selbst wenn kein Wert in einem für die Regel relevanten Feld eingegeben wurde. Die Prüfregel **muss** eingehalten werden, um eine Meldung speichern zu können.

Wenn ein für die Regel relevantes Datenfeld **nicht** ausgefüllt wurde, werden für die Prüfregel leere Felder als die Zahl „0“ interpretiert. *Beispiel:* Für intensivBettenBelegt wurde kein Wert eingetragen, patientenBeatmet = 2. Nach Regel 5A wird nun getestet auf 0 >= 2. Da dies nicht erfüllt ist, wird ein Speichern verhindert.

* **Regel 2A:**
* **Regel 2B:**
* **Regel 2C:**

## 2.5) Warnmeldung bei stark veränderten Werten
Bei zu starker Abweichung von der letzten Meldung sollte die folgende Warnmeldung für den User erscheinen: "Sind sie sicher, dass die Eingabe bei [Label] [Wert] korrekt ist, da die Zahl sehr stark abweicht?". Diese Warnmeldungen sollten nur bei numerischen Datenfeldern angezeigt werden. Wie groß die Abweichung sein muss, damit eine Warnmeldung angezeigt wird, hängt von der Größenordnung der vorherigen Meldung ab:
* bei Größenordnung 0-9 vorheriger Meldungen: Warnmeldung nur bei Sprung von +- >= 7
* bei Größenordnung 10-19 vorheriger Meldungen: Warnmeldung bei Verdoppelung oder Reduktion auf ein Viertel
* bei Größenordnung ab 20 vorheriger Meldungen: Warnmeldung bei Verdoppelung oder Halbierung
