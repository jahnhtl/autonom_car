# Projektideen & Aufgaben – Autonomes Fahrzeug

Dieses Arbeitsblatt enthält **vier strukturierte Aufgaben** zur Ideenfindung und Planung eures Projekts. Nutzt ausschließlich die vorhandenen Komponenten:

- Chassis & Räder & mechanische Kleinteile
- 1× Motortreiber mit 2 H‑Brücken
- 4× Motoren
- 2× IR‑Sensoren (10–80 cm)
- 1× IR‑Sensor (20–150 cm)
- 1× Arduino‑Board
- 1× Batteriehalter für 2× 18650 mit Schalter
- 1× Ultraschallsensor
- 1× Servo

---

## Aufgabe 1 — Sensorik‑Konzept & Montageplan
**Ziel:** Festlegen, wie der Parcours und Hindernisse mit den vorhandenen Sensoren erkannt werden.

**Gegebene Komponenten:** 2× IR (10–80 cm), 1× IR (20–150 cm), 1× Ultraschall, 1× Servo, Chassis.

**Vorgehen (30 min):**
- Entwickelt **drei Steuerkonzepte** anhand der gegebenen Komponenten und überlegt Vor- und Nachteile
- Notiert je Layout: Sichtfeld, Blindbereiche, Distanzbereiche und mögliche **Fehlerfälle** (glänzende/schwarze Oberflächen, Winkel).
- Wählt **ein bevorzugtes Layout** und begründet die Wahl.
- Definiert erste **Abstandsschwellen** (z. B. Abbremsen bei __ cm, Stopp bei __ cm) und ein **Servo‑Scanmuster** (Winkel, Rate), falls genutzt.

**Abgaben (1 Seite + Skizze):**
- Vergleichstabelle (Vor-/Nachteile, Risiken).  
- Skizze mit **Montagepositionen**, ungefähren Winkeln und Kabelwegen.  
- Liste der **Arbeitspakete** (z. B. „IR‑Kalibrierung“, „Servo‑Scan‑Treiber“, „Filterung“).

---

## Aufgabe 2 — Antrieb & Stromversorgung (4 Motoren, 2 H‑Brücken)
**Ziel:** Ein sicheres, realistisches Verdrahtungs‑ und Versorgungskonzept vorschlagen.

**Gegebene Komponenten:** 4 DC‑Motoren, 1 Motortreiber (2 H‑Brücken), Arduino, Batteriehalter (2×18650) mit Schalter.

**Vorgehen (30 min):**
- Entscheidet, wie **4 Motoren mit 2 H‑Brücken** angesteuert werden
- Legt **Pin‑Mapping** fest (PWM/Dir/Enable) und gemeinsame Masseführung; notiert evtl. nötige **Kondensatoren**/**Dioden**
- Plant den **Haupt‑Leistungspfad**

**Abgaben (1 Seite):**
- **Block-/Verdrahtungsdiagramm** (Akku, Schalter, Sicherung, Motortreiber, Motoren, Arduino, Servo).  
- Pin‑Mapping‑Tabelle + **Risikoliste** (z. B. Spannungseinbruch, Masseschleifen) mit Gegenmaßnahmen.  
- Arbeitspakete (z. B. „Stromtest“, „Thermik‑Check“, „Pinout & Kabelbaum“, „EMV‑Entstörung“).

---

## Aufgabe 3 — Autonomiestrategie & Zustandsautomat
**Ziel:** Die Fahrlogik entwerfen, die auf dem Arduino läuft.

**Gegebene Grundlage:** Euer in Aufgabe 1 gewähltes Sensor‑Konzept.

**Vorgehen (30 min):**
- Wählt eine **primäre Navigationsstrategie**
- Skizziert einen **Zustandsautomaten** (z.B. Idle → Fahren → Annäherung → Ausweichen → Rückkehr → Ziel).  
- Definiert **Schnittstellen/APIs**, die ihr implementiert um die Motoren anzusteuern, die Sensoren auszulesen, die Strategie umzusetzen.  
- Legt **Aktualisierungsraten** fest (z. B. Sensoren 20 Hz, Planner 20 Hz, Motor‑PWM ___ Hz) und Grund‑Filter (Moving Average, Debounce).
- Formuliert **Abnahmekriterien** (z. B. Parcours ohne Kontakt ≥ 3/5 Läufe; Mindestabstand ≥ __ cm).

**Abgaben (1 Seite):**
- Zustandsdiagramm (Foto oder sauber gezeichnet).  
- Kurzer Pseudocode der Main‑Loop mit Übergängen.  
- Arbeitspakete (z. B. „Sensorfusion“, „Schwellwert‑Tuning“, „Failsafe‑Stop“, „Serielles Logging“).

---

## Aufgabe 4 — Projektplan & Tests
**Ziel:** Nachweis der Funktion planen und die Arbeit strukturieren.

**Vorgehen (20 min):**
- Entwerft einen **5–7‑Wochen‑Plan** mit Meilensteinen (Chassis & Power, Basisfahrt, kalibrierte Sensorik, erste Autonomie, Tuning, Final‑Demo) und weist Verantwortliche zu.  
- Erstellt mögliche Tests für den Parcours (Gerade, 90°‑Kurve, enge Passage, Hindernis auf Spur) mit erwarteten Ergebnissen/Metriken: Zielzeit, # Kontakte, Mindestabstand.

**Abgaben (1–2 Seiten):**
- Gantt‑artige Liste der **Meilensteine mit internen Deadlines** und Teamrollen.  
- Arbeitspakete den Meilensteinen zugeordnet.

---

## Abgabeformat
Jede Gruppe reicht die Aufgabenstellung in einem eigens erstellten Repository ein mit den vier oben genannten Abgaben ein. Für jede Aufgabe soll eine eigene .md Datei erstellt werden. Haltet es präzise und konkret (Skizzen ausdrücklich erwünscht). Anhand dessen werden Scope, Teamzuschnitt und der erste Planung freigegeben.
