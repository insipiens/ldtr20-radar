# LDTR-20 Bidirectional Speed Monitor  
_ESPHome configuration & helper scripts_

---

## What this project does
* **Reads an LDTR-20 24 GHz Doppler radar** entirely over **RS-485**  
  (brown = B(–), blue = A(+)) using a MAX13487 auto-direction transceiver.  
* **Sends one configuration frame at boot** so the radar streams **both Doppler
  directions** (`V+` and `V-`).  
* **Tracks the fastest target in each direction** during rolling 30-second
  windows and publishes two numeric sensors to Home Assistant.

### Direction conventions

| Raw prefix | HA sensor | Meaning (relative to radar head) |
|------------|-----------|----------------------------------|
| `V+`       | `sensor.ldtr20_peak_outbound` | Vehicle **approaching** (outbound) |
| `V-`       | `sensor.ldtr20_peak_inbound`  | Vehicle **moving away** (inbound) |


Note: You may want to swap the outbound/inbound definitions as it was for a specific use case of mine and would make more sense reversed. 


Values are published in **mph**; if no traffic was seen in a window the sensor
reports **0 mph**.  
Every raw frame (`V±000.0 kph`) is also logged at **INFO** for debugging.

---

## Hardware

| Item | Notes |
|------|-------|
| **LDTR-20** radar module | Red = 9-12 V, Black = GND, **Blue = RS-485 A(+), Brown = RS-485 B(–)** |
| **MAX13487** RS-485 ↔ TTL board | Auto-direction; no DE/RE pin control needed |
| **ESP32** dev board | Any board ESPHome supports |
| 12 V supply | Powers radar + MAX13487; ESP32 can share 5 V via USB/buck |

*No TTL pins on the radar are used; all data travels over the differential
pair through the MAX13487.*

---

## How it works

1. **Boot frame** `43 46 02 00 01 00 0D 0A` tells the radar to  
   * output **both directions** (`0x00`),  
   * keep default report rate (`0x01`),  
   * stay in **km/h** (`0x00`) – conversion to mph happens in ESPHome.
2. A 100 ms `interval:` drains the RS-485 stream, logs each frame, and
   maintains inbound/outbound peak variables.
3. Two template sensors publish those peaks every 30 s;
   they reset to **0 mph** if no traffic was detected.

---

## Quick-start

```bash
git clone <repo-url>
cd ldtr20-radar
# add your Wi-Fi secrets to secrets.yaml
esphome run ldtr20_radar.yaml

After flashing, Home Assistant will auto-discover:
	•	LDTR-20 Peak Outbound – fastest V+ target (moving toward)
	•	LDTR-20 Peak Inbound  – fastest V- target (moving away)

⸻

Customisation

Goal	How
One-direction only	Change the direction byte in the boot frame: 0x01 = inbound only, 0x02 = outbound only.
Native mph output	Units byte 0x00 → 0x01; then remove the × 0.621 conversion in YAML.
Different window length	Edit update_interval: of each template sensor.
Mute raw logging	Set logger: level: WARNING or comment out ESP_LOGI("ldtr_raw", …).

See docs/LDTR04-datasheet-CN.pdf for the full CF… command set
(sensitivity, minimum speed, etc.) – the LDTR-20 uses the same protocol.

⸻

License & credits
	•	Datasheet excerpts © Leide Technology
	•	RS-485 ESPHome integration by @YourName

MIT License — modify and share freely.



