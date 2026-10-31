# Logger setup notes

- [Logger setup guide](logger-setup.md): ESP32 hardware, pin map, Arduino IDE setup, configuration, CSV format, alert range, troubleshooting.
- [Wiring diagram (SVG)](wiring.svg)

The logger reads tank water temperature and room air temperature/humidity every 10 minutes and appends rows to [`../data/tank_log.csv`](../data/tank_log.csv).

Credentials go in a local `secrets.h`, which is excluded by `.gitignore` and must never be committed.
