# Aerostratospheric – Non-human biologics tank temperature logger (ESP32)

Logs tank **water temperature** (DS18B20), **room air temperature and humidity** (DHT22) every **10 minutes** and appends each reading as a CSV row to a file in a GitHub repo through the GitHub Contents API. No Raspberry Pi; the ESP32 posts over WiFi on its own.


**Data repo:** `Midwest-Stratospheric/non-human-biologics-data`. Log files: `data/tank_log_0001.csv`, `_0002`, … (size-based rollover), active name in `data/CURRENT.txt`.

## Files
```
temp_logger/temp_logger.ino     main sketch
temp_logger/config.h            pins, interval, alert range, queue size
temp_logger/github_ca.h         public root CAs for api.github.com (not secret)
temp_logger/secrets.example.h   template -> copy to temp_logger/secrets.h
secrets.example.h               same template (root copy)
.gitignore                      excludes secrets.h
wiring.svg / wiring.png         wiring diagram
```

## Pin map
| Signal | ESP32 pin | Notes |
|---|---|---|
| DS18B20 DATA (yellow) | **GPIO4** | Not a strapping pin; ADC2 conflict doesn't matter for digital OneWire. 4.7 kΩ pull-up DATA→3V3 |
| DHT22 DATA (OUT) | **GPIO15** | Strapping pin (only controls boot-log output). DHT idles HIGH with module pull-up, so boot is unaffected |
| Status LED | **GPIO2** (onboard) | Strapping pin, but safe as output after boot |
| Sensor power | 3V3 / GND | Both sensors on 3.3 V |

Safe alternatives if you need to move pins: GPIO 13, 14, 16, 17, 18, 19, 21, 22, 23, 25, 26, 27, 32, 33. Avoid GPIO 0, 2, 5, 12, 15 for inputs that could be pulled at boot (12 must be LOW at boot), GPIO 6–11 (flash), and 34–39 (input-only, no pull-ups). ADC2 pins (0, 2, 4, 12–15, 25–27) only matter for analogRead while WiFi is on; this project uses no analog input.

## Wiring
```
            DS18B20 probe (in water)                       DHT22 module (room air)
          Red  Black  Yellow                               +    OUT    -
           |     |      |                                  |     |     |
           |     |      +------+-------- GPIO4             |     |     |
           |     |             |                           |     +---- GPIO15
           |     |          [4.7k]                         |           |
           |     |             |                           |           |
  3V3 -----+-----|-------------+---------------------------+           |
  GND -----------+-----------------------------------------------------+
                         ESP32 DevKit (ESP-WROOM-32)
                         GPIO2 = onboard LED   USB = power
```
See `wiring.svg` / `wiring.png`.

## Arduino IDE setup
1. Arduino IDE 2.x → **File › Preferences › Additional boards manager URLs**:
   `https://espressif.github.io/arduino-esp32/package_esp32_index.json`
2. **Tools › Board › Boards Manager** → install **esp32 by Espressif Systems** (3.x).
3. Board: **ESP32 Dev Module**. Upload speed 921600 (or 115200 if uploads fail). Partition scheme: default (includes a SPIFFS/LittleFS partition).
4. **Library Manager**: install **OneWire** (Paul Stoffregen), **DallasTemperature** (Miles Burton), **DHT sensor library** (Adafruit) + **Adafruit Unified Sensor**, **ArduinoJson** (v7). WiFi, WiFiClientSecure, HTTPClient, LittleFS and mbedtls base64 come with the ESP32 core.

## Create the GitHub token (Jeremy does this)
1. The repo `Midwest-Stratospheric/non-human-biologics-data` already exists.
2. GitHub → **Settings › Developer settings › Personal access tokens › Fine-grained tokens › Generate new token**.
3. Resource owner: the account/org that owns the repo. Expiration: set one (e.g. 1 year) and diary renewal.
4. **Repository access: Only select repositories** → `non-human-biologics-data`.
5. **Permissions › Repository › Contents: Read and write** (Metadata: read-only is added automatically). Nothing else.
6. Copy the token once; it goes only into `secrets.h` on your own machine.  If the repo is in an org, the org may need to approve fine-grained tokens.

## Configure and flash
1. Copy `temp_logger/secrets.example.h` → `temp_logger/secrets.h`; fill WIFI_SSID, WIFI_PASS (2.4 GHz network), GH_TOKEN, GH_OWNER, GH_REPO, GH_DIR (`data`), GH_PREFIX (`tank_log_`). Never commit `secrets.h`.
2. Open `temp_logger/temp_logger.ino`, connect the board over USB, choose the port, **Upload**. If it hangs on "Connecting…", hold **BOOT** until upload starts.
3. **Serial Monitor** at **115200** baud. Expected: DS18B20 device count, `[row] ...` every 10 min, `[gh] uploaded N row(s)`.

## CSV format
```
timestamp_iso,water_C,water_F,air_C,humidity_pct,in_range
2026-10-28T14:10:00-05:00,21.06,69.91,22.4,41.3,1
2026-10-28T14:20:00-05:00,NA,NA,22.5,41.1,NA
```
* Timestamp: local America/Chicago with offset (CDT/CST handled automatically) from NTP.
* Failed readings (DS18B20 −127 = disconnected, 85 = power-on reset value; DHT22 NaN) are written as `NA`, never a substitute value.
* `in_range`: `1` water inside target, `0` outside, `NA` no water reading.
* File is created with the header on first run (GitHub 404).

## File rollover (size-based)
* Before each append the logger lists `data/` (metadata only, uses the Contents API `size` field, no download).
* If current file size + new rows > **`ROLLOVER_BYTES` = 64 KB** (`config.h`), it moves to the next file: `tank_log_0001.csv` → `tank_log_0002.csv` …, each starting with the header row. ~64 KB ≈ 1,090 rows ≈ 7.5 days at 10-min sampling.
* The active index is stored in NVS (`Preferences`, namespace `tanklog`, key `fileIdx`) and survives reboots and reflashing (unless flash is erased).
* `data/CURRENT.txt` holds the active filename (e.g. `tank_log_0001.csv`). On boot, if NVS is empty (new board / erased flash), the logger reads `CURRENT.txt` and continues that file; if it is missing too, it starts at 0001 and writes `CURRENT.txt`.
* **Mapping of the old file:** the original `data/tank_log.csv` (header only, no data rows) is superseded by `data/tank_log_0001.csv`. The firmware never writes `tank_log.csv`; it can be left as-is or deleted.
* To force a new file manually: edit `CURRENT.txt` to the next name **and** erase NVS (Tools › Erase All Flash Before Sketch Upload), or simply let it roll.

```
  boot ─ NVS fileIdx? ─yes─> use it
            │no
            └─> GET data/CURRENT.txt ─found─> parse NNNN
                          │404
                          └─> NNNN=0001, PUT CURRENT.txt
  append ─ list data/ (size) ─ size+rows > 64 KB? ─yes─> NNNN+1, save NVS, PUT CURRENT.txt
                                    │no
                                    └─> GET file (sha) ─ append ─ PUT (404 → header first)
```

## Alert range
Water target **68–72 °F (20.0–22.2 °C)**. Outside it: `in_range=0`, `[ALERT]` line on Serial, onboard LED **solid ON**.
LED: solid = out of range · fast blink = sensor fault · slow blink = rows waiting to upload · off = all good. Change limits in `config.h`.

## Reliability behaviour
* Each row goes into a RAM ring buffer (7 days capacity) mirrored to LittleFS (`/queue.csv`), so it survives reboots and power cuts.
* Upload: GET file (sha + content) → append queued rows oldest-first (up to 48 per commit) → PUT base64. 409/422 sha conflict → re-fetch and retry (4 tries).
* WiFi/API failure → reconnect with exponential backoff 30 s → 30 min; backlog flushes when back online.
* Rows are only logged once NTP time is set. After a reboot with no network, sampling waits for NTP (nothing fake is logged).

## Troubleshooting
| Symptom | Fix |
|---|---|
| `DS18B20 devices found: 0` / all water `NA` | Check 4.7 kΩ pull-up between DATA and 3V3, yellow on GPIO4 |
| Water reads `85` → `NA` once at boot | Normal power-on value, discarded |
| Air/humidity `NA` | DHT22 wiring on GPIO15; module needs ≥2 s between reads (fine at 10 min) |
| `[gh] PUT 401/403` | Token expired, wrong repo selected, or Contents not Read/write |
| `[gh] GET failed: 404` repeatedly with PUT 404 | GH_OWNER/GH_REPO typo or branch name not `main` |
| TLS / `-1` errors | Clock not synced, or GitHub changed CA: update `github_ca.h` (`openssl s_client -showcerts -connect api.github.com:443`). `setInsecure()` is a commented debug-only fallback: it disables server verification and could expose the token |
| `[roll]` lines on Serial | Normal: shows the active file, NVS index and rollovers. If files grow past ~150 KB, lower `ROLLOVER_BYTES` |
| Won't boot after rewiring | Something is holding GPIO12 HIGH or GPIO0 LOW at boot |

## Build check
Compiled 2026-10-28 with arduino-cli 1.5.1, esp32:esp32@3.3.12, FQBN `esp32:esp32:esp32`, throwaway placeholder secrets.h (deleted): 1,112,808 B flash (84%), 65,840 B globals (20%). Only warnings are inside the OneWire library. Not yet tested on hardware.
