# Scenario text message API

## Version

- v1.0: Initial draft (2021-09-27)

## General

### Target msisdn

All text messages needs to be sent to: `+43676800937300`

## Trigger a scenario

### Test message code

- **K**{{customerId}} - mandatory e.g. `K500027`
- **V**{{scenarioNumber}} - mandatory e.g. `V1`
- **G**{{groupId}} - optional e.g. `G1` - adds additional groups
- **Q**0 - optional - optional - overwrites reply config
- **I** - optional - optional - overwrites alarm type
- **Z** - optional - optional - overwrite recipients confirmation
- **M**{{indexNumber}} - optional - identifies identical alarms and ignores duplicate alarms
- **T**{{additionalMsisdn}} - optional - adds additional recipients in [ISO E.164](https://en.wikipedia.org/wiki/E.164) format e.g. `T+4366412345678`

#### Alarmtext

The text can be appended `alarmText` after the syntax part and can also contain `coordinates`.


### Coordinates


**Via longitude and latitude (WGS84)**

```
[xx.xxxxxx,yy.yyyyyy]
```

or

```
(xx.xxxxxx,yy.yyyyyy)
```

Example Vienna: `(48.220778,16.3100209)`

**Via "Rechts- & Hochwert"**

```
XY: xxxxxxx.xx / yyyyyyy.yy
```

Example Munich: `XY:4468503.333/5333317.780`


### Example

```
K500027V1:Additional text(48.205587,16.342917)
```

Triggers scenario with the scenario number `V1` and sets coordinates.