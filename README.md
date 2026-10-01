# Zabbix template – Legrand UPS (CS121 network card)

Zabbix 7.4 template for monitoring **Legrand UPS** units with the **CS121 SNMP network card** (Generex). It uses the standard **UPS-MIB (RFC 1628)**, so it should also work with other UPS units that use a CS121 card.

## ✨ Highlights

- ⚡ **Automatic phase discovery**: **input, output and bypass phases are discovered automatically, based on how many phases the UPS reports.** A 1-phase UPS gets one set of items, a 3-phase UPS gets three, each with its own triggers and graphs. The same template works for both, and you don't have to set anything by hand.
- 🔋 **Battery monitoring**: status, charge, minutes remaining, voltage, temperature and time on battery
- 🚨 **Ready-to-use triggers**: on battery, low charge, overload, voltage / frequency out of range, alarms, input line problems

## Contents

| File | Description |
|------|-------------|
| `template_legrand_ups_cs121.yaml` | Zabbix export with the template `HW UPS Legrand CS121` and the host graph `Battery` |

### `HW UPS Legrand CS121` (link to the UPS host)

**Items**
- Battery: status, estimated charge remaining (%), estimated minutes remaining, voltage, temperature, seconds on battery
- Health: number of active alarms, input line bads counter
- Identification: vendor, model, agent software version, name, location, uptime (filled into host inventory)
- ICMP ping / loss / response time

**Discovery rules (phases)**

Each rule reads its table from UPS-MIB and creates one set of items **per phase found**:

| Rule | MIB table | Items per phase |
|------|-----------|-----------------|
| **UPS Input Phases** | `upsInputTable` (`1.3.6.1.2.1.33.1.3.3`) | frequency, voltage, current, power |
| **UPS Output Phases** | `upsOutputTable` (`1.3.6.1.2.1.33.1.4.4`) | voltage, power, load (%) |
| **UPS Bypass Phases** | `upsBypassTable` (`1.3.6.1.2.1.33.1.5.3`) | voltage, current, power |

Each phase also gets its own graph (`UPS Input Phase N`, `UPS Output Phase N`, `UPS Bypass Phase N`).

**Triggers**

| Trigger | Severity |
|---------|----------|
| HOST DOWN (unavailable by ICMP ping) | Disaster |
| UPS running on battery (> 30 s) | High |
| Battery status not normal | High |
| Battery charge depleted (< 20 %) | High |
| Battery temperature high (> 60 °C) | High |
| UPS input power line bad | High |
| UPS overloaded phase N (> 60 %) | High |
| Output voltage outside nominal phase N (< 200 V or > 250 V) | High |
| High ICMP ping loss | High |
| Battery charge less than 80 % | Warning |
| Battery time remaining below 10 minutes | Warning |
| Battery temperature warning (> 50 °C) | Warning |
| UPS alarm present | Warning |
| Input frequency outside nominal phase N (< 47 Hz or > 53 Hz) | Warning |
| UPS low output power phase N (< 10 W) | Warning |
| UPS bypassed phase N | Warning |
| High ICMP ping response time | Warning |
| UPS load changed phase N (± 1000 W) | Info |
| Device rebooted | Info |

All triggers depend on *HOST DOWN*, so when the UPS is unreachable you only get one alert. Most triggers can be closed manually and are tagged `Loc: {INVENTORY.LOCATION1}`.

## Requirements

- Zabbix server / proxy **7.4** or newer
- SNMP (v1 / v2c) enabled on the CS121 card, reachable from the Zabbix server / proxy
- `fping` installed on the server / proxy (ICMP items)

## Installation

1. **Import the template**: *Data collection → Templates → Import* → `template_legrand_ups_cs121.yaml`
2. **Create the UPS host**:
   - Add an SNMP interface (CS121 IP address) and set the SNMP community
   - Link the template `HW UPS Legrand CS121`
   - Turn on host inventory (*Automatic*) if you want vendor, model and location filled in
3. Wait for discovery to run (phase discovery runs once a day; you can trigger it with *Execute now*).

## Macros

| Macro | Default | Description |
|-------|---------|-------------|
| `{$ICMP_LOSS_WARN}` | `20` | Ping loss threshold (%) |
| `{$ICMP_RESPONSE_TIME_WARN}` | `0.15` | Ping response time threshold (s) |

Set the SNMP community **on the host** (or as a global macro), never in the template itself.

## Notes

- The thresholds for voltage (200–250 V), frequency (47–53 Hz), load (60 %) and battery temperature (50 / 60 °C) are set for a 230 V / 50 Hz grid. Edit the trigger prototypes if your UPS or grid is different.
- If your UPS has no bypass, the *UPS Bypass Phases* rule finds nothing and creates no items.

## Custom work & support

Need something extra? I can extend or customize this template for your company's needs, for example new metrics, triggers, dashboards, other UPS models or integration with your environment. Feel free to get in touch: 📧 [info@duprtech.sk](mailto:info@duprtech.sk)

If this template saved you time and you're happy with my work, you can buy me a coffee ☕

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/duprtech)

## License

[MIT](LICENSE)
