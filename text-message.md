# 📲 Scenario Text Message API

## Version History

| Version | Date | Changes |
| --- | --- | --- |
| v1.0 | 2021-09-27 | Initial draft |
| v1.1 | 2025-08-08 | Reworked structure and fixed some wordings. |

---

## 📘 Overview

This API enables triggering safeREACH scenarios via SMS using a syntax-based command sent to a dedicated service number. It’s ideal for:

- Simple mobile phones and offline-capable devices
- Hardware integrations (e.g. GSM modules, landline-to-SMS bridges)
- Redundant or fallback alerting when internet access is unavailable

---

## 🌐 SMS Number Selection

| Region | Login URL | SMS Target Number |
| --- | --- | --- |
| Frankfurt | `https://start.safereach.com/` | `+43 676 8009 37300` |
| Vienna | `https://start.safereach.at/` | `+43 676 8009 37460` |

> ℹ️ If you're unsure which region you're in, contact support.
> 

---

## ✅ Activation Requirement

Before using this API, **your sender number (MSISDN)** must be **explicitly enabled by the safeREACH support team**.

If you send SMS from an unregistered number, the trigger will be ignored.

---

## 🔑 Syntax Structure

Each SMS consists of:

1. A series of codes that configure the scenario trigger
2. (Optional) additional free-text alarm message
3. (Optional) coordinates in a supported format

All parts must be sent as a single SMS string.

---

### ✅ Required Parameters

| Code | Description | Example | Required |
| --- | --- | --- | --- |
| `K` | Customer ID | `K500027` | ✅ |
| `V` | Scenario number (1–999) | `V1` | ✅ |

---

### 🧩 Optional Parameters

| Code | Description | Example |
| --- | --- | --- |
| `G` | Additional group ID to include in alert | `G1` |
| `Q` | Override reply configuration (0 disables replies) | `Q0` |
| `I` | Override alarm type (e.g. info instead of alarm) | `I` |
| `Z` | Override recipient confirmation behavior | `Z` |
| `M` | Identifier for identical alerts (used for deduplication) | `M42` |
| `T` | Additional recipient MSISDN in [E.164 format](https://en.wikipedia.org/wiki/E.164) | `T+4366412345678` |

---

## 💬 Alarm Text

Any **free-form text** following the parameter section will be used as the **alarm message**.

You can also include **coordinates** (for display on the map), either as:

- GPS coordinates (WGS84)
- National coordinate formats

---

## 📍 Coordinate Formats

### 🧭 WGS84 (Longitude, Latitude)

Use square or round brackets:

```text
[48.220778,16.3100209]
(48.220778,16.3100209)
```

> Example (Vienna): (48.220778,16.3100209)
> 

---

### 🗺️ Easting and Northing Format

```text
XY:4468503.333/5333317.780
```

> Example (Munich): XY:4468503.333/5333317.780
> 

---

## ✉️ Full Example

```text
K500027V1G1T+4366412345678:Fire in server room(48.205587,16.342917)
```

### Breakdown:

- `K500027` → Customer ID
- `V1` → Scenario number
- `G2` → Add group 2
- `T+4366412345678` → Add extra recipient
- `"Fire in server room"` → Free-text alarm message
- `(48.205587,16.342917)` → Coordinates

---

## 🛠️ Notes

- Ensure that there are **no spaces between parameter codes**.
- A colon (`:`) **must separate** the parameter section from the free-text message.
- Maximum SMS length applies (typically 160 characters per message for GSM-7 encoding).
- Multiple `G`, `T` values can be chained (e.g. `G1G2G3T+4366...T+4917...`)
