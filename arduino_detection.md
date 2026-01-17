# Arduino-Board wird nicht erkannt

## Problem
Ihre Arduino-Platine wird von der IDE nicht erkannt, da USB-Treiber fehlen oder nicht kompatibel sind.

## Lösung

### 1. Geräte-Manager prüfen
- Öffnen Sie den **Geräte-Manager** (Windows)
- Suchen Sie nach unbekannten Geräten oder Geräten mit Warnsymbolen - entweder bei "USB-Controller" oder "Anschlüsse"

### 2. Richtige Treiber finden
- Laden Sie den passenden USB-Treiber für Ihr Board herunter
    - Um die genauen Infos zu erhalten um welches Gerät es sich handelt klicken Sie rechts auf das Gerät mit dem Warndreieck und wählen "Eigenschaften".
    - Wechseln Sie in den Tab Details.
    - Wählen Sie bei Eigenschaft "Hardware-IDs" aus.
    - Der Wert sollte die Hardware Kennung des Geräts anzeigen und bei der Suche nach dem richtigen Treiber helfen.
- **CH340/CH341 Chips**: Arduino Nachbauten - suchen Sie nach "CH340 Treiber" + Ihr Betriebssystem: https://www.roboter-bausatz.de/projekte/ch340-treiber-fuer-arduino-klone-installieren
- **FTDI Chips**: Arduino Original (schon bei Arduino IDE dabei) - laden Sie von ftdichip.com herunter

### 3. Treiber installieren
- Führen Sie die Datei aus (.exe.) oder entpacken Sie die Datei (.zip).
- Klicken Sie mit der rechten Maustaste auf das unbekannte Gerät im Geräte-Manager
- Wählen Sie **Treiber aktualisieren** → **Auf meinem Computer nach Treibern suchen** → Wählen Sie den Ordner mit dem entpackten Treiber in dem sich die *.inf Datei befindet.
- Klicken Sie auf "Weiter".
- Wenn der richtige passende Treiber gewählt wurde, ist das Gerät nun verfügbar.


### 6. Neustart
- Starten Sie die IDE nach der Treiberinstallation neu
- Verbinden Sie das Board erneut

