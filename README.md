# 🚀 Projekt Modularnej Sieci MeshCore/Meshtastic – „Endgame Setup v2”

## 📝 Koncepcja Systemu

Projekt zakłada budowę **dwóch węzłów** sieci MeshCore/Meshtastic opartych na ekosystemie **Seeed Studio XIAO**, z myślą o pracy Off‑Grid, minimalnym zużyciu energii i braku obsługi w trybie 24/7.

1. **Node 1 – „Tank” (Mobilny):**
   - Węzeł noszony/terenowy oparty na **XIAO nRF52840**.
   - Ma **GPS L76K** do trackingu.
   - Zasilany z baterii Li‑Po 1S ~4000 mAh z możliwością ładowania z **małego, odpinanego panelu solarnego**.
   - Bez ekranu, bez Wi‑Fi/GSM, nastawiony na długi czas pracy (BLE + LoRa).

2. **Node 2 – „Repeater” (Stacjonarny 24/7):**
   - Węzeł infrastrukturalny oparty również na **XIAO nRF52840**, zamontowany wysoko (np. dach wieżowca).
   - Czysty **LoRa repeater** – bez GPS, bez ekranu, bez Wi‑Fi, bez GSM/MQTT.
   - Zasilany z baterii Li‑Po 1S ~4000 mAh i **panelu 6 V 5–10 W** przez moduł **BQ25185** z buckiem 3.3 V.
   - Zaprojektowany jako **bezobsługowy**, bez przerabiania na mobilny node.

---

## 📊 Sprzęt – Specyfikacja

### 🧠 1. Mikrokontroler (oba nody)

**Seeed Studio XIAO nRF52840**

- Procesor: Nordic nRF52840, ARM Cortex‑M4 32‑bit @ 64 MHz
- Pamięć: 1 MB Flash / 256 KB RAM (+ 2 MB QSPI Flash)
- Łączność: Bluetooth 5.0 (BLE), NFC
- Deep Sleep: rzędu kilku µA (bardzo niskie zużycie)
- Interfejsy: GPIO, I²C, SPI, UART
- Wymiary: 21 × 17.5 mm

### 📻 2. Moduł radiowy LoRa (oba nody)

**Wio‑SX1262 LoRa Kit (dla XIAO)**

- Chip: Semtech SX1262
- Pasmo: 868 MHz (EU868)
- Moc TX: do +22 dBm
- Czułość RX: ok. −137 dBm (SF12)
- Interfejs: SPI + linie sterujące (CS, RST, DIO1)
- Złącze anteny: u.FL (IPEX) → SMA (pigtail)

### 🛰️ 3. Moduł GPS (Tylko Node 1 – Mobilny)

**GNSS Quectel L76K add‑on for XIAO**

- Multi‑GNSS: GPS, GLONASS, BDS, QZSS
- Komunikacja: I²C (możliwy UART)
- Cold Start: < 30 s, Hot Start: < 2 s
- Wbudowana antena ceramiczna (patch) – wymaga „widoku nieba” w obudowie

### ⚡ 4. Zasilanie – moduł ładowania

**Moduł BQ25185 USB/DC/Solar Charger z buckiem 3.3 V**

- Wejścia:
  - USB 5 V – serwis / ładowanie awaryjne
  - Solar/DC: panel 6 V, 3–10 W
- Bateria:
  - 1S Li‑ion/Li‑Po 3.0–4.2 V, prąd ładowania do ok. 1 A
- Funkcje:
  - Power‑path: jednoczesne zasilanie systemu i ładowanie baterii
  - Tryb solar: dostosowanie do paneli słonecznych
  - Buck 3.3 V: zasilanie logiki (XIAO + LoRa (+ GPS w nodzie mobilnym))

### 🔋 5. Bateria i Panel Solarny

**Node 1 – Mobilny**

- Bateria: Li‑Po 1S 3.7 V, ok. **4000 mAh** z PCM (płaska saszetka)
- Panel solarny:
  - 6 V, 3–5 W, **odpinany** (złączka na obudowie)

**Node 2 – Repeater 24/7**

- Bateria: Li‑Po 1S 3.7 V, ok. **4000 mAh** z PCM
- Panel solarny:
  - 6 V, **5–10 W** (min. 5 W jako sensowne minimum, 10 W dla dużego zapasu zimą)

### 🔐 6. Zabezpieczenia i okablowanie (oba nody)

- Bezpiecznik **PPTC 1–1.5 A** na plusie baterii (resetowalny)
- Dioda/transil **TVS 6–7 V** równolegle na wejściu z panelu (VIN–GND)
- Złącza 2‑pin (JST/ARK) dla baterii i panelu
- Przewody silikonowe 22–24 AWG
- Obudowa IP65/IP66 (jasna, ABS/PC) + przepusty kablowe

---

## 🎒 NODE 1 – „Tank” (Mobilny nRF52840 + GPS + Solar)

**Rola:** mobilny tracker / klient MeshCore/Meshtastic z GPS, zasilany z Li‑Po 4000 mAh i małego solara, bez ekranu.

### Hardware Node 1 – podsumowanie

- 1× XIAO nRF52840
- 1× Wio‑SX1262 LoRa Kit (dla XIAO)
- 1× GNSS Quectel L76K (I²C)
- 1× BQ25185 USB/DC/Solar + buck 3.3 V
- 1× Li‑Po 1S 3.7 V ~4000 mAh z PCM
- 1× Panel 6 V 3–5 W (odpinany)
- 1× Antena mobilna 868 MHz (SMA, np. Gizont/Moxon)
- 1× PPTC 1–1.5 A + 1× TVS 6–7 V

### Node 1 – schemat blokowy (Mermaid)

```mermaid
flowchart TD

  %% Źródła energii
  SOLAR[Panel 6V 3-5W (odpinany)]
  USB[USB-C (serwis / ładowanie)]

  %% Moduł ładowarki + 3V3
  subgraph CHG[Moduł BQ25185 USB/DC/Solar + buck 3V3]
    VIN[VIN / SOLAR+]
    VUSB[VIN_USB]
    BATTp[BATT+]
    BATTm[BATT- / GND]
    SYS3V3[SYS / 3V3 OUT]
    GND[SYS GND]
  end

  %% Bateria
  subgraph BAT[LiPo 1S 3V7 ~4000mAh z PCM]
    Bp[Bateria +]
    Bm[Bateria -]
  end

  %% Zabezpieczenia
  FUSE[PPTC 1-1A5<br/>na plusie baterii]
  TVS[TVS 6-7V<br/>na wejściu PV]

  %% Node mobilny
  subgraph NODE[Node mobilny MeshCore/Meshtastic]
    XIAO[XIAO nRF52840<br/>3V3/GND, BLE + LoRa]
    LORA[Wio-SX1262 LoRa<br/>SPI 3V3/GND]
    GPS[L76K GNSS<br/>I2C 3V3/GND]
  end

  %% Wejścia zasilania
  SOLAR -->|+| VIN
  SOLAR -->|−| GND
  USB -->|+| VUSB
  USB -->|−| GND

  %% TVS równolegle na wejściu PV
  TVS -->|+| VIN
  TVS -->|−| GND

  %% Bateria + PPTC
  Bp --> FUSE --> BATTp
  Bm --> BATTm

  %% Wyjście 3V3 do noda
  SYS3V3 --> XIAO
  SYS3V3 --> LORA
  SYS3V3 --> GPS
  GND --> XIAO
  GND --> LORA
  GND --> GPS

  %% Logika
  XIAO --- LORA
  XIAO --- GPS
```

### Node 1 – kluczowe połączenia pinów (skrót)

- BQ25185:
  - VIN/SOLAR+ ↔ plus panelu
  - GND ↔ minus panelu
  - VIN_USB ↔ USB 5 V (opcjonalnie)
  - BATT+ ↔ plus baterii przez PPTC
  - BATT− ↔ minus baterii
  - SYS/3V3 ↔ 3V3 XIAO + VCC Wio + VCC GPS
  - GND ↔ GND XIAO + GND Wio + GND GPS

- XIAO ↔ Wio‑SX1262: SPI + CS/DIO1/RST (jak w twoim oryginalnym schemacie)
- XIAO ↔ L76K: I²C (np. D4 = SDA, D5 = SCL)

---

## 🏔 NODE 2 – Repeater 24/7 (nRF52840 + LoRa + Solar)

**Rola:** stały, stacjonarny repeater MeshCore/Meshtastic, zero GPS/ekranu/Wi‑Fi/GSM.

### Hardware Node 2 – podsumowanie

- 1× XIAO nRF52840
- 1× Wio‑SX1262 LoRa Kit (dla XIAO)
- 1× BQ25185 USB/DC/Solar + buck 3.3 V
- 1× Li‑Po 1S 3.7 V ~4000 mAh z PCM
- 1× Panel 6 V **5–10 W** (stały)
- 1× Duża antena 868 MHz (4–8 dBi, włókno szklane)
- 1× PPTC 1–1.5 A + 1× TVS 6–7 V
- Obudowa IP65/IP66, jasna, na maszcie / dachu

### Node 2 – schemat blokowy (Mermaid)

```mermaid
flowchart TD

  %% Źródła energii
  SOLAR[Panel 6V 5-10W (stały)]
  USB[USB-C (serwis / awaryjne ładowanie)]

  %% Moduł ładowarki + 3V3
  subgraph CHG[Moduł BQ25185 USB/DC/Solar + buck 3V3]
    VIN[VIN / SOLAR+]
    VUSB[VIN_USB]
    BATTp[BATT+]
    BATTm[BATT- / GND]
    SYS3V3[SYS / 3V3 OUT]
    GND[SYS GND]
  end

  %% Bateria
  subgraph BAT[LiPo 1S 3V7 ~4000mAh z PCM]
    Bp[Bateria +]
    Bm[Bateria -]
  end

  %% Zabezpieczenia
  FUSE[PPTC 1-1A5<br/>na plusie baterii]
  TVS[TVS 6-7V<br/>na wejściu PV]

  %% Repeater
  subgraph NODE[Repeater MeshCore/Meshtastic 24/7]
    XIAO[XIAO nRF52840<br/>3V3/GND]
    LORA[Wio-SX1262 LoRa<br/>SPI 3V3/GND]
    ANT[Antena 868MHz<br/>wysoko, stała]
  end

  %% Wejścia zasilania
  SOLAR -->|+| VIN
  SOLAR -->|−| GND
  USB -->|+| VUSB
  USB -->|−| GND

  %% TVS równolegle na wejściu PV
  TVS -->|+| VIN
  TVS -->|−| GND

  %% Bateria + PPTC
  Bp --> FUSE --> BATTp
  Bm --> BATTm

  %% Wyjście 3V3
  SYS3V3 --> XIAO
  SYS3V3 --> LORA
  GND --> XIAO
  GND --> LORA

  %% Logika
  XIAO --- LORA
  LORA --- ANT
```

---

## 🔌 Tabela – kluczowe połączenia zasilania (wspólne dla obu nodów)

| Element          | Pin modułu BQ25185 | Do czego lutujesz                            |
|------------------|--------------------|----------------------------------------------|
| Panel +          | VIN / SOLAR+       | Plus panelu 6 V                              |
| Panel −          | GND                | Minus panelu                                 |
| USB 5 V +        | VIN_USB            | Plus z gniazda USB‑C (jeśli używasz)        |
| USB 5 V −        | GND                | Minus USB‑C                                  |
| Bateria +        | BATT+ (przez PPTC) | Plus baterii 1S → PPTC → BATT+              |
| Bateria −        | BATT− / GND        | Minus baterii                                |
| Wyjście 3.3 V    | SYS / 3V3 OUT      | Pin 3V3 XIAO + VCC Wio (+ VCC GPS w Node 1) |
| Masa systemu     | GND                | GND XIAO + GND Wio (+ GND GPS w Node 1)     |
| TVS 6–7 V (+)    | VIN / SOLAR+       | Równolegle do wejścia PV                     |
| TVS 6–7 V (−)    | GND                | Równolegle do wejścia PV                     |
| PPTC 1–1.5 A     | Na przewodzie BAT+ | Szeregowo między baterią a BATT+            |

---

Masz już wszystko w jednej paczce – opis, hardware, schematy Mermaid i tabelę połączeń, gotowe do wklejenia do repo / notatek. Jeśli chcesz, mogę teraz dopisać sekcję „Firmware i konfiguracja MeshCore/Meshtastic” (role, presety, moc TX) jako kolejny blok markdown, w tym samym stylu.