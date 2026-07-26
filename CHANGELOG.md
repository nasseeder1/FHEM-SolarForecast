# Changelog

Alle nennenswerten Änderungen an diesem Modul werden hier dokumentiert.

## [Unreleased] - Weiterentwicklung

- Einbau hint26 mit Erkennung unterer Grenze von aiControl->aiConLearnRate
- neuer Schlüssel consumerControl->iconFix zur statischen Darstellung der Verbraucher-Icons, schaltet die Dynamik entsprechend Verbrauchsstatus ab
- der Ready-Status der Fann-KI wird sprachensensitiv ausgegeben
- Einbau UTF8 Encoding im GUI Popup (Fix Darstellung Umlaute)
- Ergänzung Datensammlung und Training für consumerXX->type heatpump->opmode 'eco'
- vermeide zu wenig Datensätze im Drift-Retrain Prüfungskontext
- Änderung plantControl->writeForceType: 'file' ist Standardspeicher, 'auto' ist deprecated, verwende 'db' anstatt (incl. BugFix FileRead)
- Integration initialen Cache-Load 'initfirst' um vor dem Laden weiterer Daten Voreinstellungen festzulegen
- Setter 'reset consumptionHistory' in 'reset consumptionShort' umbenannt
- das Auslesen des Verbindungsstatus zum öffentlichen Netz kann in setupEnvironment gesetzt und mit graphicControl->headerShowEnv im Kopf angezeigt werden

- <b>Intern:</b> <br>
  * writeCacheToFile nach writeCacheFile umbenannt -> restart nötig <br>
  * Modulkopf neu strukturiert <br>
  * delConsumerFromMem um fehlende Verbraucherschlüssel ergänzt <br>
  * Commandref bearbeitet <br>
  * neue Funktion checkDevRdCond <br>
  * use feature 'state' gesetzt und Code Rework <br>

## [v2.9.1] - 19.07.2026

- neuer FEATURE BLOCKS semantics_heatpump_nopv (Beseitigung Gap in heatpump-Profilen ohne pv)
- neuer Befehl "set .. reset aiData setValue ...". Damit können aiRawdata-Werte gezielt korrigiert werden.
- das Gemini Model kann im Schlüssel aiControl->geminiAPIkey nach dem API-Key angegeben werden (default: gemini-2.5-flash)
- die Victron API ((Model VictronKiAPI) kann nun den neuen Token-Auth verwenden. Der Befehl "set ... vrmCredentials" entsprechend erweitert


### Hinweise zur Nutzung

- Neue Einträge immer oben unter `[Unreleased]` ergänzen, während du entwickelst
- Kurze Stichpunkte reichen – Zweck ist die eigene Erinnerung, keine formelle Doku
- Beim Release: Abschnitt umbenennen zu `[vX.Y.Z] - TT.MM.JJJJ`, Text 1:1 in die
  Forgejo-Release-Beschreibung kopieren, danach neuen leeren `[Unreleased]`-Block
  oben ergänzen
