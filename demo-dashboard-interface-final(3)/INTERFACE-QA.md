# Interface-Reife – Abschluss-QA

## Ziel

Die Demo wurde als konsistentes, glaubwürdiges und semantisch belastbares
B2B-SaaS-Interface überarbeitet. Grundlage waren das externe Review und die
fünf dort priorisierten Reifeschritte.

## Umgesetzte Review-Prioritäten

### 1. CSS-Namensräume und Komponentenkollisionen

- Die generische Klasse `.attention` wurde entfernt.
- Filter verwenden `.chip--attention`.
- Aufmerksamkeitssymbole verwenden gekapselte Komponentenklassen.
- Nicht mehr verwendete globale Statuspunkt-Regeln wurden entfernt.
- Navigation, Zusammenfassungen, Entscheidungsflächen und Overlay-Komponenten
  besitzen getrennte Namensräume.

### 2. Accessibility und Semantik

- Globale Suche und Command Palette sind programmatisch beschriftet.
- Drawer, Dialoge und Command Palette verwenden `role="dialog"` und
  `aria-modal="true"`.
- Fokus wird beim Öffnen in das Overlay gesetzt und beim Schliessen restauriert.
- Tab und Shift+Tab bleiben im geöffneten Dialog.
- Escape schliesst Overlays.
- Formulare besitzen programmatische Feldbezeichnungen.
- Klickbare Tabellenzeilen, Kalenderereignisse, Teamkarten und
  Einstellungszeilen wurden durch echte Buttons oder Links ersetzt.
- Toasts verwenden eine höfliche Live Region.
- Profilmenü und Menüeinträge besitzen passende Rollen und Zustände.

### 3. Mobile Priorisierung

Die mobile Heute-Seite folgt jetzt dieser Reihenfolge:

1. Tagesstatus
2. Jetzt wichtig
3. Tagesplan
4. Nächste Tage

Der Wochenkalender bleibt mobil eine aufgabenorientierte Tagesagenda.

### 4. Typografie

- Sichtbare Nutzerschrift unterschreitet 12 px nicht.
- Normale Metainformationen verwenden 12–13 px.
- Kalender, Legenden, Tabellenmetadaten und Sidebar-Gruppen wurden angehoben.
- Alle relevanten Text-/Hintergrundkombinationen erfüllen im automatisierten
  Kontrastaudit die erforderliche Schwelle.

### 5. Iconografie und Navigationsstatus

- Unicode-Navigationssymbole wurden durch ein konsistentes Inline-SVG-System ersetzt.
- Statuspunkte wurden durch sparsame Textindikatoren wie `Offen`,
  `1 Konflikt` und `2 offen` ersetzt.
- Aktive und inaktive Statusindikatoren besitzen belastbaren Kontrast.
- Einstellungen bleiben ausschliesslich über das Profilmenü erreichbar.

## Überarbeitete Seitenhierarchien

### Heute

- eine zusammenhängende Tagesübersicht statt vier konkurrierender Karten
- Tagesplan als dominante Arbeitsfläche
- konkrete Aufgaben in `Jetzt wichtig`
- kompakter Ausblick in `Nächste Tage`
- keine redundante Mehrfachanzeige derselben offenen Punkte

### Angebot & Buchbarkeit

- kompakte Bereichszusammenfassung
- eine dominante Frage: Was verhindert vollständige Buchbarkeit?
- konkrete Ursache und direkte Lösung
- Leistungen als Hauptarbeitsfläche
- ein einziger Freigabepfad
- Importverlauf klar sekundär

### Kunden und Team

- kompakte Zusammenfassungen statt zusätzlicher Kartenwand
- Listen beziehungsweise Personen bleiben die dominante Arbeitsfläche
- Datenqualität und Buchbarkeitsstatus sind direkt filter- und handlungsfähig

### Website

- kompakter Website-Status
- Vorschau als dominante Fläche
- konkrete Abweichungen statt gleichgewichtiger Statusmodule
- wiederaufnehmbarer Scan bleibt erhalten

### Analytics

- Entscheidungsmodul statt reiner Berichtsseite
- Vergleich mit der Vorperiode
- geschätzte wirtschaftliche Wirkung
- Segmentierung nach Standort, Leistung und Herkunft
- konkrete nächste Handlung

## Automatisierte Prüfungen

- 208 Seiten-/Zustands-/Viewport-Kombinationen
- Viewports: 1440 × 1000, 1024 × 900, 390 × 844 und 320 × 720
- 0 JavaScript-Laufzeitfehler
- 0 unkontrollierte globale Horizontalüberläufe
- 0 sichtbare Nutzerschriften unter 12 px
- 0 interaktive Ersatz-`div`/`tr`/`article`
- 0 unbenannte sichtbare Interaktionselemente
- 0 doppelte IDs
- 0 verschachtelte Buttons oder Links
- 0 generische `.attention`-Regeln
- 0 relevante Kontrastverstösse
- 0 Formularfelder ohne programmatische Bezeichnung
- Focus Trap, Escape und Fokusrestauration: bestanden
- Drawer-, Modal- und Command-Dialogsemantik: bestanden
- Toast-Live-Region: bestanden
- Sidebar bei niedrigen Viewports ohne Scroll: bestanden

## Ergebnis

Die Demo besitzt keine bekannten visuellen, semantischen oder technischen
P0/P1-Befunde. Sie ist weiterhin ein datenloser Offline-Prototyp und führt
keine Production-Mutation oder Bereitstellung aus.

## Kalender-Ergänzung: Samstag

- Wochenkopf auf `20.–25. Juli 2026` erweitert
- Desktop-Wochenraster zeigt sechs Spalten von Montag bis Samstag
- Samstag ist als `Sa · 25.07.` enthalten
- Rasterbreite wurde für sechs lesbare Tages-Spalten angepasst
- Mobile bleibt bewusst eine Tagesagenda und übernimmt den ausgewählten Tag
- JavaScript-Syntaxprüfung: bestanden
- keine Änderung an Filtern, Statussemantik oder Terminaktionen

## Spacing- und Robustheitsrunde

Das Interface wurde zusätzlich auf ein durchgängiges 4-Pixel-Spacing-System
normalisiert. Der vollständige Nachweis mit 424 Layoutkombinationen,
State-spezifischem Spacing-Raster-Audit, langen Inhalten, Querformaten und
Overlay-Fokusprüfung steht in `SPACING-ADVERSARIAL-QA.md`.
