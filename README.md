# SeriellerPortHTML_SML1 — SML-Konsole für eHZ-Zähler

`SeriellerPortHTML_SML1.html` ist eine einzelne HTML-Datei (kein Server, keine
Installation), mit der sich ein elektronischer Haushaltszähler (eHZ) mit
SML-Protokoll nach **DIN EN 62056-21** über einen seriellen IR-Lesekopf
auslesen und debuggen lässt. Empfangene Bytes werden als Hex-Terminal
angezeigt, erkannte SML-Transportdateien werden live dekodiert und die
enthaltenen OBIS-Messwerte tabellarisch dargestellt.

## Voraussetzungen

| Anforderung | Hinweis |
|---|---|
| **Nur Chromium-basierter Browser** | Google Chrome, Microsoft Edge, Opera, Brave etc. — **nicht** Firefox oder Safari |
| Web Serial API | Ist in Chromium integriert; muss nicht aktiviert werden |
| Aufruf über `https://` oder `http://localhost` bzw. `file://` | Die Web Serial API steht in Chromium nur in sicheren Kontexten zur Verfügung |
| Serieller IR-Lesekopf (USB) | z. B. TTL-USB-Adapter am optischen Ausgang des Zählers |

> **Warum nur Chromium?** Das Tool nutzt die Web Serial API
> (`navigator.serial`). Diese existiert nur in Chromium-basierten Browsern.
> Firefox und Safari unterstützen sie nicht.

## Serielle Parameter für eHZ-Zähler (DIN EN 62056-21)

Für das SML-Protokoll eines eHZ-Zählers gelten folgende Einstellungen:

```
9600 Baud, 8 Datenbits, keine Parität, 1 Stoppbit  →  9600 8N1
```

**Wichtig:** Die Voreinstellungen der Datei sind abweichend (**7E1** —
7 Datenbits, gerade Parität). Vor dem Verbinden im linken Panel umstellen auf:

- Baudrate: `9600`
- Bytelänge: `8`
- Parität: `Keine`
- Stopbits: `1`
- Flusssteuerung: `Keine`

## Bedienung

1. HTML-Datei im Chrome/Edge öffnen (Doppelklick genügt, `file://` ist erlaubt).
2. IR-Lesekopf am Zähler anbringen und per USB anschließen.
3. Im Panel **„Verbindung und Format“** die Parameter auf **9600 8N1** einstellen.
4. **„COM-Port auswählen und öffnen“** klicken und das serielle Gerät im
   Browser-Dialog auswählen.
5. Der Zähler sendet zyklisch SML-Dateien — diese erscheinen automatisch
   im Terminal (Hex) und im darunterliegenden **SML-Panel** als dekodierte
   OBIS-Tabelle.

### Terminal-Funktionen

- Empfang (RX) und Versand (TX) als Hex-Bytes mit Zeitstempel
- Neue Zeile bei Trennzeichen (z. B. `0A`) und/oder nach Empfangspause (Idle)
- Senden als Text (UTF-8) oder Hex-Bytes, optional mit angehängtem CR/LF
- Kopieren und Exportieren des kompletten Protokolls als TSV-Datei
- Byte-Zähler für RX/TX

### SML-Live-Parser

Der Parser erkennt automatisch SML-Transportdateien:

- **Startsequenz:** `1B 1B 1B 1B 01 01 01 01`
- **Ende:** `1B 1B 1B 1B 1A` + Füllbytes + CRC16
- Dekodierung der TL-Felder (Listen, Integer, Unsigned, Octet-String, Boolean)
- Erkennung der OBIS-Listeneinträge (6-Byte-OBIS-Code) mit Skalierung und
  Einheit gemäß SML-Spezifikation
- Anzeige als Tabelle: Zeit, OBIS, Bezeichnung, Rohwert, Skalierung, Wert, Einheit
- Bekannte OBIS-Codes werden mit Klartext versehen, z. B.:
  - `1-0:1.8.0*255` — Bezug Gesamt (Wh)
  - `1-0:2.8.0*255` — Lieferung Gesamt (Wh)
  - `1-0:16.7.0*255` — Wirkleistung Gesamt (W)
  - `1-0:32.7.0*255` / `52.7.0` / `72.7.0` — Spannung L1/L2/L3 (V)
  - `1-0:31.7.0*255` / `51.7.0` / `71.7.0` — Strom L1/L2/L3 (A)
  - `0-0:96.1.0*255` — Zähleridentifikation
- Vollständige dekodierte Struktur als JSON-Baum einblendbar
- Export aller erkannten SML-Dateien als JSON (`SML als JSON`)

## Datenschutz

Die Datei arbeitet vollständig lokal im Browser. Es werden keine Daten an
externe Server übertragen — das Auslesen des Zählers und alle Protokolle
bleiben auf dem eigenen Rechner.

## Bekannte Einschränkungen

- Nur Desktop-Chromium mit Web Serial; keine mobilen Browser
- Bei `file://` kann die Clipboard-API eingeschränkt sein (Kopieren ggf. über Export)
- Der SML-Puffer ist auf 2 MB begrenzt (ältere Daten werden verworfen)
