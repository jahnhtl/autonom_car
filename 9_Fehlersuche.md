# Troubleshooting – Autonomes Fahrzeug

> Schnellreferenz für häufige Probleme bei Hardware, Software, Sensoren und Stromversorgung.

---

## 1. Arduino wird nicht erkannt

**Symptom:** COM-Port fehlt in der IDE, Geräte-Manager zeigt Warnsymbol.

### Ursache & Lösung

**Schritt 1 – Geräte-Manager prüfen**
- Öffne den **Geräte-Manager** (Windows)
- Suche unter „USB-Controller" oder „Anschlüsse (COM & LPT)" nach unbekannten Geräten mit Warnsymbol
- Rechtsklick → Eigenschaften → Tab **Details** → Eigenschaft **Hardware-IDs** → Chipbezeichnung ablesen

**Schritt 2 – Passenden Treiber installieren**

| Chip | Typ | Treiber |
|------|-----|---------|
| CH340 / CH341 | Arduino-Nachbau (häufig) | `drivers/Treiber_CH341SER.zip` (im Repo) |
| FTDI FT232 | Arduino Original | In Arduino IDE enthalten oder von ftdichip.com |
| ATmega16U2 | Arduino Uno original | In Arduino IDE enthalten |

**Schritt 3 – Treiber manuell zuweisen**
1. Rechtsklick auf das unbekannte Gerät → **Treiber aktualisieren**
2. **Auf meinem Computer nach Treibern suchen**
3. Ordner mit entpacktem Treiber wählen (die `.inf`-Datei muss darin liegen)
4. Weiter → IDE neu starten → Board erneut verbinden

**Weitere Ursachen:**
- Nur Lade-Kabel statt Datenkabel verwendet → Datenkabel verwenden
- USB-Hub ohne eigene Stromversorgung → direkt am PC einstecken
- Arduino IDE Board-Auswahl falsch → Werkzeuge → Board → **Arduino Uno**

---

## 2. Upload schlägt fehl

**Symptom:** `avrdude: stk500_recv(): programmer is not responding`

| Ursache | Lösung |
|---------|--------|
| Falscher COM-Port | Werkzeuge → Port → richtigen COMx wählen |
| Serieller Monitor offen | Monitor schließen, dann erneut hochladen |
| Bootloader beschädigt | Board mit ISP-Programmer neu flashen |
| Sketch läuft in Endlosschleife ohne Delay | Reset-Taste drücken und sofort Upload starten |
| `delay()` blockiert Bootloader-Fenster | `Serial.begin()` am Anfang von `setup()` hinzufügen |

---

## 3. Stromversorgung & Akku

**Symptom:** Arduino startet kurz und resettet sich, Motoren laufen unregelmäßig.

| Problem | Ursache | Lösung |
|---------|---------|--------|
| Arduino resettet bei Motorstart | Spannungseinbruch durch Motorstrom | 100 µF Elko an VCC_Motor des L298N nachrüsten |
| Alle Systeme aus, obwohl Schalter an | Akku tiefentladen (< 6,0 V) | Akku laden, Tiefentladeschutz in Software prüfen |
| L298N wird sehr heiß | Dauerbetrieb > 2 A je Kanal | Testlauf verkürzen; Kühlkörper aufkleben |
| 5V-Versorgung fehlt | L298N-Jumper (5V EN) nicht gesetzt | Jumper setzen, wenn VCC_Motor < 12 V |
| Spannung bricht beim Einschalten ein | Kalter Akku oder alte Zellen | Akku aufwärmen, Kapazität prüfen |

**Spannungen prüfen (Multimeter):**
```
Akku voll:    ~8,4 V   (2× 4,2 V)
Akku leer:     6,0 V   (2× 3,0 V) → sofort laden!
L298N 5V OUT:  4,8–5,2 V  → Arduino und Sensoren versorgt
Arduino 5V:    4,8–5,2 V  → am 5V-Pin messen
```

---

## 4. Motoren

**Symptom:** Motoren reagieren nicht, drehen falsch oder ruckeln.

| Problem | Ursache | Lösung |
|---------|---------|--------|
| Motor dreht sich nicht | ENA/ENB nicht auf HIGH oder PWM = 0 | `analogWrite(ENA, 150)` testen |
| Beide Seiten drehen rückwärts | IN1/IN2 vertauscht | IN1 und IN2 im Code oder Verkabelung tauschen |
| Eine Seite dreht nicht | L298N-Kanal defekt oder Kabel offen | Kanal mit Multimeter prüfen; Kabelverbindung testen |
| Motoren ruckeln | PWM-Frequenz zu niedrig oder EMV-Störung | 100 nF Keramikkondensatoren direkt an Motorklemmen |
| Fahrzeug dreht trotz gleicher PWM | Motoren unterschiedlich, Reibung ungleich | `base_speed` asymmetrisch anpassen oder mechanisch ausgleichen |

**Schnelltest im Serial Monitor:**
```cpp
setMotorSpeed(150, 150);   // geradeaus
delay(1000);
setMotorSpeed(0, 0);       // stopp
```

---

## 5. IR-Sensoren

**Symptom:** Falsche Distanzwerte, sprunghafte Messung, Sensor liefert 0.

| Problem | Ursache | Lösung |
|---------|---------|--------|
| ADC-Wert = 0 oder konstant 1023 | Kabel offen oder Kurzschluss | VCC/GND/Signal mit Multimeter prüfen |
| Werte springen stark | Kein Bypass-Kondensator | 10 µF direkt an Sensor VCC ↔ GND |
| Distanz < 10 cm liefert unsinnige Werte | GP2Y0A21 saturiert | Clipping auf Minimalwert 12 cm im Code |
| Werte weichen > 5 cm ab | Eigener Sensor braucht Kalibrierung | Neue Messpunkte aufnehmen, Regression wiederholen (Kehrwert-Methode) |
| Schwarze Wand nicht erkannt | IR wird absorbiert | Letzten validen Wert halten; Watchdog-Flag setzen |
| Messung im Sonnenlicht instabil | Fotoempfänger gesättigt | Sensor abschirmen oder Innenraum-Test |

**Sensorwerte im Serial Monitor ausgeben:**
```cpp
Serial.print("L: "); Serial.print(distanzLinks());
Serial.print("  R: "); Serial.print(distanzRechts());
Serial.print("  F: "); Serial.println(distanzFront());
```

---

## 6. Taster (START / STOP)

**Symptom:** Taster reagiert nicht oder löst dauerhaft aus.

| Problem | Ursache | Lösung |
|---------|---------|--------|
| Kein Reaktion auf Tastendruck | `INPUT_PULLUP` vergessen | `pinMode(PIN_START, INPUT_PULLUP)` in `setup()` |
| Taster prellt (mehrfach auslösen) | Mechanisches Prellen | Software-Debounce: Zustandswechsel nur nach 20 ms stabil |
| STOP löst sofort beim Start aus | Taster nicht an GND | Taster zwischen Pin und GND schalten (nicht an 5V!) |
| Taster hängt dauerhaft | Kurzschluss nach GND | Kabel prüfen, Pin mit Multimeter messen |

**Debounce-Grundmuster:**
```cpp
bool isStartPressed() {
    if (digitalRead(PIN_START) == LOW) {
        delay(20);
        return digitalRead(PIN_START) == LOW;
    }
    return false;
}
```

---

## 7. Zustandsautomat / Fahrverhalten

**Symptom:** Fahrzeug verhält sich unerwartet, bleibt hängen oder fährt falsche Richtung.

| Problem | Ursache | Lösung |
|---------|---------|--------|
| Fahrzeug startet nicht nach START-Druck | Zustand bleibt in IDLE | `isStartPressed()` im `updateStateMachine()` prüfen |
| Fahrzeug dreht zu früh ab | Schwellwert F < 60 cm zu groß | Schwellwert verkleinern oder Sensor neu kalibrieren |
| Fahrzeug weicht immer zur selben Seite aus | dL/dR-Kalibrierung asymmetrisch | Sensoren einzeln vermessen und Korrekturfaktor anpassen |
| Rückwärts-Manöver endet nie | Rückfahr-Timer zu lang oder dL≈dR | Timer verkürzen; Totband für dL/dR-Vergleich einführen |
| Fahrzeug dreht auf der Stelle | PWM-Differenz zu groß | `Kp` (Regelfaktor) verkleinern |
| Watchdog-Reset tritt auf | Loop dauert > 500 ms | `Serial.print` minimieren; `delay()` entfernen |

---

## 8. Serieller Monitor

**Symptom:** Kein Output, Zeichensalat oder Programm hängt.

| Problem | Ursache | Lösung |
|---------|---------|--------|
| Kein Output | `Serial.begin()` fehlt | `Serial.begin(9600)` in `setup()` |
| Zeichensalat | Falsche Baudrate im Monitor | Baudrate auf **9600** stellen |
| Monitor leer nach Upload | Board resettet sich ständig | Watchdog oder Stromversorgung prüfen |
| Upload blockiert wenn Monitor offen | Gleicher COM-Port | Monitor schließen vor jedem Upload |
