# IR-Sensoren-Linearisierung – Sharp GP2Y0A21YK0F & GP2Y0A02YK0F

> Grundlage: Datenblätter Sharp GP2Y0A21YK0F & GP2Y0A02YK0F

---

## Warum Linearisiserung?

Die Sharp IR-Sensoren geben eine analoge Spannung aus, die **nicht linear** mit der Entfernung zusammenhängt. Der ADC-Wert des Arduino fällt mit wachsender Distanz, aber nicht proportional – die Kurve ist hyperbelartig.

Rohdaten können daher **nicht direkt** in Zentimeter umgerechnet werden. Erst durch Linearisierung entsteht eine handhabbare Geradengleichung.

---

## Methode: Kehrwert-Linearisierung

### Schritt 1 – Messpunkte aufnehmen

Den Sensor in bekannte Abstände (z. B. 10, 20, 30 … cm) zur Wand positionieren und den ADC-Rohwert notieren.

### Schritt 2 – Kehrwert bilden

Der Sensor verhält sich näherungsweise wie:

```
ADC ≈ 1 / (l + k)    (mit Korrekturfaktor k)
```

Durch Bildung des Kehrwerts von `(l + k)` erhält man einen Ausdruck, der **linear** vom ADC-Wert abhängt:

```
1 / (l + k) = m · ADC + d
```

### Schritt 3 – Lineare Regression

Die Wertepaare `(ADC, 1/(l+k))` werden in ein Diagramm eingetragen und eine Ausgleichsgerade ermittelt:

- **m** = Steigung der Geraden
- **d** = y-Achsenabschnitt
- **k** = empirisch gewählter Korrekturfaktor (verschiebt die Kurve, bis sie möglichst gut passt)

### Schritt 4 – Gleichung umformen

Auflösen nach `l` (Distanz in cm):

```
l = 1 / (m · ADC + d) - k
```

Mit den Ersatzgrößen `m' = 1/m` und `d' = d/m` ergibt sich die kompakte Formelschreibweise:

```
y = (m' / (x + d')) - k
```

| Symbol | Bedeutung |
|--------|-----------|
| `y` | Distanz in Zentimeter |
| `x` | ADC-Rohwert (0–1023) |
| `m'` | = 1/m (Kehrwert der Steigung) |
| `d'` | = d/m (Verschiebungsfaktor) |
| `k`  | Korrekturfaktor (empirisch bestimmt) |

---

## Kalibrierdaten pro Sensor

### Schritt 1 - Erstelle ein Tabellenkalkulation mit Diagramm

Mit Hilfe eines Tabellenkalkulationprogramms kann die Aufgabe recht leicht gelöst werden. Als Hilfe dient folgende Datei: [1_IR-Sensoren-Linearisierung.xlsx](1_IR-Sensoren-Linearisierung.xlsx)

- Starte mit einem Sensor
- Messe alle 10cm einen Datenpunkt
- Ermittelte die Parameter k, m', d'
- Fahre mit dem nächsten Sensor fort

### Schritt 2 - Baue die Berechnung im Code ein

```cpp
// Sensor Front (A2) – GP2Y0A02YK0F – Bereich: 20–150 cm

const int IR_SENSOR_FRONT = A2;
const int PARAM_M_FRONT = 0;
const int PARAM_D_FRONT = 0;
const int PARAM_K_FRONT = 0;

uint16_t ir_sensor_front_raw = analogRead(IR_SENSOR_FRONT);
ir_sensor_front_new = (uint16_t) (PARAM_M_FRONT / (ir_sensor_front_raw + PARAM_D_FRONT)) - PARAM_K_FRONT;

// limit to min/max specification
if(ir_sensor_front_new > 150)
    ir_sensor_front_new = 151;
else if(ir_sensor_front_new < 20)
    ir_sensor_front_new = 19;
```

> **Hinweis:** Die Formeln gelten nur im kalibrierten Messbereich. Außerhalb (z. B. < 20 cm oder > 150 cm) liefert der Sensor unzuverlässige Werte – daher im Code Bereichsgrenzen prüfen.

---

## Häufige Fehlerquellen

| Problem | Ursache | Lösung |
|---------|---------|--------|
| Werte springen stark | Kein Bypass-Kondensator | 10 µF zwischen VCC und GND direkt am Sensor |
| Formel gibt negative Werte | Objekt zu nah (< Mindestabstand) | Bereichsprüfung im Code |
| Messung weicht ≥ 5 cm ab | Sensor nicht kalibriert / anderes Exemplar | Eigene Messpunkte aufnehmen, neue Regression |
| Sonnenlicht stört | IR-Umgebungslicht sättigt den Empfänger | Sensor abschirmen, in Innenräumen testen |
| Messung bei dunklen Flächen ungenau | Schwarze Oberflächen absorbieren IR | Kalibrierung auf typische Parcours-Farbe anpassen |
