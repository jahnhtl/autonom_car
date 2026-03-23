# Musterlösung – Autonomes Fahrzeug

> Grundlage: `project_setup.md`, Datenblätter L298N, Sharp GP2Y0A21YK0F & GP2Y0A02YK0F, Skizze `Arduino_PinBelegung_Skizze.png`

---

## Aufgabe 0 — Repository erstellen

### Empfohlene Ordnerstruktur

```
autonom_car_01/
├── README.md
├── docs/
│   ├── aufgabe1_sensorik.md
│   ├── aufgabe2_antrieb.md
│   ├── aufgabe3_autonomie.md
│   └── aufgabe4_projektplan.md
├── skizzen/
│   ├── sensor_layout.png
│   ├── verdrahtung.png
│   └── zustandsautomat.png
└── src/
    └── main.ino
```

### README.md (Minimalinhalt)

```markdown
# Autonom Car 01

**Team:** Max Mustermann, Anna Beispiel
**Ziel:** Autonomes Fahrzeug für den Crazy-Car-Parcours
**Hardware:** Arduino Uno, L298N, 4× DC-Motor, 3× Sharp IR, HC-SR04, Servo
```

**Checkliste:**
- [ ] Repository `autonom_car_<nummer>` angelegt (privat)
- [ ] `jahnhtl` als Collaborator eingeladen und Zugriff bestätigt
- [ ] Je Aufgabe eine eigene `.md`-Datei in `/docs/`
- [ ] Ordner `/skizzen/` für Bilder/Zeichnungen vorhanden

---

## Aufgabe 1 — Sensorik-Konzept & Montageplan

### Sensoren – technische Eckdaten (aus Datenblättern)

| Sensor | Modell | Messbereich | Versorgung | Strom | Ausgang | Messzeit |
|--------|--------|------------|-----------|-------|---------|---------|
| IR kurz (×2) | Sharp GP2Y0A21YK0F | 10–80 cm | 4,5–5,5 V | ~30 mA | Analog | 38,3 ms |
| IR lang (×1) | Sharp GP2Y0A02YK0F | 20–150 cm | 4,5–5,5 V | ~33 mA | Analog | 38,3 ms |
| Ultraschall | HC-SR04 | 2–400 cm | 5 V | ~15 mA | Digital (Echo) | ~30 ms |
| Servo | Standard | 0–180° | 5 V | ~500 mA peak | PWM | – |

> **Wichtig (aus Datenblatt Sharp):** 10 µF Bypass-Kondensator zwischen VCC und GND direkt am Sensor platzieren. Linse sauber halten. Empfindlich gegenüber direktem Sonnenlicht und spiegelnden Oberflächen.

---

### Drei Sensor-Konzepte

#### Konzept A – Front-Fokus (alle Sensoren vorne)

```
         IR kurz   IR lang   IR kurz
          (-45°)   (0°)     (+45°)
            ↘       ↓       ↙
         [===  Fahrzeug  ===]
            (Servo und HC-SR04 ungenutzt)
```

| Merkmal | Bewertung |
|---------|-----------|
| Sichtfeld vorne | ✅ Sehr gut (3 IR-Sensoren, ~90° Kegel) |
| Frühwarnung geradeaus | ✅ IR lang bis 150 cm |
| Asymmetrie-Erkennung | ✅ Links/rechts 45° → Kurven und Engstellen erkennbar |
| Seitliche Abdeckung | ❌ Blindbereich ab ~50° seitlich |
| Heckabdeckung | ❌ Keine |
| Einfachheit | ✅ Alle Sensoren fix montiert, keine beweglichen Teile |
| Fehlerfälle | Schwarze/glänzende Wände im Blindbereich werden nicht erkannt |

---

#### Konzept B – Seitenregelung mit Servo-Scan *(bevorzugtes Konzept)*

```
             [HC-SR04 auf Servo]
             ↙    ↓    ↘  (schwenkbar 0–180°)
         [===  Fahrzeug  ===]
        ↑                    ↑
   IR kurz               IR kurz
  (90° links)           (90° rechts)
        IR lang (0°, fest, vorne)
```

| Merkmal | Bewertung |
|---------|-----------|
| Sichtfeld vorne | ✅ Gut (IR lang 20–150 cm + Servo-Scan) |
| Seitliche Abdeckung | ✅ Sehr gut (beide IR kurz als Wandfolger) |
| Heckabdeckung | ❌ Keine |
| Wandfolge/Spurhaltung | ✅ Direkt messbar (links/rechts Abstand) |
| Fehlerfälle | Servo-Scan hat Totzeiten (~100 ms pro Sweep) |

---

#### Konzept C – Mittenregelung (45°-Wächter)

```
       IR kurz   IR lang   IR kurz
        (-45°)    (0°)     (+45°)
          ↘        ↓        ↙
         [===  Fahrzeug  ===]
             Servo + HC-SR04 vorne
             (schwenkbar für Ausweichentscheidung)
```

| Merkmal | Bewertung |
|---------|-----------|
| Sichtfeld vorne | ✅ Sehr gut |
| Seitliche Abdeckung | ⚠️ Nur 45°-Kegel (Blindzone ab ca. 60°) |
| Hindernisumfahrung | ✅ Servo zeigt, wohin ausgewichen werden soll |
| Komplexität | ⚠️ Servo-Timing muss mit Fahrtlogik synchronisiert werden |

---

### Gewähltes Layout: **Konzept A – Front-Fokus**

**Begründung:**
1. Alle 3 IR-Sensoren fest montiert – keine beweglichen Teile, robuster Aufbau.
2. Die ±45°-Sensoren (10–80 cm) ermöglichen Asymmetrie-Regelung für Kurven und Engstellen.
3. Der IR-Frontsensor (20–150 cm) liefert frühe Vorwarnung für Hindernisse geradeaus.
4. Einfachster Regelkreis: kein Servo-Timing, kein HC-SR04 nötig.

### Montageskizze (textuelle Darstellung)

```
                   Vorne
         ┌──────────────────────┐
         │ IR kurz  IR lang  IR kurz │
         │ (-45°)    (0°)   (+45°)  │
         │   ↘        ↓       ↙    │
         │                          │
         │   Arduino + Motorshield  │
         │                          │
         └──────────────────────────┘
                   Hinten
```

| Sensor | Position | Winkel | Sichtbereich |
|--------|---------|--------|-------------|
| Sharp GP2Y0A21YK0F (#1) | Front links | −45° | 10–80 cm schräg vorne links |
| Sharp GP2Y0A21YK0F (#2) | Front rechts | +45° | 10–80 cm schräg vorne rechts |
| Sharp GP2Y0A02YK0F | Front-Mitte | 0° (geradeaus) | 20–150 cm |

### Fehlerfälle

| Fehlerfall | Ursache | Gegenmaßnahme |
|-----------|---------|--------------|
| Kein Messwert (0 V) | Sensor nicht angeschlossen oder defekt | Plausibilitätsprüfung: Wert < 0,2 V → Fehler-Flag setzen |
| Falscher Wert bei < 10 cm | GP2Y0A21YK0F saturiert unter 10 cm | Minimalabstand auf 12 cm begrenzen (Clipping) |
| Glänzende/schwarze Oberfläche | IR wird absorbiert oder nicht reflektiert | Letzten validen Wert halten (Plausibilitätsprüfung), Sensor-Watchdog auslösen |
| Direktes Sonnenlicht | Sättigung des Fotodetektors | Optischen Filter 850 ± 70 nm (empfohlen lt. Datenblatt) |
| Spiegelnde Wand | Falsche Winkelreflexion | Messwert mit Moving Average filtern |

### Arbeitspakete Aufgabe 1

- [ ] Montage der 3 IR-Sensoren an Halterungen (3D-gedruckte IR-Halterung vorhanden)
- [ ] Infrarot-Sensoren kalibrieren (ADC-Wert → cm, Kehrwert-Methode, siehe IR-Sensoren-Linearisierung.md)
- [ ] Moving-Average-Filter für IR-Rohdaten (Fenstergröße 3–5)
- [ ] Fehlerbehandlung: Minimalwert-Clipping und Sensor-Watchdog
- [ ] Empirische Validierung auf weißer und grauer Parcours-Oberfläche

---

## Aufgabe 2 — Antrieb & Stromversorgung

### Komponenten & Spannungen

| Komponente | Versorgung | Quelle |
|-----------|-----------|--------|
| DC-Motoren (×4) | ~7,4 V (Akku direkt) | Batterie → L298N OUT |
| L298N Motortreiber | 7,4 V (Eingang VCC_Motor) | Batterie |
| Arduino Uno | 5 V | L298N interner Regler (Jumper gesetzt, VCC < 12 V ✅) |
| Sharp IR-Sensoren (×3) | 5 V | Arduino 5V-Pin |

**Akkuberechnung (2× 18650 in Reihe):**
- Nennspannung: 2 × 3,7 V = **7,4 V**
- Vollgeladen: 2 × 4,2 V = **8,4 V**
- Tiefentladegrenze: 2 × 3,0 V = **6,0 V** → nicht unterschreiten!
- Effektive Motorspannung: 7,4 V − 2 V (L298N-Spannungsabfall) ≈ **5,4 V**

### Motorverdrahtung: 4 Motoren mit 2 H-Brücken

Da der L298N nur 2 H-Brücken hat, werden je 2 Motoren einer Seite **parallel** geschaltet:

```
H-Brücke 1:  Motor VL (vorne links) ──┐
              Motor HL (hinten links) ─┴─→ OUT1 / OUT2

H-Brücke 2:  Motor VR (vorne rechts) ─┐
              Motor HR (hinten rechts)─┴─→ OUT3 / OUT4
```

**Strombetrachtung:**
- L298N max. 2 A pro Kanal
- 2 Motoren parallel → max. 1 A je Motor → unkritisch bei Normalbetrieb

### Block-/Verdrahtungsdiagramm

```
       ┌──────────────────────────────────────────┐
       │              2× 18650 Akkupack           │
       │                  (+)              (-)    │
       └───────────────────┬────────────────┬─────┘
                           │                │
                           │                │
 ┌──────────────────────┐  │                │
 │   L298N              │  ▼                │
 │  VCC_Motor (12V)     │← 7,4V Akku+       │ GND (gemeinsam) 
 │  GND                 │← Akku- ───────────│
 │  5V OUT              │→ 5V ────────┐     │ 
 │                      │             ▼     ▼
 │  ENA  ←──── D9 (PWM) │→       ┌───────────────────────┐
 │  ENB  ←──── D10 (PWM)│→       │   +5V    GND          │
 │  IN1  ←──── D4       │→       │                       │
 │  IN2  ←──── D7       │→       │    Arduino Uno        │
 │  IN3  ←──── D8       │→       │                       │
 │  IN4  ←──── D12      │→       │ A0 ← IR links (-45°)  │
 │                      │        │ A1 ← IR rechts (+45°) │
 │  OUT1 ──→ Motor L +  │        │ A2 ← IR front (0°)    │
 │  OUT2 ──→ Motor L -  │        │                       │
 │  OUT3 ──→ Motor R +  │        │ D2 ← Taster START     │
 │  OUT4 ──→ Motor R -  │        │ D3 ← Taster STOP      │
 └──────────────────────┘        └───────────────────────┘

  Taster-Beschaltung (INPUT_PULLUP):
  D2 ──[START]── GND    (LOW = gedrückt)
  D3 ──[STOP ]── GND    (LOW = gedrückt)
```

### Pin-Mapping-Tabelle

| Arduino-Pin | Funktion | Verbunden mit | Typ |
|------------|---------|--------------|-----|
| D2 | Taster START (INT0) | Taster → GND, INPUT_PULLUP | Digital IN |
| D3 | Taster STOP (INT1) | Taster → GND, INPUT_PULLUP | Digital IN |
| D4 | Richtung links vorne | L298N IN1 | Digital OUT |
| D7 | Richtung links zurück | L298N IN2 | Digital OUT |
| D8 | Richtung rechts vorne | L298N IN3 | Digital OUT |
| D12 | Richtung rechts zurück | L298N IN4 | Digital OUT |
| D9 | Geschwindigkeit links | L298N ENA | PWM OUT |
| D10 | Geschwindigkeit rechts | L298N ENB | PWM OUT |
| A0 | IR Sensor links (−45°) | Sharp GP2Y0A21YK0F | Analog IN |
| A1 | IR Sensor rechts (+45°) | Sharp GP2Y0A21YK0F | Analog IN |
| A2 | IR Sensor front (0°) | Sharp GP2Y0A02YK0F | Analog IN |
| 5V | Versorgung Sensoren | Alle 5V-Verbraucher | Power |
| GND | Gemeinsame Masse | Alle GND | Power |

> Gemäß Skizze `Arduino_PinBelegung_Skizze.png`: Der Akku versorgt das Motorshield (12V-Eingang), dieses stellt 5V für das Arduino Sensorshield bereit.

### Risikoliste mit Gegenmaßnahmen

| Risiko | Wahrscheinlichkeit | Gegenmaßnahme |
|-------|-------------------|--------------|
| Spannungseinbruch bei Motoranlauf | Mittel | 100 µF Kondensator an VCC_Motor des L298N |
| Masseschleifen (Noise) | Mittel | Sternförmige Masseführung, 100 nF Keramik-Kondensatoren an IR-Sensoren |
| Tiefentladung Akkus | Mittel | Software-Watchdog: Bei VCC < 6,2 V → STOPP-Zustand auslösen |
| L298N Überhitzung | Gering | Thermik-Check nach Testfahrten; nicht dauerhaft > 75 °C |
| Motoren drehen rückwärts | Gering | Polarität prüfen beim Montieren; Software-Korrektur IN1/IN2 tauschen |
| EMV-Störungen durch Motoren | Mittel | Freilaufdioden (L298N intern vorhanden ✅); Keramik-C direkt an Motorklemmen |
| Servo / HC-SR04 nicht verbunden | – | Gewähltes Konzept A nutzt diese Komponenten nicht; Pins frei für spätere Erweiterung |

### Zusätzliche Bauteile

| Bauteil | Wert | Zweck | Platzierung |
|--------|------|-------|------------|
| Elektrolytkondensator | 100 µF / 16 V | Spannungsstabilisierung Motor-VCC | L298N VCC-Pin ↔ GND |
| Keramikkondensator | 100 nF | HF-Entstörung Sensoren | Je IR-Sensor VCC ↔ GND |
| Keramikkondensator | 10 µF (Datenblatt Sharp!) | DC-Bypass Sensor | Je IR-Sensor VCC ↔ GND |

### Arbeitspakete Aufgabe 2

- [ ] Montage der 4 Motoren am Chassis
- [ ] Verdrahtung L298N → Arduino (IN1–IN4, ENA/ENB)
- [ ] Simpler Funktionstest: geradeaus fahren (beide Seiten gleiche PWM)
- [ ] Pinout & Kabelbaum dokumentieren und testen
- [ ] Konzept zur Tiefentladungsvermeidung: Spannungsteiler an A3 + Schwellwert-Abfrage
- [ ] Thermik-Check: L298N nach 5 min Dauerbetrieb mit Handmessung prüfen (optional)
- [ ] EMV-Entstörung: Kondensatoren einlöten, Leitungsführung prüfen (optional)

---

## Aufgabe 3 — Autonomiestrategie & Zustandsautomat

### Primäre Navigationsstrategie: **Mittenregelung (Asymmetrie-Regelung)**

Das Fahrzeug nutzt die Asymmetrie der beiden seitlich-vorwärts gerichteten IR-Sensoren (±45°), um sich im Korridor zu zentrieren. Der IR-Frontsensor (0°) erkennt Hindernisse geradeaus frühzeitig.

```
dL = Abstand IR links  (−45°, Sharp GP2Y0A21, 10–80 cm)
dR = Abstand IR rechts (+45°, Sharp GP2Y0A21, 10–80 cm)

Fehler e = dL − dR
  e > 0  → mehr Platz links  → leicht nach links lenken
  e < 0  → mehr Platz rechts → leicht nach rechts lenken
  e ≈ 0  → zentriert → Geradeausfahrt

Korrektur (P-Regler):  speed_diff = Kp × e  (Kp ≈ 2)
  links_PWM  = base_speed − speed_diff
  rechts_PWM = base_speed + speed_diff

IR-Front (0°, Sharp GP2Y0A02, 20–150 cm):
  → Vorwarnung und Hindernis-Stop geradeaus
```

### Zustandsautomat

```
                    ┌─────────────┐
                    │    IDLE     │  Motoren aus
                    └──────┬──────┘
                           │ START-Taster (D2)
                           ▼
              ┌────────────────────────┐
         ┌───►│       VORWÄRTS         │◄──────────────┐
         │    │  Mittenregelung aktiv  │               │ F > 80 cm
         │    └──────┬──────────┬──────┘               │
         │     dL>dR │          │ dR>dL                │
         │     F<60  │          │  F<60                │
         │           ▼          ▼                      │
         │  ┌──────────────┐ ┌──────────────┐          │
         └──│  KURVE LINKS │ │ KURVE RECHTS │──────────┘
    F>80cm  └──────▲───────┘ └───────▲──────┘  F>80cm
                   │ dL>dR           │ dR>dL
                   │  nach Timer     │
                   └────────┬────────┘
                            │
                   ┌────────┴────────┐
  VORWÄRTS F<20 ──►│    RÜCKWÄRTS    │──► F>40 VORWÄRTS
                   └─────────────────┘

    ─── jederzeit, von jedem Zustand ───────────────────────►
        STOP-Taster (D3) / Sensor-Fehler / VCC < 6,2 V
                   ┌─────────────┐
                   │     IDLE    │  alle Motoren aus
                   └──────┬──────┘
                          │ START-Taster (D2) → IDLE
```

### Zustandsübergänge & Bedingungen

| Von | Nach | Bedingung |
|-----|------|-----------|
| IDLE | VORWÄRTS | START-Taster (D2) gedrückt ODER serielles Kommando |
| VORWÄRTS | KURVE LINKS | IR-Front < 60 cm UND dL > dR (mehr Platz links) |
| VORWÄRTS | KURVE RECHTS | IR-Front < 60 cm UND dR > dL (mehr Platz rechts) |
| VORWÄRTS | RÜCKWÄRTS | IR-Front < 20 cm (kein Ausweg vorne) |
| KURVE LINKS | VORWÄRTS | IR-Front > 80 cm (Weg wieder frei) |
| KURVE RECHTS | VORWÄRTS | IR-Front > 80 cm (Weg wieder frei) |
| RÜCKWÄRTS | KURVE LINKS | Rückfahr-Timer abgelaufen UND dL > dR |
| RÜCKWÄRTS | KURVE RECHTS | Rückfahr-Timer abgelaufen UND dR > dL |
| JEDER | IDLE | STOP-Taster (D3) gedrückt ODER Sensor-Fehler ODER Watchdog-Timeout (500 ms) ODER VCC < 6,2 V ODER (manueller Reset) |

### Schnittstellen / APIs

```cpp
// Taster (INPUT_PULLUP → LOW = gedrückt)
bool isStartPressed();     // true wenn D2 LOW
bool isStopPressed();      // true wenn D3 LOW

// Motorsteuerung
void setMotorSpeed(int left, int right);  // -255..255; negativ = rückwärts

// Sensorabfragen – Konzept A: 3 IR-Sensoren fest verbaut (kein HC-SR04, kein Servo)
float readIR_left();      // Sharp GP2Y0A21YK0F (A0), −45°, 10–80 cm
float readIR_right();     // Sharp GP2Y0A21YK0F (A1), +45°, 10–80 cm
float readIR_front();     // Sharp GP2Y0A02YK0F  (A2),   0°, 20–150 cm

// Mittenregelung
float getAsymmetry();     // dL − dR; positiv = mehr Platz links

// Zustandsmaschine
enum State { IDLE, VORWAERTS, KURVE_LINKS, KURVE_RECHTS, RUECKWAERTS };
void updateStateMachine();               // einmal pro Schleife aufrufen
State getCurrentState();
```

### Aktualisierungsraten

| Komponente | Rate | Begründung |
|-----------|------|-----------|
| Sharp IR-Sensoren (×3) | 25 Hz (40 ms) | Messzeit laut Datenblatt: 38,3 ms → 1 Messung pro Zyklus |
| Motor-PWM | ~490 Hz | Arduino Hardware-PWM (Timer 1 & 2, Standard) |
| Zustandsautomat | 25 Hz | Synchron mit Sensor-Leserate |

### Filter

| Signal | Filter | Parameter | Zweck |
|-------|--------|-----------|-------|
| IR links (−45°) / rechts (+45°) | Moving Average | N = 2 Werte | Rauschen für Asymmetrie-Regelung glätten |
| IR front (0°) | Moving Average | N = 3 Werte | Langsamerer Einfluss = sanfteres Abbremsen |
| Asymmetrie e = dL − dR | Clipping / Begrenzen | \|e\| > 60 cm → auf ±60 cm begrenzen | Ausreißer bei Kurveneinfahrt unterdrücken |
| Alle IR-Sensoren | Clipping / Begrenzen | Min 10/20 cm, Max = 80/150 cm | Ungültige Werte unterhalb Messbereich ausschließen |

### Abstandsschwellen

| Schwelle | Wert | Aktion |
|---------|------|--------|
| Kurveneinleitung | F < 60 cm | Zustand → KURVE LINKS oder KURVE RECHTS (je nach dL vs. dR) |
| RÜCKWÄRTS auslösen | F < 20 cm | Zustand → RÜCKWÄRTS, alle Motoren rückwärts |
| Rückfahr-Timer | 800 ms | Wechsel RÜCKWÄRTS → KURVE LINKS/RECHTS |
| Zurück zu VORWÄRTS | F > 80 cm | Zustand → VORWÄRTS (Weg wieder frei) |
| Asymmetrie Totband | \|e\| < 5 cm | Keine Korrektur (zentriert genug) |
| Notabschaltung | Watchdog > 500 ms ODER VCC < 6,2 V | alle Motoren aus, Zustand → IDLE |
| Engstelle erkannt | dL < 15 cm UND dR < 15 cm | PWM reduzieren auf 30%, langsam durchfahren |

### IR-Sensor-Linearisierung (Spannungs-/Distanztabelle)

Die Sharp-Sensoren liefern eine nichtlineare Spannung. Umrechnung über Lookup-Table oder Formel. Siehe IR-Sensoren-Linearisierung.md File.

### Arbeitspakete Aufgabe 3

- [ ] Sensor-Werte kalibrieren und Parameter bestimmen
- [ ] Sensorfusion implementieren (alle Sensorwerte zusammenführen, 25 Hz)
- [ ] Zustandsautomat programmieren
- [ ] Schwellwert-Tuning auf realem Parcours
- [ ] Failsafe-Stopp: Watchdog-Timer + VCC-Überwachung
- [ ] Serielles Logging: alle 100 ms Zustand + Sensorwerte via `Serial.println`
- [ ] Mittenregler parametrieren (P-Regler: `speed_diff = Kp × (dL − dR)`, Kp ≈ 2, Totband ±5 cm)
- [ ] Erster Testlauf: Geradeausfahrt im Korridor

---

## Aufgabe 4 — Projektplan & Tests

### 7-Wochen-Meilensteinplan

| Woche | Meilenstein | Verantwortlich | Arbeitspakete |
|-------|------------|----------------|--------------|
| 1 | **Planung & Konzepte abgeschlossen** | Alle | Aufgaben 1–3 finalisiert, Issues angelegt |
| 2 | **Fahrzeug fährt gerade** | Hardware | Motormontage, Verdrahtung, Geradeaus-Test |
| 3 | **Stabile Sensorwerte** | Software | Linearisierung, Filter, serielles Logging |
| 4 | **Autonome Navigation – erste Kurve** | Software | FSM, Mittenregler (Asymmetrie 45°), Kurven-Test |
| 5 | **Ganze Strecke gemeistert** | Alle | Vollständiger Parcours, Failsafe aktiv |
| 6 | **Strecke unter 50 Sekunden** | Software | Geschwindigkeitstimmung, Schwellwerte |
| 7 | **Strecke unter 30 Sekunden + Finales Rennen** | Alle | Letzte Optimierung, Rennen |

### GitHub-Issues den Meilensteinen zuordnen

Empfohlene Issue-Struktur (alle Arbeitspakete aus Aufgaben 1–3 als einzelne Issues):

```
Meilenstein 1 – Planung abgeschlossen:
  #1  Sensor-Konzept auswählen und dokumentieren
  #2  Verdrahtungsdiagramm erstellen
  #3  Zustandsautomat skizzieren

Meilenstein 2 – Fahrzeug fährt gerade:
  #4  Motoren montieren
  #5  L298N verdrahten und testen
  #6  Geradeausfahrt-Test

Meilenstein 3 – Stabile Sensorwerte:
  #7  IR-Sensoren kalibrieren
  #8  Moving-Average-Filter implementieren
  #9  Serielles Logging einrichten

Meilenstein 4 – Erste Kurve:
  #10 Mittenregler implementieren (Asymmetrie-Regelung über 45°-IR-Sensoren)
  #11 Zustandsautomat programmieren
  #12 Kurven-Test

Meilenstein 5 – Ganze Strecke:
  #13 Hindernis-Ausweichen implementieren
  #14 Failsafe-Stopp
  #15 Vollparcours-Test

Meilenstein 6 – Unter 50 s:
  #16 Geschwindigkeits-Tuning
  #17 Schwellwert-Feinabstimmung

Meilenstein 7 – Finales Rennen:
  #18 Finale Kalibrierung
  #19 Rennen-Dokumentation
```
