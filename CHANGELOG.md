# Changelog

Alle nennenswerten Änderungen an diesem Modul werden hier dokumentiert.

## [Unreleased] - Weiterentwicklung (Planrelease: 2.9.1)

- neuer FEATURE BLOCKS semantics_heatpump_nopv (Beseitigung Gap in heatpump-Profilen ohne pv)
- neuer Befehl "set .. reset aiData setValue ...". Damit können aiRawdata-Werte gezielt korrigiert werden.
- das Gemini Model kann im Schlüssel aiControl->geminiAPIkey nach dem API-Key angegeben werden (default: gemini-2.5-flash)
- die Victron API ((Model VictronKiAPI) kann nun den neuen Token-Auth verwenden. Der Befehl "set ... vrmCredentials" entsprechend erweitert



## [v1.1.0] - 13.07.2026

- (Beispiel: hier steht später der Inhalt, der beim Release v1.1.0 aus "Unreleased" übernommen wurde)

---

### Hinweise zur Nutzung

- Neue Einträge immer oben unter `[Unreleased]` ergänzen, während du entwickelst
- Kurze Stichpunkte reichen – Zweck ist die eigene Erinnerung, keine formelle Doku
- Beim Release: Abschnitt umbenennen zu `[vX.Y.Z] - TT.MM.JJJJ`, Text 1:1 in die
  Forgejo-Release-Beschreibung kopieren, danach neuen leeren `[Unreleased]`-Block
  oben ergänzen
