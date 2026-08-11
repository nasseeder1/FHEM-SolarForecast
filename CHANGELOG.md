# Changelog

Alle nennenswerten Änderungen an diesem Modul werden hier dokumentiert.

## [Unreleased]
xx.xx.2026 Rev. xxxxx

- BEV Batteriedaten auch bei nicht aktivierten BEV-Consumer gespeichert
- Logausgabe des ausgeführten set reset Befehls zum Datenspeicher Management vor Ausgabe der Ergebnisse
- neuer Debug Modus aiData_long
- Verringerung der Log Frequenz bei Debug aiData und aiData_long
- List pvCircular, pvHistory um BEV accum_csmXX_<mode>_wseconds bzw. BEV csmXX_<mode>_points erweitert
- Aktivierung BEV opmodes 'auto' und 'prio' -> Retraining bei Verwendung bev-Flag nötig!
- Model VictronKiAPI: Fix fehlenden success-Status in Victron VRM API Forecast Response wenn vorher Response fehlerhaft war
- weitere kleinere Patches

- <b>Intern:</b> 
  * __consumerIdentityFp: opmode aus @fpkeys entfernt
  * neue Konstante BEVOPMODES
  * _aiFannBevConsumerAggregate: um BEV Modes erweitert
  * _listDataPoolPvHist, _listDataPoolCircular, _listDataPoolAiRawData erweitert
  * neue Funktion __bevConsumerOpmode
  * neue pvHistory Schlüssel:  "csm${c}_other_points", "csm${c}_prio_points", "csm${c}_auto_points"
  * neue pvCircular Schlüssel: $data{$name}{circular}{99}{"accum_csm${c}_total_wseconds"}
                               $data{$name}{circular}{99}{"accum_csm${c}_prio_wseconds"}
                               $data{$name}{circular}{99}{"accum_csm${c}_auto_wseconds"}

## [v2.9.4]
03.08.2026 Rev. 31539

- Resync Consumer Schaltstatus bei einer AUS->EIN Flanke des Automatikmodus, unabhängig ob der vorherige Schaltzustand des 
  Consumers vor einer manuellen Änderung des Schaltzustands wiederhergestellt wurde
- Mitteilungssystem: 
  * es wird immer das Icon für die Severity der letzten/aktuellsten Mitteilung angezeigt und nicht mehr die höchste 
    Severity aller vorhandenen Mitteilungen
  * Post-Icon für Schweregrad '2' geändert
- die Attribute graphicBeamXContent können fehlerfrei in den graphicHeaderOwnspec-Bereich integriert werden  
  (Bugfix in _addDynAttr: Regexfilter für statische Platzhalter korrigiert)
  
- <b>Intern:</b> 
  * Debug consumerSwitchingXX erweitert
  

## [v2.9.3]
01.08.2026 Rev. 31533

- Die reset-Funktion 'set ... reset ..' kann Daten in pvCircular suchen, löschen und bearbeiten
- Einbau hint27 und hint28 sowie Überprüfung hint12 abhängig von aiConShuffleMode und aiConShufflePeriod
- Implementierung einer sequentiellen Energiebilanz-Simulation der PV-Prognose in Abhängigkeit der PV Raw-Prognose, der Verbrauchsprognose, dem eingestellten FeedIn-Limit und der Batterie Ladungsprognose.
  Dadurch wird die Unterstützung einer Nulleinspeisung bzw. eines gesetzten Einspeiselimits in der PV Prognose realisiert.
- neue Auswahl pvForecastLimited im Attribut 'graphicBeamXContent' zur Anzeige der PV-Prognose unter Berücksichtigung einer gesetzten Einspeiselimitierung im Attribut plantControl->feedinPowerLimit

- <b>Intern:</b> 
  * _calcConsForecast_legacy: eigener consForecastBase-Durchlauf auf conraw, konsistent zu confc/confcex 
  * neuer Wert 'pvfcfeedlim' in Datenpool pvHistory und NextHours -> wird in Balkengrafik mit Fallback auf 'pvfc' verwendet
  * Kompatibilitätsfix feature 'state' für ältere Perl Versionen  


## [v2.9.2]
27.07.2026 Rev. 31518

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

## [v2.9.1]
19.07.2026 Rev. 31499

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
