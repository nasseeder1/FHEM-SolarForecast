# Änderunglog 76_SolarForecast

Alle nennenswerten Änderungen an diesem Modul werden hier dokumentiert.

## [unreleased]
xx.xx.xxxx Rev. xxxxx


## [v2.11.0]
09.11.2026 Rev. xxxxx

- **Fix:**
  * Speicher: Die LRU-Caches der History-Arrays ('BEVHISTARR', 'CYCLECSMHISTARR',
    'HPOPMODEHISTARR', 'PVHISTARR') behielten pro Stunde einen neuen Eintrag und
    hielten dadurch bis zu 16 Stundenversionen vor, von denen nur die aktuelle noch
    gelesen wird. Veraltete Stunden-Einträge derselben Serie werden jetzt beim
    Anlegen des neuen Eintrags entfernt (neue Funktionen 'LRU_delete' und
    'LRU_purge_stale').
  * Speicherleck: '___ownSpecGetFWwidget' fügte versteckte Readings bei jedem Aufruf erneut
    in das globale Array '@widgetreadings' ein. Das Array wuchs dadurch unbegrenzt
    und verursachte beim Tageswechsel (Task 2) einen RSS-Sprung. Die Merker werden jetzt als Hash je Device
    ('%widgetreadings') geführt, Duplikate entfallen und jedes Device löscht nur seine eigenen Hilfsreadings.

- **Änderung:**
  * Reading **Battery_OptimumTargetSoC_XX** entfernt
  * Speicher: Serien, die nur aus Nullen bestehen (z. B. BEV- oder Wärmepumpen-Historie
    bei Anlagen ohne entsprechenden Verbraucher), werden im Cache nur noch als Länge
    abgelegt und beim Lesen wieder zu einem Array expandiert ('_aiPackSeries',
    '_aiUnpackSeries'). Der Aufrufer erhält weiterhin eine eigene Kopie.
  * Speicher: Die Größe des 'TS_OFFSET_Cache' wurde von 4000 auf 700 Einträge
    reduziert. Der Cache legte pro Sekunde einen neuen Eintrag an, der praktisch nur
    innerhalb derselben Sekunde wiederverwendet wird.
  * Performance: Die Debug-Prüfung in 'LRU_get', 'LRU_insert' und 'LRU_evict_tail'
    verwendet 'index()' statt eines bei jedem Zugriff neu kompilierten regulären
    Ausdrucks.

- **Diagnose**
  * Neue Zähler 'CNT_BEVHIST_HIT' und 'CNT_BEVHIST_MISS' in '$hash->{HELPER}' für die
    Aufrufe von '_aiFannBevHistArray'.
  * Neue Hilfsfunktion '__logDailyTask': Protokollierung des Prozessspeichers (VmRSS) zu Beginn
    und Ende der täglichen Sonderaufgaben (Tasks 2 bis 7), Verbose 3.


## [v2.10.11]
04.11.2026 Rev. xxxxx

- **Neu:**
  * **Eigener Regenmengenmesser**: Im Attribut 'setupEnvironment' kann mit dem neuen Schlüssel 'rainAmount=<Device>:<Reading>' ein eigener Regenmesser angegeben werden. Das Reading muss die Regenmenge der aktuellen Stunde in mm (l/m²) liefern.
    - Der Messwert überschreibt für die aktuelle Stunde (NextHour00) den Niederschlagswert 'rr1c' der Wetter-API. Er wird damit auch in 'nexthours', 'circular' und 'pvHistory' übernommen, und damit auch in den AI-Rohdaten.
    - Ist das Device oder Reading nicht vorhanden, nicht numerisch oder negativ, bleibt der API-Wert erhalten.
  * **Header-Anzeige**: 'rainAmount' ist als Wert in 'graphicControl' → 'headerShowEnv' auswählbar. 
      Angezeigt wird die Regenmenge der aktuellen Stunde in mm, mit dem Messwert oder ohne eigenen Messer mit dem API-Wert. 
	  Das Symbol wechselt nach Menge zwischen 'rain_light' (< 2,5 mm), 'rain' (2,5 bis < 7,6 mm) und 'rain_heavy' (≥ 7,6 mm). 
	  Bei 0 mm wird das Symbol grau dargestellt.
    - Commandref (DE/EN) für 'setupEnvironment' und 'headerShowEnv' ergänzt.
  * Patch Subs _aiFannBevConsumerAggregate, _aiFannBevHistArray, _aiFannBuildLagFeatures, aiFannConDataLoad, aiFannConInfer, 
    _aiFannFeatureBuilder, FEATURE_BLOCKS
  * **KI Verbrauchsprognose**: BEV Phasenanzahl (bevcsmPhasesXX, Consumer-Schlüssel phases) als Lag1-Feature bev_phases_lag1 
      (mittlere Phasenanzahl der ladenden EVs relativ zu 3 Phasen, nur Stunden mit Ladung), neuer Feature-Block bev_phases in 
	  allen Profilen mit BEV
      Hinweis: der Eingangsvektor wächst bei BEV-Profilen um ein Feature -> neues Training der KI erforderlich

- **Fix:**
  * Die Hilfsfunktionen _aiFannBevHistArray, _aiFannHpOpmodeHistArray und _aiFannCycleConsumerHistArray haben ihre Ergebnis-Arrays als Referenz sowohl in den
    pvHistCache gelegt als auch an den Aufrufer zurückgegeben. Da aiFannConInfer diese Arrays im Prognosehorizont per push fortschreibt, wurden die gecachten Arrays bei
    jedem Lauf innerhalb derselben Stunde verlängert (Speicherwachstum). Folgeläufe arbeiteten zudem mit verlängerten, nicht mehr zu @flat_targets ausgerichteten
    BEV-/WP-Serien, was die lag1-Features verfälschen konnte. Cache und Rückgabewert verwenden jetzt getrennte Kopien (Cache-Hit und Cache-Insert).
  * getPvHistTargetArray: im Cache-Miss-Fall werden nun ebenfalls Kopien gecacht, sodass der Aufrufer nie dieselben Arrays wie der Cache erhält (konsistent zum Cache-Hit-Pfad).
  * timestringsFromOffset: Cache-Hit und Cache-Insert verwenden eine Kopie des Hashes, damit Änderungen des Aufrufers den Cache nicht verändern können.

- **Änderung:**
  * Parser __parseAttrEnvironment gehärtet
     

## [v2.10.10]
23.10.2026 Rev. 31830

- **Fix:**
  * Perl-Warnungen "Use of uninitialized value" in __batChargeOptTargetPower und
    ___batRemainHodsAndSurp, die bei sehr geringer PV-Prognose (kein PV-Überschuss in einzelnen
    Stunden) auftraten. Fehlende Überschusswerte werden jetzt als 0 Wh behandelt, die Ladelogik
    bleibt unverändert. Zusätzlich wurde die Operatorpriorität des "// 0"-Fallbacks bei $replacement
    und $limpower korrigiert.
    Forum: https://forum.fhem.de/index.php?msg=1370567


## [v2.10.9]
20.10.2026 Rev. 31822

- **Fix:**
  * __openMeteo_ApiResponse: Klammerung gefixt, Forum: https://forum.fhem.de/index.php?msg=1370333 
  * Message-System: Fix fehlerhafte Sortierung mit idx >=10
  

## [v2.10.8]
11.10.2026 Rev. 31766

- **Fix:**
  * Batterie-Ladesteuerung (optPower/smartPower bei safetyMargin=0): Ist das Ladeziel nicht erreichbar (Ratio <= 100 %), 
    wird jetzt immer mit der maximalen Ladeleistung (bpinmax) geladen. Bisher wurde dieser Fall bei safetyMargin=0 (otpMargin=0) 
	nicht erkannt und die Ladeleistung fälschlich auf den prognostizierten Mindestwert begrenzt.


## [v2.10.7]
10.10.2026 Rev. 31760

- **Änderung:**
  * _aiFannDriftHistSlice: Zeitreihe für die Drift-Analyse direkt aus pvHistory aufgebaut, nur nutzbare Slots (Ist- und AI-Wert vorhanden) werden belegt und gezählt
  * Driftanalyse (aiFannDetectDrift): Ersatz der Quelle airaw durch Werte aus pvHistory (_aiFannDriftHistSlice) -> Verringerung RAM Footprint
  * CachedHistoryVal: Zeitkontext in timestringsFromOffset auf die Minute gerundet, damit pro Minute statt pro Sekunde nur ein 
    Eintrag im TS_OFFSET_Cache entsteht (weniger Cache-Misses und Verdrängungen, weniger Speicher-Churn)


## [v2.10.6]
08.10.2026 Rev. 31752

- **Neu:**
  * _calcDataEveryFullHour: Einbau einmalige Größenausgabe der internen Datenstrukturen (auskommentiert)
  * Reading Battery_OptimumBaseSoC_XX parallel zum bestehenden Reading Battery_ChargeOptTargetPower_XX welches abgelöst werden soll (Forum: https://forum.fhem.de/index.php?msg=1369429)
  * sub mallocTrim -> Rückgabe aller freigegebenen Arenen
  * Einbindung mallocTrim in centralTask Nachtverarbeitung Task 4

- **Änderung:**
  * __delObsoleteAPIData: Speicheroptimierung der Funktion
  * Nachtverarbeitung: aiDelRawData aus Task 6 nach Task 4 verschoben
  * Wrapper AIF_modelRun gehärtet und Fehlerausgabe in _aiFannPredict verbessert (Forum: https://forum.fhem.de/index.php?msg=1369865)
  * Änderungen von MCache_get, MCache_set für Robustheit und echtes FIFO, Anpassung der Aufrufe von MCache_get
  * Parametrisierung des Aufrufs HttpUtils_NonblockingGet sowie return Funktion geändert ($caller)
  * Änderung der Anzeigereihenfolge im Mitteilungssystem: neueste Mitteilungen stehen ganz oben
  

## [v2.10.5]
27.09.2026 Rev. 31691

- **Fix:**
  * _createReadingsFromArrayFast: exists Prüfung zur Verhinderung Auto-Vivification (Forum: https://forum.fhem.de/index.php?msg=1369271)
  * removeMinMaxArray: fix limit
  * aiAddInstance (AI::DecisionTree): push gebinntes sunalt in @pvhdata
  
  * Neural Network Wrapper neue Wrapper‑Methode
    AIF_isModelValid()  
    Neue Validierungsmethode, die das FANN-Modell leak-frei prüft.
    Verwendet get_num_input() statt MSE(), da diese XS-Methode keine Exceptions wirft und keine temporären Referenzen erzeugt.
    
  * Absturzgefahr (Segfault) bei Verwendung von gebrochenen 'AI::FANN'-C-Pointern aus wiederhergestellten Storable-Cache-Dateien behoben.
  * Falsch-positive I/O-Fehlermeldungen beim Auslesen gültiger Cache-Dateien mit Null-Werten behoben.
  * _calcDataEveryFullHour: durch Array-Kopie bedingtes Speicherleck beseitigt
    
- **Änderung:**
  * Anpassung bezüglich OpenMeteo API Änderung für OpenMeteoDWDEnsembleAPI
  * Der Befehl "get ... rooftopData" kehrt bei Erfolg anstatt bisher undefiniert mit der Meldung 
    "A data retrieval request for the selected radiation and/or weather API has been triggered" (zweisprachig EN/DE)
	zurück.
  * __aiAddRawData: writeCacheFile in den BlockingCall Wrapper writeCacheFileBlocking eingebettet -> Vermeidung Speicherleck
  * Der Befehl "get ... data" meldet in EN oder DE zurück
  
  * **AiFannModelWrapper**: Speichersicherheit bei der Serialisierung über 'Storable' deutlich erhöht.
    - Implementierung von 'STORABLE_freeze' und 'STORABLE_thaw' Hooks, um ungültige C-Pointer/Memory-Leaks nach Demaskierung (Deserialisierung) zu verhindern und FHEM vor Segmentation Faults zu schützen.
    - Überarbeitung der Methode 'AIF_isModelValid' zur besseren Unterscheidung zwischen Objekt- und Klassenaufrufen.
  
  * **fileStore / fileRetrieve**: Fehlerbehandlung und Evaluierung geglättet.
    - Umstellung auf das 'eval { ... 1; } or do { ... }' Pattern, damit skalare Falsy-Werte ('0', '""') nicht fälschlicherweise als I/O-Fehler interpretiert werden.
    - Expliziter Guard-Check bei Nichtexistenz von Dateien in 'fileRetrieve'.
  
  * **readCacheFile**: Ressourcenverwaltung für 'AI::FANN'-Modelle verbessert.
    - Vor dem Laden neuer Cache-Daten werden alte XS-Objekte nun explizit via 'AIF_modelDestroy()' freigegeben.
  
  * **Serialize / Deserialize**:
    - 'Deserialize' um Guard-Clauses gegen leere/unverarbeitbare Eingaben ergänzt sowie Fehler-Logging robuster gestaltet.
  
  * removeMinMaxArray, limitArray, _addCon2CircArray, __aiAddRawData, aiFannDetectDrift, medianArray, _calcCaQcomplex, _calcDataEveryFullHour, _aiFannSlopeBias, getPvHistTargetArray, LRU_reset, LRU_evict_tail, __calcNewFactor_migrated, __readConFromCircular refaktoriert
  

## [v2.10.4]
20.09.2026 Rev. 31670

- **Fix:**
  * AI::FANN Speicherleck durch globales DESTROY-Patching behoben.
    writeCacheFile neutralisierte AI::FANN::DESTROY permanent und prozessweit,
    so dass kein AI::FANN-Objekt (Training, Reload, Inferenz) nach dem ersten
    Speichervorgang seinen C-seitigen Speicher jemals freigab. Behoben durch
    Einführung eines schlanken Perl-Wrappers (AiFannModelWrapper), der das
    XS-Objekt kapselt und dessen Freigabe sauber über Perls Referenzzählung
    steuert – ohne globale Seiteneffekte.

- **Änderung:**
  * _batSocTarget: Debuglog für Step6 korrigiert


## [v2.10.3]
19.09.2026 Rev. 31664

- **Änderung:**
  * plantControl ->consForecastBase: Das Verfahren zur Anwendung des Basiswerts ist jetzt über den optionalen Token 
    'Mode->Base' (Default) bzw. 'Mode->AddOn' steuerbar. Base hebt Prognosen unterhalb des Schwellwerts an (bisheriges Verhalten). 
	AddOn addiert den Basiswert als festen Aufschlag auf die berechnete Prognose - unabhängig von deren Höhe. Rückwärtskompatibel.
  * Online-Hilfe für Schlüssel consForecastBase präzisiert

- **Fix:**
  * Flowgrafik: 
    Korrektur der Darstellung bei Netzladung der Batterie über den Hausknoten. 
    Ein negativer Energiefluss zwischen Inverterknoten und Hausknoten wird nun korrekt als direkter Fluss Hausknoten → Batterie dargestellt. 
    Der angezeigte Hausverbrauch bleibt davon unberührt. (__calcVectorConsumption + _flowGraphic)
    
  * SOC-Prognose LR überschätzt erreichbaren Ladestand wenn aktueller SoC < batoptsocwh
    ___batClampValue snappte den prognostizierten SoC im Ladefall bedingungslos auf batoptsocwh,
    wodurch bpinmax wirkungslos war und 100% SoC bereits nach wenigen Stunden prognostiziert
    wurde. Im Ladefall (delta >= 0) wird nun nur noch auf [lowSocwh, batinstcap] begrenzt;
    der Snap-up auf batoptsocwh bleibt ausschließlich dem Entladefall vorbehalten.
	   
  * readCacheFile - RAM-Peak beim Nachladen von KI-Modellen reduziert (Forum: https://forum.fhem.de/index.php?msg=1368934)
    Beim Reload von 'aitrained', 'airaw' und 'neuralnet' lagen kurzzeitig zwei vollständige
    Modelle gleichzeitig im Heap (altes Objekt + neu deserialisiertes). Auf speicherschwachen
    Systemen (z.B. Raspberry Pi 2, ~1 GB RAM) konnte dies den OOM-Killer triggern.
    Behoben durch gezieltes Freigeben der Vorgängerdaten unmittelbar vor fileRetrieve:
    'aitrained'/'airaw': delete der alten Hash-Referenz vor dem Retrieve (Guard: -s $file).
    'neuralnet': selektives Löschen der XS-seitigen FannModel-Objekte (nicht serialisierbar,
    dominanter RAM-Anteil); Blob-Daten und Validierungslogik bleiben unverändert.

  * _aiFannPercentileBasedLimits: der overshoot-Faktor (raw_max / p999) hob den
    Percentile-Clip rechnerisch auf raw_max zurück, womit der Double-Percentile-Filter
    wirkungslos wurde; targmaxval wird jetzt korrekt als p999 * 1.05 berechnet
  
    _aiFannNormAsymFixRange: fehlender Hard-Clamp [0,1] ließ normierte Werte > 1.0
    ins Netz laufen wenn Targets den targmaxval überschritten; Werte werden jetzt hart auf
    [0,1] begrenzt
	
    Beide obiges Fixes zusammen aktivieren das Percentile Clipping für BEV-bedingte Heavy-Tail-
    Verteilungen im Trainingstarget korrekt
	 
- **Neu:**
  * Kreuzvalidierung stepSoC * careCycle in ctrlBatSocManagementXX
    Das Produkt aus stepSoC und careCycle muss 100 ergeben (oder stepSoC=0
    zur Deaktivierung des SoC-Managements). Ungültige Kombinationen werden
    beim Setzen des Attributs abgewiesen. Gültige Paare: 1/100, 2/50, 4/25,
    5/20, 10/10, 20/5, 25/4, 50/2, 100/1.
    Der zulässige Wertebereich von stepSoC wurde auf die ganzzahligen Teiler
    von 100 im Bereich 1..100 erweitert (zuvor: 0..5).
    
  * Schlüssel plantControl->plantCoordinates hinzugefügt, um mehrere SF-Instanzen
    an verschiedenen Standorten innerhalb eines FHEM-Systems zu unterstützen.
    Format: latitude-><Wert>, longitude-><Wert>
    Wenn dieses Attribut gesetzt ist, haben die gerätespezifischen Koordinaten Vorrang vor den Werten,
    die im globalen Device definiert sind. Die Höhe (altitude) ist hier nicht konfigurierbar
    und muss im globalen Device festgelegt bleiben (erforderlich für das Astro-Modul).
    Die Eingabe wird beim Setzen validiert; unbekannte Schlüssel werden abgelehnt.
	   


## [v2.10.2]
30.08.2026 Rev. 31608

- userExit bzgl. zirkulären Referenzen gehärtet 
- potenzielle Speicherleaks geschlossen
- _aiFannAutoArchitecture: Warnung durch undefiniertes dataParamRatio beseitigt
- _aiFannEpochDiagnostic: neuer hint29, very_early-Zweig: hint1 und hint26 zusaätzlich gated, 
  early-Zweig: hint5 und hint23 zusätzlich gated (Forum: https://forum.fhem.de/index.php?msg=1368313)


## [v2.10.1]
22.08.2026 Rev. 31587

- writeCacheFile: singleUpdateState entfernt (Forum: https://forum.fhem.de/index.php?msg=1368075)
- weitere singleUpdateState in Getter entfernt

- **Fix:**
  * isGhoValFormValid geändert: die Prüfung erfolgt nun zuverlässig bei Eingabe des graphicHeaderOwnspecValForm-Attributs 


## [v2.10.0]
16.08.2026 Rev. 31573

- **Neu:**
  * Ausgabe des ausgeführten set reset Befehls zum Datenspeicher Management vor Ausgabe der Ergebnisse im Log
  * neuer Debug Modus aiData_long
  * List pvCircular, pvHistory um BEV accum_csmXX_<mode>_wseconds bzw. BEV csmXX_<mode>_points erweitert
  * List pvHistory, aiRawData um Anzeige bevcsmPhasesXX erweitert
  * vollständige Pipeline-Integration (Training + Inferenz) für die BEV opmode-Fraktionen 'auto' und 'prio' -> ACHTUNG: Retraining bei Verwendung bev-Flag nötig!
  * BEV-Consumer: Aufzeichnung der zum Laden verwendete Anzahl Phasen – reine Rohdatenerfassung für später (ToDo: in Training noch einzubauen)
  * neuer Get-Befehl 'stepTimes' zur detailliierten Anzeige von Phasenzeiten
  * Sun Position Caching integriert
  
- **Änderung:**
  * BEV Batteriedaten werden auch bei nicht aktivierten BEV-Consumer gespeichert
  * Verringerung der Log Frequenz bei Debug aiData und aiData_long
  * ACHTUNG: Funktionsänderung storeReading ('<Readingname>', '<Wert>') -> storeReading ($name, '<Readingname>', '<Wert>')
  * weitere kleinere Patches

- **Fix:**
  * Model VictronKiAPI: fehlender success-Status in Victron VRM API Forecast Response wenn vorher die Response fehlerhaft war

- <b>Intern:</b> 
  * __consumerIdentityFp: opmode aus @fpkeys entfernt
  * neue Konstante BEVOPMODES
  * neue Funktion checkModVerBatch
  * _aiFannBevConsumerAggregate: um BEV Modes erweitert
  * _listDataPoolPvHist, _listDataPoolCircular, _listDataPoolAiRawData um BEV-Opmode-Daten erweitert
  * Verwendung createReadingsFromArrayFast statt createReadingsFromArray
  * neue Funktion __bevConsumerOpmode
  * neue pvHistory Schlüssel:  "csm${c}_other_points", "csm${c}_prio_points", "csm${c}_auto_points"
  * neue pvCircular Schlüssel: $data{$name}{circular}{99}{"accum_csm${c}_total_wseconds"}
                               $data{$name}{circular}{99}{"accum_csm${c}_prio_wseconds"}
                               $data{$name}{circular}{99}{"accum_csm${c}_auto_wseconds"}
  * __bevConsumerOpmode: Punktesystem prio/auto/other mit cactive-Gating, korrektem Stundenwechsel-Reset, Fallback über csme für Altinstallationen, plus Phasenerfassung als reiner Snapshot				   
						

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

- Neue Einträge immer oben unter '[Unreleased]' ergänzen, während du entwickelst
- Kurze Stichpunkte reichen – Zweck ist die eigene Erinnerung, keine formelle Doku
- Beim Release: Abschnitt umbenennen zu '[vX.Y.Z] - TT.MM.JJJJ', Text 1:1 in die
  Forgejo-Release-Beschreibung kopieren, danach neuen leeren '[Unreleased]'-Block
  oben ergänzen
