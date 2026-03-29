# 🚀 Projekt Modularnej Sieci Meshtastic: "Endgame Setup"

## 📝 Koncepcja Systemu
Projekt zakłada budowę dwóch zaawansowanych węzłów (nodów) sieci Meshtastic, opartych na ekosystemie **Seeed Studio XIAO**. System został zaprojektowany z myślą o maksymalnej elastyczności, energooszczędności i działaniu Off-Grid.

1. **Node 1 (The Tank):** Stricte mobilny, oparty na energooszczędnym chipie nRF52840. Zbudowany do wielodniowej pracy w terenie z zasilaniem solarnym.
2. **Node 2 (The Chameleon):** Hybrydowy node domowo-terenowy oparty na ESP32-S3. Na co dzień pełni rolę stacjonarnej bramki z podłączeniem do domowego Wi-Fi (MQTT) oraz dużą anteną. W razie potrzeby (wyjście w teren, ewakuacja) moduł główny wraz z panelem solarnym jest wyciągany z obudowy balkonowej, zyskuje małą antenę i staje się drugim, pełnoprawnym trackerem mobilnym.

---

## 🎒 NODE 1: Stricte Mobilny (nRF52840)
**Główne zadanie:** Tracker terenowy, praca w plecaku/na rowerze. Ultra-niski pobór prądu dzięki architekturze Nordic, stabilny Bluetooth. Przewidywany czas pracy bez słońca: ok. 7-14 dni.

### 📦 Lista komponentów:
* **1x Płytka główna:** Zestaw XIAO nRF52840 + LoRa Wio-SX1262 (Meshtastic/MeshCore)
* **1x Moduł GPS:** GNSS Quectel L76K dla XIAO (I2C/UART)
* **1x Zasilanie:** Moduł BQ25185 USB/DC/Solar Charger (płynne przełączanie między baterią a solarem)
* **1x Bateria:** Akumulator Li-Po Akyga 3,7V / 4000mAh (z konektorem JST)
* **1x Ekran:** Moduł wyświetlacza OLED 0,96" 128x64 niebieski (E) - Waveshare 24103 (I2C)
* **1x Panel Solarny:** 5V / 6V z przewodem zakończonym wtyczką szybkozłączką (np. wodoszczelne JST lub wtyk DC)
* **1x Antena:** Krótka antena mobilna (SMA męska) wkręcona na stałe do obudowy (np. Gizont / Moxon)

---

## 🏠➡️🎒 NODE 2: Hybryda Balkon / Mobile (ESP32-S3)
**Główne zadanie:** Bramka domowa z Wi-Fi (MQTT) -> szybka transformacja w drugi node terenowy.

### 📦 Lista komponentów:
* **1x Płytka główna:** XIAO ESP32S3 + Wio-SX1262 Kit (Meshtastic/MeshCore LoRa)
* **1x Moduł GPS:** GNSS Quectel L76K dla XIAO
* **1x Zasilanie:** Moduł BQ25185 USB/DC/Solar Charger
* **1x Bateria:** Akumulator Li-Po Akyga 3,7V / 4000mAh
* **1x Ekran:** Moduł wyświetlacza OLED 0,96" 128x64 niebieski (E)
* **1x Panel Solarny:** (Zabierany w teren!) Z przewodem i szybkozłączką
* **1x Antena Balkonowa:** Duża antena np. z włókna szklanego (strojona na 868MHz) zamocowana na stałe na balkonie
* **1x Antena Mobilna:** Mała antena zapasowa, trzymana "w pogotowiu"

### 🔄 Architektura "Matrioszki" (Transformacja Balkon -> Teren)
Node 2 opiera się na koncepcji obudowy modułowej (pudełko w pudełku):

1. **Moduł Wewnętrzny (Mobilny):** Małe, szczelne pudełko zawierające baterię, elektronikę i wyświetlacz. Na obudowie posiada gniazdo antenowe SMA(ż) oraz gniazdo zasilania (JST/DC) na solar.
2. **Stacja Balkonowa:** Kabel od dużej anteny balkonowej wchodzi na balkon. Panel solarny posiada zaczep/uchwyt na balkonie, ale nie jest przyklejony na stałe.
3. **Procedura ewakuacji w teren (W razie "W"):**
   * Odłączasz wtyczkę solara od obudowy modułu.
   * Odkręcasz przewód dużej anteny balkonowej złącza SMA.
   * Wyjmujesz moduł wewnętrzny z koszyka balkonowego.
   * Wkręcasz w gniazdo SMA małą antenę przenośną.
   * Zdejmujesz panel solarny z uchwytu balkonowego.
   * **Ważne (Software):** W aplikacji Meshtastic wyłączasz w Node 2 funkcję Wi-Fi, zostawiając sam Bluetooth. Znacznie wydłuży to pracę baterii procesora ESP32S3 w terenie!

---

## 🛠️ Schemat Połączeń i Lutowania
Zarówno nRF52840 jak i ESP32-S3 z serii XIAO mają **identyczny układ pinów (Pinout)**. Sprzęt lutujesz dokładnie tak samo dla obu nodów.

### 1. Zasilanie (Moduł BQ25185)
| Moduł BQ25185 | Miejsce docelowe | Uwagi |
| :--- | :--- | :--- |
| **VIN (+)** | Czerwony kabel Panelu Solarnego | Przez zewnętrzne gniazdo JST/DC |
| **GND (-)** | Czarny kabel Panelu Solarnego | Przez zewnętrzne gniazdo JST/DC |
| **BAT (+)** | Czerwony kabel Akumulatora 4000mAh | Gniazdo dedykowane na BQ25185 |
| **BAT (-)** | Czarny kabel Akumulatora 4000mAh | Gniazdo dedykowane na BQ25185 |
| **OUT (lub SYS)** | Płytka XIAO: Pin **[5V]** | Zasila całą płytkę główną |
| **GND** | Płytka XIAO: Pin **[GND]** | Zamyka obwód zasilania XIAO |

### 2. Magistrala I2C (GPS i Wyświetlacz współdzielą piny)
| Peryferia (OLED + GPS L76K) | Pin na Płytce XIAO | Uwagi konfiguracji w Meshtastic |
| :--- | :--- | :--- |
| **VCC** / VIN (oba moduły) | Pin **[3V3]** | Wyjście stabilizowanego prądu z XIAO |
| **GND** (oba moduły) | Pin **[GND]** | Wspólna masa |
| **SDA** (OLED) + **RX/SDA** (GPS) | Pin **[D4]** | Ustawienia I2C: SDA = 4 |
| **SCL** (OLED) + **TX/SCL** (GPS) | Pin **[D5]** | Ustawienia I2C: SCL = 5 |

### 3. Magistrala SPI (Moduł LoRa SX1262)
*(Jeśli lutujesz przewodami, a nie nakładasz Wio-SX1262 jako Shield)*

| Moduł LoRa SX1262 | Pin na Płytce XIAO |
| :--- | :--- |
| **3V3** (VCC) | Pin **[3V3]** |
| **GND** | Pin **[GND]** |
| **SCK** (Zegar SPI) | Pin **[D8]** |
| **MISO** (Dane z LoRa) | Pin **[D9]** |
| **MOSI** (Dane do LoRa)| Pin **[D10]** |
| **NSS / CS** (Chip Select)| Pin **[D3]** |
| **DIO1 / IRQ** | Pin **[D1]** |
| **RST / RESET** | Pin **[D0]** |

---

## 💡 Porady Konstruktorskie
1. **Szyny Zasilania (Perfboard):** Użyj małej płytki uniwersalnej (perfboard). Stwórz "kroplę cyny" połączoną z pinem `3V3` XIAO, aby łatwo dolutować do niej 3 kable VCC (od LoRa, OLED i GPS). Zrób to samo dla szyny masowej (`GND`).
2. **Pigtail (Kabel Antenowy):** Z modułu LoRa wyjdź cienkim kablem u.FL i zakończ go gniazdem **SMA (żeńskim)** wklejonym mocno w ścianę obudowy. Dzięki temu przykręcanie/odkręcanie zewnętrznej anteny nie urwie delikatnego złącza u.FL na płytce.
3. **Zabezpieczenie przed wodą:** Miejsca wklejenia gniazd SMA oraz gniazd zasilania solara zalej wewnątrz obudowy klejem epoksydowym, a na zewnątrz stosuj gumowe zaślepki zatykane na czas transportu.