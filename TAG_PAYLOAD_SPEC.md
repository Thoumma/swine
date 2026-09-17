# Swine Seeker — Ear Tag Payload Spec v0.1

Contract between the ESP32 ear tag (Jack) and the dashboard (Thoum).
Build the firmware against this and the dashboard will accept it without changes.

## Transport

One HTTPS `POST` per wake cycle, every **30 minutes**.

```
POST /api/v1/readings
Content-Type: application/json
Authorization: Bearer <per-tag token>
```

Wake → read sensors → POST → sleep. One request per wake, not one per sensor.
If the POST fails, buffer the reading in RTC memory and send up to **48 queued
readings** (24 h) in the `backlog` array on the next successful connection.
Never block sleep on a retry — battery is the scarce resource, not bandwidth.

## Payload

```json
{
  "tag_id": "TAG-07",
  "farm_id": "nongteng-01",
  "ts": 1789459200,
  "seq": 4412,
  "temp_c": 40.8,
  "activity_idx": 18.4,
  "hr_bpm": 112,
  "spo2_pct": 93,
  "batt_pct": 84,
  "rssi_dbm": -67,
  "flags": ["hr_low_confidence"],
  "backlog": []
}
```

| Field | Type | Unit / range | Source | Notes |
|---|---|---|---|---|
| `tag_id` | string | `TAG-NN` | flashed | Stable for the life of the tag. Pig↔tag mapping lives on the server, not the tag. |
| `farm_id` | string | slug | flashed | |
| `ts` | int | Unix seconds, UTC | NTP on wake | If NTP fails, send `null` and the server stamps arrival time. |
| `seq` | int | monotonic | RTC counter | Survives deep sleep. Lets the server detect gaps without clock trust. |
| `temp_c` | float | 30.0–45.0, 1 dp | LM35 | Tag-surface temperature, **not** core. See calibration below. |
| `activity_idx` | float | 0–200, 1 dp | MPU6050 | % of this pig's own rolling 7-day baseline. See below. |
| `hr_bpm` | int | 30–200 | MAX30100 | `null` if no lock within the sampling window. |
| `spo2_pct` | int | 70–100 | MAX30100 | `null` if no lock. |
| `batt_pct` | int | 0–100 | ADC on Li-Po | |
| `rssi_dbm` | int | −100–0 | ESP32 Wi-Fi | Drives the "signal" indicator and helps site AP placement. |
| `flags` | string[] | see below | firmware | Optional. Empty array if nothing to report. |
| `backlog` | object[] | ≤48 entries | RTC buffer | Same shape, minus `backlog`. Empty on the normal path. |

**Send `null`, never `0`, for a failed sensor read.** A zero is a valid-looking
value that will be scored as a critical anomaly and send someone to the wrong pen.

### `flags` vocabulary

`hr_low_confidence` · `spo2_low_confidence` · `temp_sensor_fault` ·
`imu_fault` · `cold_boot` · `backlog_flush` · `low_battery`

## activity_idx — what the tag must compute

The dashboard compares each pig against **itself**, not against the herd, because
a nursing sow and a grower have completely different normal activity. The tag does
the reduction so it sends one number instead of 30 minutes of accelerometer data:

1. Sample the MPU6050 in short bursts across the wake window.
2. Sum the magnitude of acceleration change above a movement threshold → raw count.
3. Report as a percentage of the pig's rolling 7-day median for the same
   time-of-day bucket (pigs move on a daily rhythm; comparing 02:00 against a
   flat all-day average produces false alarms every night).
4. During the first 7 days on a new animal, send the raw count and set
   `cold_boot` — the server will hold off on activity scoring until a baseline exists.

If the 7-day baseline is easier to keep server-side, send the raw count every time
and say so — the server can do step 3. Decide this once, don't mix the two.

## Calibration note (open hardware question)

`temp_c` from an ear-mounted LM35 reads the **tag**, which tracks ambient
temperature as well as the pig. Before the thresholds below mean anything, the
prototype needs a calibration pass: simultaneous tag readings against rectal
temperature across a range of barn ambient conditions, to establish the offset and
how much ambient compensation is required. Until that exists, treat `temp_c` as a
relative trend against the animal's own baseline rather than an absolute number.

## Thresholds the dashboard currently applies

Provisional — they need a vet's sign-off before any field deployment.

| Metric | Normal | Watch | Urgent |
|---|---|---|---|
| Temperature °C | 37.6 – 39.3 | 39.3 – 40.0 | > 40.0 |
| Activity (% of own baseline) | > 85 | 70 – 85 | < 70 |
| Heart rate bpm | 55 – 90 | 90 – 105 | > 105 |
| SpO₂ % | ≥ 95 | 92 – 95 | < 92 |

**Flagging rule:** a pig is flagged when any one vital is *urgent*, **or** when two
or more vitals leave *normal* at once. Two weak signals together beat one strong
one — that is what buys the lead time, since motion falls before fever appears.
A flag requires **3 consecutive readings** (90 minutes) so a single bad sample
cannot send a farmer across the barn.

## Server response

```json
{ "ok": true, "next_interval_s": 1800 }
```

The tag should honour `next_interval_s` so the farm can slow reporting to save
battery, or speed it up for a pig under watch, without reflashing tags.
