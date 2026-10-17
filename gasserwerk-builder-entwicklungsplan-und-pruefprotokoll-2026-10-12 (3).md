# Gasserwerk Builder – Entwicklungsplan und Prüfprotokoll

**Stand:** 12. Oktober 2026  
**Ziel:** technische Weiterentwicklung des persönlichen Zwei-Seiten-Builders; keine gestalterische Umgestaltung.  
**Ausgangsstand:** `gasserwerk-editor-ruhig-layoutregeln.html` (ausgelieferte letzte Arbeitsversion).  
**Arbeitskopie:** `gasserwerk-builder-datensicher-v1.html`.  
**Status:** technisch überprüfter **Kandidat**, **keine uneingeschränkte Freigabe**, solange der reale GrapesJS- und CDN-Integrationstest aussteht.

## 1. Architekturgrenzen

- Eine GrapesJS-Instanz mit `editor.Pages` ist und bleibt das Seitenmodell.
- Zwei Seiten erscheinen in derselben Bearbeitungsfläche; keine zusätzliche Editorinstanz.
- GrapesJS-Projektdaten bleiben als JSON-Daten erhalten; reines HTML/CSS ist kein Ersatzformat für Projektwiederherstellung.
- Die HTML-Datei des bisherigen Builders, sein Layoutvertrag, die Pattern-Vorlagen, das DesignSystem und die GrapesJS-Version `0.22.14` wurden **nicht geändert**.
- Die aktuelle lokale Speicherung unter `gasserwerk-editor-ruhig-v1` bleibt kompatibel.
- Neue Funktionen werden in kleinen, isoliert prüfbaren Eingriffen ergänzt; kein Rewrite, kein neues Framework.

## 2. Umsetzungsplan mit klaren Abnahmebedingungen

| Phase | Eingriff | Begründung | Abnahme und Freigabegate |
|---|---|---|---|
| **A – jetzt** | **Import/Export vervollständigen**; Design und gewählte Seite wiederherstellen; Fehler mit Rollback behandeln | Ein konkreter Roundtrip-Fehler war reproduziert | Gültige Sicherung → Seiten, Design und Auswahl identisch; ungültige Sicherung oder Speicherfehler → Altzustand bleibt erhalten. |
| **B – danach** | Speicher-Snapshots mit klaren Grenzen und Konflikterkennung für mehrere Fenster | Bisher nur direkt vorheriger localStorage-Stand, konkurrierende Fenster können überschreiben | Schreibkonflikte sichtbar; mehrere Versionen gezielt wiederherstellbar; Quota-Fehler unter Kontrolle. |
| **C** | Reproduzierbare lokale Laufzeit und **echte** GrapesJS-Integrationstests | CDN-Abhängigkeit und Mock-only-Testlücke | Seitenwechsel, Drag & Drop, Undo/Redo, Import/Export und Vorschau mit echter Bibliothek und echtem DesignSystem geprüft. |
| **D** | Versionsvergleich GrapesJS 0.22.14 vs. aktuelle 0.23.x-Reihe | Potenzielle CSS-/Pages-Fixes | Ein Upgrade erst nach grüner Regression mit echten Projektkopien und Seiten-CSS. |
| **E** | Layout-Wächter **zunächst diagnostisch** | Verhindert unbeabsichtigte Selbstkorrekturschleifen | Messbare Bereichsgrenzen und Überläufe; Diagnosen zuverlässig; keine Layoutmutation ohne gesondert freigegebenen Eingriff. |

## 3. Tatsächlich ausgeführte Änderung (Phase A)

In **einer neuen Datei** ergänzt:

1. **Backup-v1-Validierung:** Erkennung von `gasserwerk-editor-backup`, Version 1, gültigen `pages`, zulässigem `tone`, `mode`, `density` und einer tatsächlich vorhandenen `selectedPage`. Unbekannte Formate und Versionen werden ausdrücklich abgewiesen.
2. **Vollständiger Import:** Neben `project` werden Design-Ton, Farbmodus, Dichte und ausgewählte Seite wiederhergestellt. Für rohe GrapesJS-Projekte ohne Backup-Umschlag bleiben die bisherigen Design-Einstellungen erhalten und die erste Seite wird gewählt.
3. **Kontrollierte Schreibfolge:** Das importierte Projekt und seine Metadaten werden in den bisherigen drei localStorage-Schlüsseln abgelegt. Bei einem gemeldeten Schreibfehler wird der vorherige Speicherzustand aus zuvor gelesenen Werten rekonstruiert; ausserdem werden Editor-Modell, Seitenauswahl und UI-Einstellungen zurückgesetzt.
4. **Event-Sperre während des Imports:** Seitenwechsel- und `update`-Events speichern während der Importoperation nicht nebenher; ein vorher ausstehendes Autosave wird nach fehlgeschlagenem Import bei ungesicherten Änderungen neu geplant.
5. **Rückfallprojekt:** Ein intakter vorheriger Projektstand wird erst **nach** erfolgreichem Import als optionale Rückfallversion gesichert. Ein Fehler in diesem zusätzlichen Schritt wird gemeldet, macht die erfolgreiche Übernahme aber nicht rückgängig.

**Nicht Bestandteil dieser Phase:** neue Speicherplattform, zusätzliche Revisionshistorie, Tab-Sperre, CMS-Publishing, Layout-Automatik, Versionsupgrade, Designänderungen.

## 4. Tests und Beweisgrenzen

- `node --check` des extrahierten Inline-JavaScript: **bestanden**.
- Playwright/Chromium: **27 Verhaltensprüfungen bestanden**, mit **explizit simulierter GrapesJS-API und simuliertem localStorage**.
- Darunter: Export–Import-Rundlauf; drei Design-Achsen; gewählte Seite; Projektinhalt; Rohprojekt-Kompatibilität; fünf fehlerhafte Backupvarianten; Ausnahme nach teilweise mutierendem Laden; Speicherfehler beim dritten Schlüssel mit Wiederherstellung; erneuter Import inklusive Bereinigung alter Fehlermeldungen; vorhandene Rückfallversion; Neustart mit den persistierten Testwerten; Ausfall der optionalen Rückfallversion; keine JavaScript-Laufzeitfehler im Test.
- Versuche, **echtes localStorage** in Chromium über `http://127.0.0.1` oder `file://` zu prüfen, wurden durch `net::ERR_BLOCKED_BY_ADMINISTRATOR` dieser Umgebung verhindert. **Nicht bestanden, sondern nicht ausführbar.**
- Die echten GrapesJS-/DesignSystem-Dateien konnten hier nicht über die CDN-Hosts geladen werden (`Could not resolve host`). **Kein visueller Integrationstest bestanden.**
- Die Atomizität der drei getrennten Browser-Schlüssel ist ein bestmöglicher Rollback bei regulär gemeldeten Schreibfehlern, **keine browserübergreifende Transaktion**. Ein harter Prozessabbruch zwischen den Schlüssel-Updates oder zwei gleichzeitige Schreibende sind noch ungeschützt. Dafür ist Phase B vorgesehen.

## 5. Betriebs- und Rückfallregel

1. Vor einem Test mit persönlichen Originalseiten stets über **„JSON-Sicherung herunterladen“** eine Kopie ausserhalb des Browsers anlegen und deren Datei prüfen.
2. Den neuen Kandidaten **parallel** zum alten Builder öffnen, **nicht** eine bestehende Arbeitskopie ersetzen. Beide verwenden **dieselben** lokalen Speicherschlüssel, sofern sie auf demselben Ursprung liegen: daher **nicht gleichzeitig mit beiden Dateien an einem Projekt arbeiten**.
3. Mit einer **Kopie** der realen Projektdaten eine Sicherung importieren; Seite und Design prüfen, erneut exportieren und Datei vergleichen; danach Browser neu starten und dasselbe bestätigen.
4. Bei Fehlern den alten Builder weiterverwenden und aus der externen JSON-Sicherung wiederherstellen. Die unveränderte Ausgangs-HTML-Datei bleibt verfügbar.
5. Eine Freigabe für produktives Arbeiten erst nach echtem GrapesJS-Test, Speichertest im Zielbrowser und Test mit einer echten Projektsicherung.

## 6. Quellen für die technischen Entscheidungen

- GrapesJS – **Storage Manager / Projekt-JSON**: https://grapesjs.com/docs/modules/Storage.html
- GrapesJS – **Editor API: `getProjectData` und `loadProjectData`**: https://grapesjs.com/docs/api/editor.html
- MDN – **Web Storage API und synchrones Speichern**: https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API
- MDN – **`storage`-Event in anderen Fenstern**: https://developer.mozilla.org/en-US/docs/Web/API/Window/storage_event

## 7. Nächster tatsächlicher Schritt

**Keine zusätzliche Funktion einschieben, bevor die Gate-Prüfung der Phase A im echten Zielbrowser abgeschlossen ist.** Danach Phase B: Konflikterkennung für mehrere Fenster und sinnvoll begrenzte Wiederherstellungspunkte; diese Phase zunächst separat testen und erst nach Erfolg übernehmen.
