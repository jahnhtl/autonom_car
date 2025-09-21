# Projektideen & Aufgaben – Autonomes Fahrzeug

Dieses Arbeitsblatt enthält **fünf strukturierte Aufgaben** zur Ideenfindung und Planung eures Projekts. Nutzt ausschließlich die vorhandenen Komponenten:

- Chassis & Räder & mechanische Kleinteile
- 1× Motortreiber mit 2 H‑Brücken
- 4× Motoren
- 2× IR‑Sensoren (10–80 cm)
- 1× IR‑Sensor (20–150 cm)
- 1× Arduino‑Board
- 1× Batteriehalter für 2× 18650 mit Schalter
- 1× Ultraschallsensor
- 1× Servo

## Aufgabe 0 - Repository erstellen
**Ziel:** Ein eigenes GitHub-Repository für das Projekt anlegen, das den Namen des Autos inklusive Autonummer enthält und alle Aufgaben klar strukturiert als Markdown-Dateien dokumentiert.

**Vorgehen (15 min):**
- Legt ein neues privates Repository auf GitHub an. Der Name muss das Fahrzeug und die Autonummer enthalten (z. B. `autonomcar_auto3`).
- Erstellt für jede Aufgabe eine eigene Markdown-Datei (`aufgabe1_sensorik.md`, `aufgabe2_antrieb.md`, etc.).
- Legt eine sinnvolle Ordnerstruktur an (z. B. `/docs/` für Dokumentation, `/skizzen/` für Bilder/Skizzen).
- Erstellt eine kurze `README.md` mit Projektname, Teammitgliedern und kurzer Beschreibung.
- Prüft, dass jahnhtl Zugriff auf das Repository bekommt und die Einladung angenommen hat.
- Dokumentiert die Aufgabenstellungen und Ergebnisse ausschließlich in den jeweiligen `.md`-Dateien.

---

## Aufgabe 1 — Sensorik‑Konzept & Montageplan
**Ziel:** Festlegen, wie der Parcours und Hindernisse mit den vorhandenen Sensoren erkannt werden.

**Gegebene Komponenten:** 2× IR (10–80 cm), 1× IR (20–150 cm), 1× Ultraschall, 1× Servo, 1x Arduino Uno, Chassis.

**Vorgehen (60 min):**
- Schaut euch die Charakteristiken der verfügbaren Sensoren in den Datenblättern an.  
- Entwickelt **drei Konzepte** welche und wo die gegebenen Komponenten positioniert werden sollten und überlegt Vor- und Nachteile
- Notiert je Layout: Sichtfeld, Blindbereiche, Distanzbereiche und mögliche **Fehlerfälle** (glänzende/schwarze Oberflächen, Winkel).
- Wählt **ein bevorzugtes Layout** und begründet die Wahl.

**Abgaben (1 Seite + Skizze):**
- Vergleichstabelle (Vor-/Nachteile).  
- Skizze mit **Montagepositionen**, ungefähren Winkeln und abgedeckte Sichtbereichen.  
- Liste der **Arbeitspakete** (z. B. "Montage der Sensoren", "Infrarot-Sensoren‑Kalibrierung", "Servo‑Scan‑Treiber", "Filterung", "Fehlerbehandlung und -korrektur").

---

## Aufgabe 2 — Antrieb & Stromversorgung (4 Motoren, 2 H‑Brücken)
**Ziel:** Ein sicheres, realistisches Verdrahtungs‑ und Versorgungskonzept vorschlagen.

**Gegebene Komponenten:** 4 DC‑Motoren, 1 Motortreiber (2 H‑Brücken), Arduino, Batteriehalter (2×18650 Lithium-Ionen-Akkus) mit Schalter.

**Vorgehen (60 min):**
- Schaut euch das Datenblatt des Motortreibers sowie die Akkugenau an.  
- Entscheidet, wie **4 Motoren mit 2 H‑Brücken** angesteuert werden
- Plant den **Haupt‑Leistungspfad** wie die Akkus, die Motoren, der Arduino versorgt werden.
- Legt ein **Pin‑Mapping** fest (PWM/Dir/Enable) und gemeinsame Masseführung
- Notiert falls zusätzliche **Kondensatoren**/**Dioden** benötigt werden und wie verhindert wird, dass die Akkus tiefentladen werden

**Abgaben (1 Seite und Skizze):**
- **Block-/Verdrahtungsdiagramm** (Akku, Schalter, Motortreiber, Motoren, Arduino).  
- Pin‑Mapping‑Tabelle + **Risikoliste** (z. B. Spannungseinbruch, Masseschleifen) mit Gegenmaßnahmen.  
- List der Arbeitspakete (z. B. "Montage der Motoren", "Simpler Funktionstest - geradeaus fahren", "Thermik‑Check", "Pinout & Kabelbaum testen", "EMV‑Entstörung", "Konzept zur Tiefentladungvermeidung").

---

## Aufgabe 3 — Autonomiestrategie & Zustandsautomat
**Ziel:** Die Fahrlogik entwerfen, die auf dem Arduino läuft.

**Gegebene Grundlage:** Euer in Aufgabe 1 gewähltes Sensor‑Konzept.

**Vorgehen (60 min):**
- Wählt eine **primäre Navigationsstrategie** (z. B. Seitenregelung, Mittenregelung)
- Skizziert einen **Zustandsautomaten** (z.B. Idle → Fahren → Annäherung → Ausweichen → Rückkehr → Ziel).  
- Definiert **Schnittstellen/APIs**, die ihr implementiert um die Motoren anzusteuern, die Sensoren auszulesen, die Strategie umzusetzen.  
- Legt **Aktualisierungsraten** fest (z. B. Sensoren __ Hz, Motor‑PWM ___ Hz - also wie oft Sensoren eingelesen bzw. Motoren-PWM angepasst wird)  
- Welche **Filter** wollt ihr einsetzen (z.B. Moving Average, Debounce).
- Definiert erste **Abstandsschwellen** (z. B. Abbremsen bei __ cm, Stopp bei __ cm, Abbiegen bei __ cm).


**Abgaben (1 Seite und Skizze):**
- Zustandsdiagramm (Foto oder sauber gezeichnet).  
- Kurzer Pseudocode der Main‑Loop mit Übergängen.  
- Arbeitspakete (z. B. "Sensorfusion", "Schwellwert‑Tuning", "Failsafe‑Stop", "Serielles Logging", "Simulation", "Erster Test").

---

## Aufgabe 4 — Projektplan & Tests
**Ziel:** Projekt plan erstellen und die Arbeit strukturieren.

**Vorgehen (60 min):**
- Entwerft einen **5–7‑Wochen‑Plan** mit Meilensteinen (z.B. Chassis & Power, Basisfahrt, kalibrierte Sensorik, erste Autonomie, Tuning, Final‑Demo) und weist Verantwortliche zu.  
- Überlegt mögliche Tests für den Parcours (Gerade, 90°‑Kurve, enge Passage, Hindernis auf Spur) mit erwarteten Ergebnissen/Metriken: Zielzeit, # Kontakte, Mindestabstand.

**Abgaben (vollständiger Plan):**
- Liste der **Meilensteine mit internen Deadlines** im Git-Repository angelegt.
- Arbeitspakete (Issues) von Aufgaben 1-3 den Meilensteinen zugeordnet. (Anleitung siehe unten)

---

## Anleitung: Issues und Meilensteine auf GitHub anlegen und verknüpfen

### 1. Issue anlegen

1. Öffne dein Repository auf [github.com](https://github.com).
2. Klicke oben auf den Reiter **"Issues"**.
3. Klicke auf **"New issue"**.
4. Gib einen **Titel** und eine **Beschreibung** für die Aufgabe ein.
5. Optional: Weise das Issue einem Teammitglied zu, füge Labels hinzu oder verlinke Pull Requests.
6. Klicke auf **"Submit new issue"**.

### 2. Meilenstein anlegen

1. Im Reiter **"Issues"** klicke rechts auf **"Milestones"**.
2. Klicke auf **"New milestone"**.
3. Gib einen **Titel**, eine **Beschreibung** und ein **Fälligkeitsdatum** ein.
4. Klicke auf **"Create milestone"**.

### 3. Issue einem Meilenstein zuordnen

1. Öffne das gewünschte Issue.
2. Rechts findest du das Feld **"Milestone"**.
3. Wähle den passenden Meilenstein aus der Liste aus.


**Tipp:** Nutze die Meilenstein-Ansicht, um den Fortschritt aller zugehörigen Aufgaben im Blick zu behalten.

---
