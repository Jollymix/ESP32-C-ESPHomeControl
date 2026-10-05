# Sensornode – ESP32-C3 SuperMini + PT100 + AHT20/BMP280 + OLED

ESPHome-node med innebygd 0,42" OLED (72×40) som rapporterer til Homey Pro over ESPHome Native API (kryptert).

## Koblingsskjema

| Modul | Signal | ESP32-C3 |
|---|---|---|
| OLED (innebygd) | SDA / SCL | GPIO5 / GPIO6 (0x3C) |
| AHT20+BMP280 | SDA / SCL | GPIO5 / GPIO6 (0x38, 0x77/0x76) |
| AHT20+BMP280 | VIN / GND | 3V3 / GND |
| MAX31865 | CLK | GPIO4 |
| MAX31865 | SDO (MISO) | GPIO3 |
| MAX31865 | SDI (MOSI) | GPIO10 |
| MAX31865 | CS | GPIO7 |
| MAX31865 | VIN / GND | 3V3 / GND |
| Status-LED | blå, aktiv lav | GPIO8 |

Ikke bruk GPIO2, GPIO8 og GPIO9 til noe annet. De er strapping- og boot-pinner.

## Flashing

1. `pip install esphome`
2. Fyll inn `secrets.yaml` (WiFi). `api_key` er allerede generert.
3. På Windows: sett `ESPHOME_ESP_IDF_PREFIX=C:\eidf` (se fallgruver).
4. `esphome config sensornode.yaml` må gå uten feil.
5. `esphome run sensornode.yaml --device COM13` (porten vises som «Seriell USB-enhet»).
6. Hvis flashet ikke lykkes: hold **BOOT** inne mens du kobler til USB, og prøv igjen.
7. Senere oppdateringer kan gå over WiFi (OTA er kryptert med `api_key`).

## Kalibrering

AHT20/BMP280 sitter i et eget kammer og leser litt høyt. Juster `substitutions` øverst i `sensornode.yaml`:
`aht_temp_offset = PT100 − AHT20` (f.eks. `"-1.2"`), tilsvarende for `bmp_temp_offset`.

## Homey Pro

Installer **ESPHome Controller**-appen fra Homey App Store, og legg til en enhet. Den eldre appen «ESPhome» (v1.3.12) virker ikke med ESPHome 2026.x, fordi den sender en passord-innlogging som er fjernet fra firmwaren. Homey kobler da til og gir opp etter 5 sekunder. Noden blir funnet via mDNS (`sensornode.local`), eller du kan legge den til med IP-adresse. Lim inn `api_key` fra `secrets.yaml` når appen spør etter encryption key.

## Kjente fallgruver

- **`bmp280` er fjernet**. Plattformen heter nå `bmp280_i2c`.
- **`mains_filter` er 60HZ som standard**. I Norge skal den være `50HZ`.
- **For lange stier på Windows**: ESP-IDF-verktøykjeden overskrider 260 tegn (`bits/os_defines.h: No such file`). Løses med `ESPHOME_ESP_IDF_PREFIX=C:\eidf` eller med long paths slått på.
- **OTA-passord frarådes** i nyere ESPHome. Bruk `ota: encryption:`, som gjenbruker API-nøkkelen.
- **MAX31865 leser 0x0000 eller 0xFFFF**: feil ledning eller strøm, eller MISO og MOSI er byttet om.
- **MAX31865 fault ved romtemperatur**: loddejumperne for 2/3/4-leder matcher ikke `rtd_wires: 3`, eller referansemotstanden ikke er 430 Ω (sjekk koden på motstanden, «4300» = 430 Ω). Ved 20–25 °C skal RTD-en ligge på ca. 108–110 Ω.
- **I2C-scan finner bare 0x3C**: kombomodulen får ikke strøm, eller SDA/SCL er byttet om.
- **C3 SuperMini og WiFi**: noen kort har dårlig antenne. Hvis tilkoblingen er ustabil, sett `wifi: output_power: 8.5dB`.
