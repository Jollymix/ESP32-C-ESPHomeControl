# Prompt til Claude Code – ESPHome sensornode med OLED og Homey

Lim inn alt under streken i Claude Code, i en tom prosjektmappe. Legg `sensornode.yaml` i samme mappe.

---

Jeg vil bygge en ESPHome-node som viser sensordata på en innebygd OLED og som jeg legger til i Homey Pro. Sett opp prosjektet, valider det, flash det over USB og hjelp meg å verifisere hver del. Svar på norsk.

## Maskinvare
- **MCU:** ESP32-C3 SuperMini med innebygd 0,42" OLED (SSD1306, 72×40, I2C 0x3C på GPIO5/GPIO6). Blå LED på GPIO8, BOOT på GPIO9, USB-C med innebygd USB-Serial/JTAG.
- **MAX31865** (lilla klone, PT100, referansemotstand 430 Ω, 3-leder). Koblet på SPI: CLK=GPIO4, SDO/MISO=GPIO3, SDI/MOSI=GPIO10, CS=GPIO7. RDY er ikke koblet.
- **AHT20+BMP280-kombomodul** på samme I2C-buss som OLED-en (GPIO5/6). AHT20 bruker 0x38, BMP280 bruker 0x77 eller 0x76.
- Alle moduler får 3V3 fra ESP-en.

## Utgangspunkt
`sensornode.yaml` i denne mappen er et utkast. Behandle det som et forslag og ikke som en fasit.

## Oppgaver
1. **Verktøy:** Sjekk at Python og ESPHome er installert (`pip install esphome` eller `pipx`). Bruk siste stabile ESPHome.
2. **Verifiser utkastet mot gjeldende dokumentasjon på esphome.io:**
   - `max31865` (gyldige parametere, støttes `mains_filter`?)
   - `aht10` med `variant: AHT20`
   - BMP280-plattformen (`bmp280_i2c` i nyere versjoner, `bmp280` i eldre)
   - `ssd1306_i2c` med `model: "SSD1306 72x40"`
   - `font` med `gfonts://`, der `°` må tas med i `glyphs` hvis det trengs
   - `display.page.show_next`

   Rett det som er utdatert, og forklar kort hva du endret.
3. **secrets.yaml:** Lag den med plassholdere for `wifi_ssid`, `wifi_password` og `ota_password`. Generer en ny 32-byte base64 `api_key`. Legg `secrets.yaml` i `.gitignore`.
4. **Robusthet:**
   - Legg til `captive_portal` og en fallback-AP.
   - Vis «--» på displayet når en sensor mangler verdi, ikke `nan`.
   - Legg til en `status_led` på GPIO8 (aktiv lav).
   - Legg til en diagnostikkside på displayet med IP og WiFi-signal.
5. **Kompiler og valider:** Kjør `esphome config sensornode.yaml` og deretter `esphome compile sensornode.yaml`. Fiks alle feil.
6. **Flash over USB:** Kjør `esphome run sensornode.yaml --device <port>` og finn porten selv. Hvis det første flashet feiler, be meg holde BOOT inne mens jeg kobler til USB.
7. **Verifiser i loggen**, steg for steg:
   - I2C-scan skal finne 0x3C, 0x38 og 0x76/0x77. Rett BMP280-adressen om nødvendig.
   - MAX31865 skal ikke rapportere feil. Romtemperatur skal gi ca. 108 Ω og 20–25 °C. Hvis den rapporterer fault, forklar sannsynlig årsak, for eksempel at jumperne for 2/3/4-leder ikke matcher `rtd_wires`, eller at referansemotstanden er feil.
   - Displayet skal vise alle sidene.
8. **Homey:** Forklar hvordan noden legges til i Homey Pro via ESPHome-appen i Homey App Store. Sjekk først hvilken app som er aktuell og om den krever `encryption key`. Bekreft at alle sensorene dukker opp som capabilities.
9. **README.md:** Lag en kort fil med koblingsskjema som tabell, flash-prosedyre og kjente fallgruver.

## Kjente forhold
- AHT20 og BMP280 sitter i et eget, ventilert kammer i kabinettet, men leser likevel litt høyt. Legg til en `offset`-filter-parameter jeg kan kalibrere mot PT100.
- Ikke bruk GPIO2, GPIO8 og GPIO9 til noe annet. De er strapping- og boot-pinner.
- Ikke flash noe før `esphome config` går gjennom uten feil.
