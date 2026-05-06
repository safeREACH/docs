# 📲 Scenario Text Message API

<details>
	<summary><h2>🧩 Version History</h2></summary>


| Version | Date | Changes |
| --- | --- | --- |
| v1.0 | 2021-09-27 | Initial draft |
| v1.1 | 2025-08-08 | Reworked structure and fixed some wordings. |
| v1.2 | 2026-05-06 | Fix G1/G2 mismatch in example; fix WGS84 coordinate order label; clarify colon separator placement; expand SMS character limit note; fix "Login URL" column label; add Scenario API cross-reference; document `I`, `Z`, `Q` parameter values; add silent error discard note. |

</details>

---

## 📘 Overview

This API enables triggering safeREACH scenarios via SMS using a syntax-based command sent to a dedicated service number. For programmatic scenario triggering over HTTP, see the [Scenario API](./api-scenario.md).

It’s ideal for:

- Simple mobile phones and offline-capable devices
- Hardware integrations (e.g. GSM modules, landline-to-SMS bridges)
- Redundant or fallback alerting when internet access is unavailable

---

## 🌐 SMS Number Selection

| Region | Web App URL | SMS Target Number |
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

> 💡 Scenario numbers (`V` codes) can be viewed and managed in the **scenario configuration** in the safeREACH web interface.

---

### 🧩 Optional Parameters

| Code | Description | Values | Example |
| --- | --- | --- | --- |
| `G` | Additional group ID to include in alert | Group ID | `G1` |
| `Q` | Override reply configuration | `Q0` = no reply, `Q1` = single reply, `Q2` = qualitative reply | `Q0` |
| `I` | Send as **info** message instead of alarm | Flag — presence alone activates it, no value needed | `I` |
| `Z` | Disable technical confirmation — recipients are alerted without delivery confirmation (sent blindly) | Flag — presence alone activates it, no value needed | `Z` |
| `M` | Deduplication identifier — a second SMS with the same `M` value is ignored while an alert with that identifier is still active (see note below) | Integer | `M42` |
| `T` | Additional recipient MSISDN in [E.164 format](https://en.wikipedia.org/wiki/E.164) | Phone number | `T+4366412345678` |

---

## 💬 Alarm Text

Any **free-form text** following the parameter section will be used as the **alarm message**.

You can also include **coordinates** (for display on the map), either as:

- GPS coordinates (WGS84)
- The `XY:` easting/northing format (see [Coordinate Formats](#-coordinate-formats) below)

---

## 📍 Coordinate Formats

### 🧭 WGS84 (Latitude, Longitude)

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
- `G1` → Add group 1
- `T+4366412345678` → Add extra recipient
- `Fire in server room` → Free-text alarm message
- `(48.205587,16.342917)` → Coordinates

---

## 🛠️ Notes

- Ensure that there are **no spaces between parameter codes**.
- A colon (`:`) **must separate** the parameter section from the free-text message. The colon comes immediately after the last parameter code, before the alarm text:
  ```text
  K500027V1:Fire in server room
  K500027V1G1T+4366412345678:Fire in server room
  ```
- **SMS length limits:** GSM-7 encoding supports up to **160 characters** per segment. If your message contains non-GSM-7 characters (e.g. certain diacritics, emoji), the message switches to UCS-2 encoding and the limit drops to **70 characters** per segment. Concatenated (multi-part) SMS can exceed one segment but increases cost and may behave differently on some hardware.
- Multiple `G`, `T` values can be chained (e.g. `G1G2G3T+4366...T+4917...`)
- **`M` deduplication is tied to active alerts.** A duplicate is suppressed for as long as an alert with the same `M` identifier is still running. Alerts have a default runtime of 7 days unless terminated earlier.
- **Errors are silently discarded.** If the message is malformed, the customer ID is invalid, the scenario does not exist, or the sender MSISDN is not whitelisted, the trigger is ignored with no feedback to the sender.
