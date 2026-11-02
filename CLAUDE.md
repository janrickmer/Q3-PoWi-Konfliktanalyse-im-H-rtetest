# Escape-Room „Konfliktanalyse im Härtetest“ – Hinweise für die Entwicklung

Escape-Room für Politik und Wirtschaft, Qualifikationsphase Q3 (Jahrgangsstufe 13, Grundkurs, Kurshalbjahr Q3.1 Internationale Konflikte): Konfliktanalyse am Beispiel des Irankriegs 2026 und des Bürgerkriegs im Sudan. Story: Im Lagezentrum (Referat 3 Krisenlagen) ist die Referatsleiterin Dr. Nadja Holm ausgefallen; die Schüler:innen prüfen als Analyst:innen ihr Lagebild „Iran 2026“ (Sektionen 1–3) und erstellen bis zur Sitzung des Krisenstabs um 18:00 Uhr das fehlende Lagebild „Sudan“ (Sektionen 4–6) – mit jeder freigegebenen Sektion rastet ein Segment des Freigabesiegels ein. Sechs Stationen
(Bibliotheksraum → Rätselraum, kein Weg zurück), je ein Buchstabe für das Codewort SIEGEL, am Ende der Arena-Tunnel.
Oberfläche deutsch, Schüler:innen werden geduzt, echte Umlaute und typografische Anführungszeichen („…“).

## Aufbau

- `index.html` – der ganze Raum in einer Datei (CSS, HTML-Gerüst, ein klassisches `<script>`), keine externen Bibliotheken
  oder CDNs. Reihenfolge im Skript: Konfiguration (`CODE_WORD`, `MAGIC`, `FEIBEITSBLATT_*`, `ESCAPE_ROOM`) → `ROOMS` →
  Dimensionen/HIIK-Stufen → Rätseldaten `PUZZLE_R1` … `PUZZLE_R6` → Zustand → Hilfsfunktionen → Labels/Lösungserläuterungen
  (`ROOM_LABELS`, `SOLUTION_NOTES`) → PDF-Writer → Überblick-PDF (`generateMistakesPdf`) → Datensatz → Siegel-Leiste →
  Bildschirme (Start, Einführung, Bibliotheksräume mit `LIBRARIES`, Rätselbausteine, Material der Sudan-Sektionen, Lösungen
  und Hinweise, Rätselräume 1–6, Enthüllung, Finale) → Download der Überblick-PDF → Tunnel → Router → Zwischenstand →
  `renderStart()`.
- Bildschirme: `start` → (`resume`) → `intro` → `lib1` → `room1` → … → `final`. Keine History-Einträge, kein Zurück vom
  Rätselraum in die Bibliothek. Bibliothekstexte, Rätsel und Lösungserläuterungen sind von der Lehrkraft freigegeben und
  wörtlich übernommen – Inhalte nur dort ändern.
- Videos: YouTube erst nach Klick (youtube-nocookie, Lader im `<head>`), daneben ein Link, der das Video direkt auf YouTube öffnet (neuer Tab).
  Einführung („Anleitung: Escape-Room meistern“) und Arena („Anleitung: Das Feiditor-Feibeitsblatt als Endgegner“) nutzen die
  Anleitungsvideos der Reihe; Bibliotheksraum 4 hat ein eigenes Erklärvideo (siehe „Besonderheiten dieses Raums“). Nie „von
  deiner Lehrkraft geprüft“ schreiben – den öffentlichen Raum setzen auch andere Lehrkräfte ein.
- `fonts/` – Manrope, Source Serif 4 und JetBrains Mono als woff2 (Lizenztexte in `fonts/LIZENZEN.txt`).
- `impressum.html` – Impressum & Datenschutz. Anpassen, wenn sich Datenflüsse ändern.
- `CNAME` – `escape-room-q3-powi-konfliktanalyse-im-haertetest.janrickmer.de` (GitHub Pages, Branch `main`).

## Endgegner: das Feiditor-Feibeitsblatt

Der Raum gehört zur Reihe janrickmer/escape-room-* und arbeitet wie der Escape-Room „FastFashion“ (Jg. 10): **kein
eingebetteter Feiditor, keine Abgabe bei Moodle und keine Auswertung im „Check des Feind(t)es“** – also auch kein
`REFLECT_TASKS`, `RED_FIELDS`, `ROOM_CONTEXT`, `%ESCAPEDATA`. Der Endgegner ist das **Feiditor-Feibeitsblatt** (Repository
janrickmer/Feibeitsblatt): Der Arena-Tunnel öffnet es im selben Tab und übergibt den Datensatz im Link. Der
Bestätigungscode steht nur in der Überblick-PDF (und im Datensatz).

## Schnittstelle v1 zum Feibeitsblatt

- Datensatz `feibeitsblatt-escape`: Hülle `{kind:"feibeitsblatt-escape", v:1, d, s}`; `d` = JSON-String der Daten,
  `s` = Prüfsumme `fbEscapeHash('FbEscape|v1|' + d)` (64 Bit als 16 Hex, Testvektor `fbEscapeHash('FbEscape|v1|{}')` =
  `0bdda452e8d9ff95`). Die Prüfsumme ist nur ein Indiz gegen versehentliche Änderungen, kein Schutz.
- Felder von `d` in dieser Reihenfolge: `code`, `room` {id, name, url}, `person` {vorname, nachname, geburtsdatum TT.MM.JJJJ,
  klasse}, `confirm`, `word`, `completedAt` (ISO), `stations` [{label, attempts, errors [{item, wrong?, right?, note?}]}],
  `notes` [{label, text}], `resumes` [{place, savedAt, loadedAt}]. Die Fehler des ersten Versuchs werden aus den
  gespeicherten Fehlerobjekten (`kind`, `item`, `wrongAnswer`, `correctAnswer`, `explanation`) in Textform gebracht
  (`errorToRecord`), mit demselben Wortlaut wie in der Überblick-PDF. Grenzen wie in der CLAUDE.md des Feibeitsblatts
  (Hülle ≤ 96 KB u. a.).
- Übergabe: Tunnel → `location.assign(FEIBEITSBLATT_URL + '#escape=' + base64url(UTF-8(Hülle)))` (`buildEscapeRecord`,
  `b64url`, `enterTunnel`); Überblick-PDF → eingebettete Datei `feibeitsblatt-escape.json` (`/Type /EmbeddedFile`,
  Kennzeichen `/FbEscape 1`, unkomprimiert, im `/EmbeddedFiles`-Namensbaum und in `/AF`).
- Änderungen an der Schnittstelle nur in beiden Repositorys gemeinsam (neue Version `v`).

## Konstanten

- `FEIBEITSBLATT_URL = 'https://feibeitsblatt.janrickmer.de/'`, `ESCAPE_ROOM = {id:'powi-q3-konfliktanalyse-haertetest', name:'Konfliktanalyse im Härtetest', url:'https://escape-room-q3-powi-konfliktanalyse-im-haertetest.janrickmer.de/'}`.
  Legt die Lehrkraft das Escape-Blatt im Feibeitsblatt an, trägt sie dort genau diese Kennung (`id`) und diesen Namen ein.
- **`FEIBEITSBLATT_CODE`** – Code des Escape-Blattes im Feibeitsblatt (6 Zeichen ohne Bindestrich). Zurzeit **leer**: Das
  Finale ist gesperrt, bis die Lehrkraft das Escape-Blatt anlegt und den Code hier einträgt. Legt sie das Blatt neu an (z. B.
  nach dem Löschen oder weil es mit fremden Abgaben gefüllt wurde – der Code ist öffentlich), hier den neuen Code eintragen.
  Solange er leer ist, ist der Tunnel gesperrt und die Überblick-PDF trägt `code: ""`; das Feibeitsblatt nimmt eine solche PDF
  nie an. Deshalb raten Sperrhinweis, Pause-Karte, der Satz über der Speicherleiste und der Hinweis in der PDF im gesperrten
  Zustand zur Zwischenstand-Datei des Raums (Knopf direkt unter dem Sperrhinweis), nicht zur Überblick-PDF; die Ladeseite
  „Weiterspielen“ nennt die Überblick-PDF dann nicht, und eine dort hochgeladene PDF bekommt die Meldung, dass hier die
  Zwischenstand-Datei (.txt) gebraucht wird.
- Testhaken nur für automatische Tests: Ist vor dem Laden `window.__FF_TEST__ = {feibeitsblattUrl, code}` gesetzt (Playwright
  `addInitScript`), gelten diese Werte. Produktiv nie setzen.
- Labels „Sektion 1 · Das Schema: sechs Dimensionen (Iran)“ (`ROOM_LABELS`, `SOLUTION_NOTES`) nie ändern – die Aufträge an die KI im Feibeitsblatt verweisen
  wörtlich darauf.
- `SAVE_FORMAT = 2`: Zwischenstand-Dateien tragen seit dem Umbau die Klasse; Dateien der Fassung 1 (ohne Klasse) werden mit
  einer eigenen Meldung abgelehnt.

## Regeln

- Jeder Versuch zählt; die Fehler des ersten Versuchs stehen in Textform (`item`, `wrong`, `right`, `note`) in Überblick-PDF
  und Datensatz.
- Eine geladene Zwischenstand-Datei wird vor dem Übernehmen geprüft (`saveCheckData`: Startregeln samt Klasse, genau sechs
  Stationen, Fortschritt passend zum Bildschirm, Bestätigungscode neu berechnet). Die Prüfsummen von Zwischenstand und
  Datensatz stehen im öffentlichen Code – sie schützen vor Beschädigung, beweisen aber nicht, dass der Raum gespielt wurde.
- Bekannte Grenze (ohne Speicherung im Browser nicht zu verhindern): Wer vor einem Rätselraum speichert, den Raum mit Fehlern
  versucht und dann die Datei lädt, startet den Raum neu – die Fehler dieses Versuchs fehlen in PDF und Datensatz; sichtbar
  bleibt nur der Vermerk „Weitergespielt ab …“.
- Personendaten und Eingaben nur per `textContent` oder `escapeHtml` ins DOM. Kein `alert`, Meldungen stehen im Dokument.
  Namen und Klasse nur mit Zeichen, die die Überblick-PDF (WinAnsi) darstellen kann (`pdfPrintable`).
- Kein Server, keine Cookies, kein localStorage, kein `fetch`. YouTube erst nach Klick. Überblick-PDF und Zwischenstand nur
  als Download (`application/octet-stream`), nie in einem neuen Tab. Nach dem Start fragt `beforeunload` vor dem
  Neuladen/Schließen nach (nicht beim Gang durch den Tunnel, `leaving`). Der Link durch den Tunnel bleibt im
  Browser-Verlauf – das sagen `#gateHint` und `impressum.html`.
- „Copyright Jan-Rickmer Feindt“ auf jedem Bildschirm (Footer) und auf jeder PDF-Seite (Fußzeile).
- Automatische Tests (Playwright) liegen nicht im Repository. Live für Schüler:innen ist Branch `main`: vor jedem Push den
  ganzen Durchlauf testen und eine echte Überblick-PDF im Feibeitsblatt hochladen.

## Besonderheiten dieses Raums

- Die Stationen heißen in der Oberfläche „Sektionen“: Lagebild „Iran 2026“ = Sektionen 1–3, Lagebild „Sudan“ = Sektionen 4–6
  (dort liegt das Material im Rätselraum selbst: `CHART_SUDAN`, `MATERIAL_R5`, `MATERIAL_R6`). `ROOM_LABELS` tragen die Labels
  „Sektion 1 · …“ bis „Sektion 6 · …“; `SOLUTION_NOTES` sind `{room: ROOM_LABELS[i], text}`, im Datensatz wird `room` zu
  `notes[].label`.
- Die Rätselräume sind einzeln programmiert (`renderRoom1` … `renderRoom6`, `checkR1` … `checkR6`) und nutzen gemeinsame
  Bausteine (`evaluateBuckets`, `evaluateMC`, `evaluateClozeProse`, `evaluateQRows`, `evaluateAssign`, `evaluateChoiceRows`,
  `evaluateGrid`); gestufte Hinweise in `HINTS`/`SOLUTIONS`, die höchste angesehene Stufe (`state.hintMax`) steht in der
  Überblick-PDF.
- Fehlerarten (`kind`) in `state.mistakes[i].errors`: `falsch_einsortiert`, `falsche_wahl`, `falsche_eingabe`,
  `falsche_zuweisung` – mit `item`, `wrongAnswer`, `correctAnswer` und (außer bei Eingaben) `explanation`. Die Überblick-PDF
  erzeugt `generateMistakesPdf`; ihre erste Zeile je Fehler und `errorToRecord` nutzen beide `errorItemText` (Auszeichnung wie
  `<em>`/`<br>` aus den Rätseltexten entfernt `plainText`; Aussagen aus Sektion 3 tragen schon eigene „…“ und werden nicht
  noch einmal eingefasst). Fehlerblöcke brechen Zeile für Zeile auf die nächste Seite um (`paragraphFlow` mit `ensure`).
- Bibliotheksraum 4 hat ein eigenes Erklärvideo (`LIBRARIES[3].video`, YouTube-ID `YbSQ649VoUs`): Das Video steht zuerst, der
  Knopf „Lieber mit dem Text arbeiten“ (`libShowText`) blendet stattdessen den Bibliothekstext ein, „Zurück zum Video“
  (`libShowVideo`) wieder das Video. Ändert sich dieser Bibliothekstext, das Video neu erzeugen und prüfen.
- Zwischenstand: `saveCollectExtra` speichert neben der Sperre der gelösten Rätselräume (`locked`) die im offenen Rätselraum
  schon eingetragenen Antworten (`inputs`, ohne rot markierte Auswahlen; `saveRestoreInputs` setzt sie nach dem Laden wieder
  ein). `saveResumeTarget` trägt einen gerade richtig geprüften Rätselraum, dessen Enthüllung (`triggerReveal`, 900 ms
  verzögert) noch aussteht, beim Speichern schon als gelöst ein.
