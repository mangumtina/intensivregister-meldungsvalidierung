Hinweise zum Dokument:

    Gelbe Markierungen: Änderungen zur vorhergehenden Version

    Blaue Markierungen: Diese Vorgaben sind seitens des DIVI-Intensivregisters noch nicht implementiert, werden aber zeitnah nachgezogen.

Inhalt

Dieses Dokument fasst alle Plausibilitäts-Checks & Systemanforderungen zusammen, welche der Software-Hersteller implementiert haben soll beim Erfassen von Meldungen für das DIVI-IntensivRegister.
1) Zusammenfassung nach der Meldung aller Daten

Nach der Eingabe aller Meldungen, soll der meldende User in einem letzten Schritt aufgefordert werden, alle zusammengefassten Daten nochmals zu prüfen. Dies dient der Sicherstellung einer hohen Datenqualität.  An diesem Schritt sollen auch Warnmeldungen erscheinen, um den User auf potenzielle Fehler hinzuweisen (siehe Punkt 6) und 7)).
2) Pflichtfelder zum Speichern der Meldung

Eine Meldung kann von technischer Seite gespeichert werden, wenn sie mindestens Angaben zu den folgenden Datenfeldern enthält:

    betreibbare Betten (intensiv_betten)

    belegte Betten (intensiv_betten_belegt)

    covid_genesen

    covid_verstorben

Für die anderen Datenfelder sind auch NULL-Werte zugelassen. Wenn eines dieser Felder leer gelassen wurde (NULL), sollte es einen Warnhinweis auf der Zusammenfassungsseite geben. In diesem Fall ist das Speichern der Meldung trotzdem möglich. 
Es wird jedoch ausdrücklich auf die bestehende Meldepflicht entsprechend der jeweils geltenden DIVI-Intensivregister-Verordnung hingewiesen. Alle Datenfelder sind demnach rechtlich verpflichtend für die Meldung, ausgenommen Datenfelder, die explizit mit dem Label „(optional)“ versehen sind.

Fall:  Datenfeld Betriebseinschränkung 
NULL-Werte sind auch hier zugelassen und bedeuten, dass der Anwender nicht um den Zustand des Feldes weiß.

    Bei Auswahl der Betriebssituation mit ‚KEINE_ANGABE‘ oder ‚REGULAERER_BETRIEB‘ sollen die vier Felder für Betriebseinschränkungen (betriebseinschraenkungPersonal, betriebseinschraenkungRaum, betriebseinschraenkungBeatmungsgeraet, betriebseinschraenkungVerbrauchsmaterial) NULL sein.

    Bei Auswahl der Betriebssituation mit ‚TEILWEISE EINGESCHRÄNKT‘ oder ‚EINGESCHRÄNKT‘ sollen die vier Felder ‚Gründe der Betriebseinschränkung‘ angezeigt werden. Es können beliebig viele dieser Betriebseinschränkungsgründe gesetzt werden (Mehrfachauswahl), kein Ausfüllen (NULL-Wert) der vier Felder ist auch möglich.

3) Notwendige Datentypen für Datenfelder

Die Datentypen der gemeldeten Werte müssen mit den Datentypen übereinstimmen, die in Spalte B des Datenmodells spezifiziert sind (siehe Datenmodell Dokument).
4) Notwendige Zahlenintervalle für numerische Eingabewerte

    Die gemeldeten Werte der folgenden Datenfelder müssen ganze Zahlen sein, die größer gleich 0 sind, aber keine obere Grenze haben:

        faelle_covid_genesen

        faelle_covid_verstorben

Für diese beiden Felder siehe auch Regeln 5H und 5I.

    Die gemeldeten Werte aller anderer numerischer Datenfelder müssen ganze Zahlen sein in dem Intervall [0,999] (0 und 999 eingeschlossen).

5) Notwendige Prüfregeln auf Eingabeseite

Die Prüfregeln sind immer aktiv, selbst wenn kein Wert in einem für die Regel relevanten Feld eingegeben wurde. Die Prüfregel muss eingehalten werden, um eine Meldung speichern zu können.

Wenn ein für die Regel relevantes Datenfeld nicht ausgefüllt wurde, werden für die Prüfregel leere Felder als die Zahl „0“ interpretiert. Beispiel: Für intensiv_betten wurde kein Wert eingetragen, intensiv_betten_belegt = 2. Nach Regel 5A wird nun getestet auf 0 >= 2. Da dies nicht erfüllt ist, wird ein Speichern verhindert.

Sollte gegen eine Regel verstoßen werden, soll der User auf diesen Regelbruch hingewiesen werden (z.B. in Form einer Fehlermeldung oder durch ein Pop-up während der Eingabe).

    Regel 5A:
    intensiv_betten >= intensiv_betten_belegt 


    Regel 5B (Beziehung freie Behandlungskapazitäten <-> Bettenzahl):

5B1: intensiv_betten_belegt >= faelle_covid_aktuell

5B2: intensiv_betten_belegt >= faelle_rsv_aktuell

5B3: intensiv_betten_belegt >= faelle_influenza_aktuell

Hinweis: für Erwachsenen-Meldebereiche sind RSV/Influenza-Felder alle NULL, im Sinne der Validierung 0.

Regeln 5B1 bis 5B3 sind überflüssig durch Regel 5D4. Diese Regeln nicht mehr implementieren bzw. wieder entfernen.

5B4: intensiv_betten >= freie_iv_kapazitaet

5B5: intensiv_betten >= freie_ecmo_kapazitaet

Zum Jahreswechsel 2022/2023 wurde eine Änderung der Regeln 5B4 und 5B5 angekündigt. Diese Änderung ist jedoch NICHT erfolgt und soll bis auf Weiteres NICHT implementiert werden. 5B4 und 5B5 bleiben vorerst wie hier beschrieben.


    Regel 5C (Beziehung COVID-19- (bzw. RSV-, Influenza-)Patient*innen_Gesamt <-> Behandlungsgruppen COVID-19-/RSV-/Influenza-Patient*innen):

 

Regeln für COVID-19 (alle Meldebereiche)

5C1 faelle_covid_aktuell >= faelle_covid_aktuell_beatmet

5C2 faelle_covid_aktuell >= faelle_covid_aktuell_high_flow_oxygen

5C3 faelle_covid_aktuell >= faelle_covid_aktuell_nicht_invasiv_beatmet

Regeln 5C1 bis 5C3 sind überflüssig durch Regel 5C5. Diese Regeln nicht mehr implementieren bzw. wieder entfernen.

5C4 faelle_covid_aktuell >= faelle_covid_aktuell_ecmo
5C4A faelle_covid_aktuell_mit_manifestation >=
                  faelle_covid_aktuell_mit_manifestation_ecmo

5C4B faelle_covid_aktuell_ohne_manifestation >=
                  faelle_covid_aktuell_ohne_manifestation_ecmo

5C5 faelle_covid_aktuell >=

faelle_covid_aktuell_beatmet

+ faelle_covid_aktuell_high_flow_oxygen

+ faelle_covid_aktuell_nicht_invasiv_beatmet

5C5A faelle_covid_aktuell_mit_manifestation >= 

faelle_covid_aktuell_mit_manifestation_beatmet

+ faelle_covid_aktuell_mit_manifestation_high_flow_oxygen

+ faelle_covid_aktuell_mit_manifestation_nicht_invasiv_beatmet

5C5B faelle_covid_aktuell_ohne_manifestation >= 

faelle_covid_aktuell_ohne_manifestation_beatmet

+ faelle_covid_aktuell_ohne_manifestation_high_flow_oxygen

+ faelle_covid_aktuell_ohne_manifestation_nicht_invasiv_beatmet

5C6 faelle_covid_aktuell ==

faelle_covid_aktuell_mit_manifestation

+  faelle_covid_aktuell_ohne_manifestation


Regeln für RSV (nur für Kinder-Meldebereiche)

5C11 faelle_rsv_aktuell >= faelle_rsv_aktuell_beatmet

5C21 faelle_rsv_aktuell >= faelle_rsv_aktuell_high_flow_oxygen

5C31 faelle_rsv_aktuell >= faelle_rsv_aktuell_nicht_invasiv_beatmet

Regeln 5C11 bis 5C31 sind überflüssig durch Regel 5C51. Diese Regeln nicht mehr implementieren bzw. wieder entfernen.

 

5C41 faelle_rsv_aktuell  >= faelle_rsv_aktuell_ecmo

5C51 faelle_rsv_aktuell  >= 

faelle_rsv_aktuell_beatmet

+ faelle_rsv_aktuell_high_flow_oxygen

+ faelle_rsv_aktuell_nicht_invasiv_beatmet

 

Regeln für Influenza (nur für Kinder-Meldebereiche)

5C12 faelle_influenza_aktuell >= faelle_influenza_aktuell_beatmet

5C22 faelle_influenza_aktuell >= faelle_influenza_aktuell_high_flow_oxygen

5C32 faelle_influenza_aktuell >= faelle_influenza_aktuell_nicht_invasiv_beatmet

Regeln 5C12 bis 5C32 sind überflüssig durch Regel 5C52. Diese Regeln nicht mehr implementieren bzw. wieder entfernen.

 

5C42 faelle_influenza_aktuell  >= faelle_influenza_aktuell_ecmo

5C52 faelle_influenza_aktuell  >= 

faelle_influenza_aktuell_beatmet

+ faelle_influenza_aktuell_high_flow_oxygen

+ faelle_influenza_aktuell_nicht_invasiv_beatmet

 

    Regel 5D (Beziehung Patient*innen_Alle <-> COVID-19-/RSV-/Influenza-Patient*innen):

Hinweis: für Erwachsenen-Meldebereiche sind RSV/Influenza-Felder alle NULL, im Sinne der Validierung 0.

5D1 patienten_invasiv_beatmet >=

faelle_covid_aktuell_mit_manifestation_beatmet

+ faelle_covid_aktuell_ohne_manifestation_beatmet

+ faelle_rsv_aktuell_beatmet

+ faelle_influenza_aktuell_beatmet. 

5D2 patienten_nicht_invasiv_beatmet >= 

faelle_covid_aktuell_mit_manifestation_nicht_invasiv_beatmet

+ faelle_covid_aktuell_ohne_manifestation_nicht_invasiv_beatmet

+ faelle_rsv_aktuell_nicht_invasiv_beatmet

+ faelle_influenza_aktuell_nicht_invasiv_beatmet

5D3 patienten_ecmo >=

faelle_covid_aktuell_mit_manifestation_ecmo

+ faelle_covid_aktuell_ohne_manifestation_ecmo

+ faelle_rsv_aktuell_ecmo

+ faelle_influenza_aktuell_ecmo

5D4 intensiv_betten_belegt >=

faelle_covid_aktuell

+ faelle_rsv_aktuell

+ faelle_influenza_aktuell

+ faelle_pims_aktuell


    Regel 5E (Beziehung Belegte Betten_Patient*innen_Alle <-> Behandlungsgruppen Patient*innen_Alle):

5E1 intensiv_betten_belegt >= patienten_ecmo

5E2 intensiv_betten_belegt >= patienten_invasiv_beatmet

5E3 intensiv_betten_belegt >= patienten_nicht_invasiv_beatmet
Regeln 5E2 und 5E3 sind überflüssig durch Regel 5E4. Diese Regeln nicht mehr implementieren bzw. wieder entfernen.

5E4 intensiv_betten_belegt >= 

patienten_invasiv_beatmet

+ patienten_nicht_invasiv_beatmet 


    Regel 5F (Beziehung Behandlungskapazitäten_Alle  <->  Behandlungskapazitäten_COVID-19)

5F0 intensiv_betten >= covid_kapazitaet_frei

5F1 freie_iv_kapazitaet >= covid_kapazitaet_frei_iv

5F2 freie_ecmo_kapazitaet >= covid_kapazitaet_frei_ecmo

5F3 Folgende Felder werden nur für Erwachsenen-Meldebereiche bzw. Meldebereiche ohne definierten Behandlungsschwerpunkt erfasst:

- covid_kapazitaet_frei,

- covid_kapazitaet_frei_iv,

- covid_kapazitaet_frei_ecmo

5F4 Folgende Felder werden nur für Kinder-Meldebereiche erfasst:

    kapazitaet_frei_isolationspflichtig

5F5 kapazitaet_frei_isolationspflichtig <= intensiv_betten


    Regel 5G (Beziehung COVID-19-Patient*innen <-> COVID-19-Alterstrata)
    5G1 Folgende Altersstrata werden nicht in Kinder-Meldebereichen erfasst, sondern nur für Erwachsenen-Meldebereiche bzw. Meldebereiche ohne definierten Behandlungsschwerpunkt:

        stratum17minus

        stratum18bis29

        stratum30bis39

        stratum40bis49

        stratum50bis59

        stratum60bis69

        stratum70bis79

        stratum80plus

5G2 sum (Alterstrata Covid Fälle) <= faelle_covid_aktuell   
[Diese Regel prüft „<=“, damit keine Meldungen verloren werden, wenn Meldebereiche einige Altersgruppen nicht angeben können oder möchten.]
Siehe hierzu auch Regel 6B: nicht speicherverhindernde Warnung bei sum (Alterstrata Covid Fälle) < faelle_covid_aktuell.

5G3: existiert nicht (Zählfehler)

5G4: faelle_covid_aelter_als_17j darf nur für Kinder-Meldebereiche erfasst werden

5G5: faelle_covid_aelter_als_17j <= faelle_covid_aktuell


    Regel 5H (Genesene COVID-19-Fälle)

5H1 faelle_covid_genesen nur als kumulativ anwachsende Zahl erlaubt; 
faelle_covid_genesen (aktuelle Meldung) >= faelle_covid_genesen (letzte Meldung)

5H2 faelle_covid_genesen darf nicht um mehr als 50 pro Meldung ansteigen; 
faelle_covid_genesen (aktuelle Meldung) <= 50 + faelle_covid_genesen (letzte Meldung)

Für die Regeln 5H siehe auch Abschnitt 10: Keine Regeln 5H bei Korrektur von Meldungen

 

    Regel 5I (Verstorbene COVID-19-Fälle)

5I1 faelle_covid_verstorben nur als kumulativ anwachsende Zahl erlaubt; 
faelle_covid_verstorben (aktuelle Meldung) >= faelle_covid_verstorben (letzte Meldung)

5I2 faelle_covid_verstorben darf nicht um mehr als 50 pro Meldung ansteigen; 
faelle_covid_verstorben (aktuelle Meldung) <= 50 + faelle_covid_verstorben (letzte Meldung)

Für die Regeln 5I siehe auch Abschnitt 10: Keine Regeln 5I bei Korrektur von Meldungen


    Regel 5J (Beziehung COVID-19-Patient*innen <-> SARS-CoV-2-Virusvarianten)

5J1 sum (SARS-CoV-2-Virusvarianten-Gruppe Covid Fälle) <= faelle_covid_aktuell   
[Diese Regel prüft „<=“, damit keine Meldungen verloren werden, wenn Meldebereiche keine Angaben zu VOCs machen.]


    Regel 5K (Neuaufnahmen COVID-19-Patient*innen)

5K1 Wenn die betreffende Meldung die erste Meldung des Kalendertages (0:00:00 - 23:59:59 Uhr) ist, erfolgt keine automatische Vorausfüll-Funktionalität bei dem Datenfeld Neuaufnahmen. Falls am gleichen Kalendertag schon eine Meldung für den Meldebereich abgegeben wurde, wird der Wert der Neuaufnahmen der letzten vorherigen Meldung des Tages vorausgefüllt. (siehe auch Abschnitt 9, Ausnahmen)

5K2 ((faelle_covid_aktuell  +  covid_kapazitaet_frei) * 2 ) +2   > 

(faelle_covid_vortag_erstaufnahmen  +  faelle_covid_vortag_verlegungen)


    Regel 5L (Schwangere COVID-19-Patient*innen)

5L1 Die Abfrage zu Schwangeren soll nur für Meldebereiche angezeigt werden, die als Behandlungsschwerpunkt „Erwachsene“ angegeben haben oder den Behandlungsschwerpunkt nicht angegeben haben („NA“).

5L2 schwangere_werden_aktuell_behandelt == “NEIN“ oder

anzahl_schwangere <= faelle_covid_aktuell

5L3 schwangere_werden_aktuell_behandelt == “NEIN“ oder

anzahl_schwangere_beatmet <=

faelle_covid_aktuell_nicht_invasiv_beatmet

+ faelle_covid_aktuell_beatmet

5L4 schwangere_werden_aktuell_behandelt == “NEIN“ oder

anzahl_schwangere_ohne_beatmung <=

faelle_covid_aktuell

– faelle_covid_aktuell_nicht_invasiv_beatmet

– faelle_covid_aktuell_beatment

5L5 schwangere_werden_aktuell_behandelt == “NEIN“ oder

anzahl_schwangere >=

anzahl_schwangere_beatmet

+ anzahl_schwangere_ohne_beatmung

 

    Regel 5M (Impf-Status der COVID-19-Patient*innen)

5M3A faelle_covid_vortag_erstaufnahmen ==

impfstatus_unbekannt

+ impfstatus_0_impfungen

+ impfstatus_1_impfung

+ impfstatus_2_impfungen

+ impfstatus_3_impfungen

+ impfstatus_4+_impfungen 

5M7 Wenn die betreffende Meldung die erste Meldung des Kalendertages (0:00:00 - 23:59:59 Uhr) ist, erfolgt keine automatische Vorausfüll-Funktionalität bei den Datenfeldern für Impfstatus. Falls am gleichen Kalendertag schon eine Meldung für den Meldebereich abgegeben wurde, werden die gemeldeten Werte für Impfstatus der letzten vorherigen Meldung des Tages vorausgefüllt. (siehe auch Abschnitt 9, Ausnahmen)

 

    Regel 5N (COV- positive Patient*innen ohne Symptomatik)

5N1: COV- positive Patient*innen ohne Symptomatik darf nur für Kinder-Meldebereiche erfasst werden

5N2: faelle_covid_ohne_symptomatik <= faelle_covid_aktuell

5N3: (Erweiterung der Regel 5C5 für Kinder-Meldebereiche)

faelle_covid_aktuell  >= 

faelle_covid_aktuell_beatmet

+ faelle_covid_aktuell_high_flow_oxygen

+ faelle_covid_aktuell_nicht_invasiv_beatmet

+ faelle_covid_ohne_symptomatik


    Regel 5O (PIMS / MIS-C)

5O1: PIMS / MIS-C Patient*innen dürfen nur für Kinder-Meldebereiche erfasst werden

5O2: Wenn faelle_pims_vortag_erstaufnahmen die erste Meldung des Kalendertages (0:00:00 - 23:59:59 Uhr) ist, erfolgt keine automatische Vorausfüll-Funktionalität bei dem Datenfeld (wie für alle Datenfelder im Tab „Neuaufnahmen“). Falls am gleichen Kalendertag schon eine Meldung für den Meldebereich abgegeben wurde, wird der Wert mit dem entsprechenden Wert der letzten vorherigen Meldung des Tages vorausgefüllt. (siehe auch Abschnitt 9, Ausnahmen)

5O3: ((faelle_pims_aktuell  +  intensiv_betten_frei) * 2 ) +2   > 

faelle_pims_vortag_erstaufnahmen


    Regel 5P (technische Regel)
    5P1: wenn meldung.covid19StatusV3 übermittelt wird, müssen folgende veraltete Felder null sein (bzw. nicht definiert):

        faelle_covid_aktuell_beatmet

        faelle_covid_aktuell_high_flow_oxygen

        faelle_covid_aktuell_nicht_invasiv_beatmet

        faelle_covid_aktuell_ecmo

        faelle_covid_ohne_symptomatik

6) Zweifache Warnmeldung bei bestimmten Prüfregeln auf Eingabeseite

Die folgenden Prüfregeln werden nur mit einer Warnung versehen; eine Meldung kann gesichert werden, wenn diese Prüfregeln nicht erfüllt sind. 
Die Warnungen sollen zweimal erscheinen mit der Aufforderung die Angaben zu überprüfen, sollte eine Regel verstoßen werden:

    Hinweis auf Regelbruch bei der Eingabe der Werte (entweder in der Zusammenfassung oder durch ein Pop-up bei der Eingabe),

    erneute Warnanzeige nachdem der Meldende auf „Speichern“ gedrückt hat.

Wenn ein für die Regel relevantes Datenfeld nicht ausgefüllt wurde, soll die Warnmeldung nicht angezeigt werden.

    Regel 6A:
    patienten_invasiv_beatmet >=  patienten_ecmo

Regel 6A2:
faelle_covid_aktuell_beatmet >=  faelle_covid_aktuell_ecmo

Regel 6A2A:
faelle_covid_aktuell_mit_manifestation_beatmet >=

faelle_covid_aktuell_mit_manifestation_ecmo

Regel 6A2B:
faelle_covid_aktuell_ohne_manifestation_beatmet >=

faelle_covid_aktuell_ohne_manifestation_ecmo


    Regel 6B: 
    Eine Warnung auf der Zusammenfassungsseite ist anzuzeigen, wenn die Summe(Altersstrata COVID-19-Fälle) < faelle_covid_aktuell

7) Zweifache Warnmeldung bei stark veränderten Werten

Bei zu starker Abweichung von der letzten Meldung sollte die folgende Pop-up Meldung für den User erscheinen: "Sind sie sicher, dass die Eingabe bei [Label] [Wert] korrekt ist, da die Zahl sehr stark abweicht?". Diese Warnmeldungen sollten nur bei numerischen Datenfeldern angezeigt werden. Wie groß die Abweichung sein muss, damit ein Pop-up angezeigt wird, hängt von der Größenordnung der vorherigen Meldung ab:

    bei Größenordnung 0-9 vorheriger Meldungen: Pop-up nur bei Sprung von +- >= 7

    bei Größenordnung 10-19 vorheriger Meldungen: Pop-up bei Verdoppelung oder Vierteilung

    bei Größenordnung ab 20 vorheriger Meldungen: Pop-up bei Verdoppelung oder Halbierung

Eine Warnung muss für jedes Datenfeld, das eine starke Abweichung aufzeigt, einzeln angezeigt werden. Insgesamt sollen diese Warnungen mindestens zweimal erscheinen mit der Aufforderung die Angaben zu überprüfen. Erstens sollen die Warnungen bei der Zusammenfassung der Meldung erscheinen (siehe 1). Zweitens sollen die Warnhinweise erneut angezeigt werden, nachdem der Meldende auf „Speichern“ gedrückt hat.

Die Integration der Warnmeldungen setzt voraus, dass das Client-System die letzte Meldung des Meldebereichs gespeichert hat und abrufen kann.
8) Anzeige der abgefragten Datenfelder je nach Meldebereich

Alle Datenfelder werden in „Meldung erfassen“ per Default allen Meldebereichen angezeigt.

Ausnahmen sind:

1. Die Abfrage des Alters wird für Erwachsene und Kinder differenziert abgefragt (siehe auch 5G1 und 5G4):

    Bei Meldebereichen für Erwachsene bzw. ohne Behandlungsschwerpunkt wird das Alter über folgende Variablen erfasst:

        stratum17minus

        stratum18bis29

        stratum30bis39

        stratum40bis49

        stratum50bis59

        stratum60bis69

        stratum70bis79

        stratum80plus

    Bei Meldebereichen für Kinder wird das Alter über folgende Variablen erfasst:

        faelle_covid_aelter_als_17j

2. Felder zum RSV- und Influenza-Patient*innen werden nur für Kinder erfasst:

    faelle_rsv_aktuell

    faelle_rsv_aktuell_high_flow_oxygen

    faelle_rsv_aktuell_nicht_invasiv_beatmet

    faelle_rsv_aktuell_beatmet

    faelle_rsv_aktuell_ecmo

    faelle_influenza_aktuell

    faelle_influenza_aktuell_high_flow_oxygen

    faelle_influenza_aktuell_nicht_invasiv_beatmet

    faelle_influenza_aktuell_beatmet

    faelle_influenza_aktuell_ecmo

3. Felder zu PIMS/MIS-C-Patient*innen werden nur für Kinder erfasst:

    faelle_pims_aktuell

    faelle_pims_vortag_erstaufnahmen

4. Die Abfragen zu freien Kapazitäten für COVID-19/RSV/Influenza werden für Erwachsene und Kinder differenziert erfasst (siehe auch 5F3 und 5F4):

    Folgende Felder werden nur für Meldebereiche für Erwachsene bzw. ohne Behandlungsschwerpunkt erfasst:

        covid_kapazitaet_frei,

        covid_kapazitaet_frei_iv

        covid_kapazitaet_frei_ecmo

    Folgende Felder werden nur für Meldebereiche für Kinder erfasst:

        kapazitaet_frei_isolationspflichtig

5. Die Abfragen zu schwangeren COVID-19-Patient*innen werden nur für Meldebereiche für Erwachsene bzw. ohne Behandlungsschwerpunkt erfasst (siehe Regel 5L1):

    schwangere_werden_aktuell_behandelt

    anzahl_schwangere

    anzahl_schwangere_beatmet

    anzahl_schwangere_ohne_beatmung   

6. Die Abfrage zur Symptomatik wird nur für Meldebereiche für Kinder erfasst:

    faelle_covid_ohne_symptomatik

7. ECMO-bezogene Eingabefelder (in ICU-Status, COVID-19-Status) werden nicht in Meldebereichen abgefragt, die unter „Mein Krankenhausstandort/ Meldebereichsdaten“ die Check-Box des dortigen Datenfeldes „ICU ECMO vorhanden“ abgewählt haben und damit über keine ECMO-Kapazitäten im Meldebereich verfügen.
9) Funktionalität: Automatisches Werte-Vorausfüllen

Beim „Meldung erfassen“ kann eine automatische Werte-Vorausfüll-Funktionalität genutzt werden, die alle Datenfelder automatisch mit den Daten der letzten Erfassung befüllt.
Dies soll den Meldenden das Melden erleichtern, da nur Werte, die sich seit der letzten Erfassung verändert haben, angepasst werden müssen.

Ausnahmen

Für Abfragen im Reiter „COVID-19-Status: Neuaufnahmen“ erfolgt keine automatische Vorausfüll-Funktionalität, wenn die betreffende Meldung die erste Meldung des Kalendertages (0:00:00 - 23:59:59 Uhr) ist.

Falls jedoch am gleichen Kalendertag schon eine Meldung für den Meldebereich abgegeben wurde, werden die Datenfelder im Reiter „COVID-19-Status: Neuaufnahmen“ der letzten vorherigen Meldung des Tages vorausgefüllt.

Auch im Korrektur-Modus einer schon erfolgten Meldung ist ein Vorausfüllen möglich, unabhängig vom Zeitpunkt der vorhergehenden Meldung.

Dies betrifft folgende Datenfelder im Konkreten:

    faelle_covid_vortag_erstaufnahmen

    faelle_covid_vortag_verlegungen

    impfstatus_unbekannt

    impfstatus_0_impfungen

    impfstatus_1_impfung

    impfstatus_2_impfungen

    impfstatus_3_impfungen

    impfstatus_4+_impfungen

    faelle_pims_vortag_erstaufnahmen

10) Funktionalität: Nachträgliche Werte-Korrektur von erfassten Meldungen

Eine Meldung kann rückwirkend bis zu 14 Tage nach der initialen Erfassung der Meldung von allen Meldenden eines Meldebereichs korrigiert werden.
Im Frontend des Intensivregisters ist die Korrekturfunktionalität zu finden unter dem Menüpunkt Mein Krankenhaus-Standort / Mein Meldebereich: / Meldungshistorie des Meldebereichs (korrigierbar).

Prüfregeln (siehe Punkt 5), die bei der Korrektur-Anwendung nicht mehr in obiger Form gelten:

    Regeln 5H (Genesene COVID-19-Fälle) werden relaxiert: 
    Einträge die nicht den Regeln 5H1 oder 5H2 entsprechen, sind erlaubt: die Werte können kleiner werden bzw. um mehr als 50 ansteigen.

    Regeln 5I (Verstorbene COVID-19-Fälle) werden relaxiert: 
    Einträge die nicht den Regeln 5H1 oder 5H2 entsprechen, sind erlaubt: die Werte können kleiner werden bzw. um mehr als 50 ansteigen.
