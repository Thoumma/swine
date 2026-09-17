# Swine Seeker — Project Overview

**Team:** Leung Mou Mou
**Members:** Jack (Hardware) · Thoum (Software) · Ning (Research)
**Institution:** National University of Laos (NUOL)

---

## 1. What It Is

**Swine Seeker** is a low-cost smart ear tag that continuously monitors individual pigs and flags the ones that need attention — before visible symptoms appear.

It is an **early-warning system, not a diagnostic test.** It does not identify African Swine Fever. It identifies the animal that needs to be looked at today. The veterinarian still makes the call; the system only decides where to send them first.

---

## 2. The Problem

African Swine Fever (ASF) has been devastating pig farming in Laos since 2019 and is still spreading in 2026.

| Fact | Detail |
|---|---|
| Mortality rate | Up to 90–100% |
| Vaccine status in Laos | **None approved for official use** |
| Affects humans? | No |
| Environmental persistence | Up to 3 years in frozen meat, 1 year in dried meat |
| Wild boar reservoir | Present in Laos since 2020 — cannot be vaccinated or controlled |
| Smallholder biosecurity | Average implementation only **27%** |

**The core gap:** Laos relies on *reactive* control — red/yellow zoning, movement bans, and villagers reporting visible symptoms. By the time a farmer sees that a pig is sick, the virus has usually already spread through the herd.

---

## 3. Timeline of ASF in Laos

| Date | Event |
|---|---|
| **ມິຖຸນາ 2019** | ASF enters Laos for the first time — Saravane province (south) |
| **ກໍລະກົດ 2019** | Pakheuang village, Bolikhamxay — 201 pigs, **154 lost (77%) within one week** |
| **24 ກັນຍາ 2020** | Xiengluang district — spread to 13 villages, 240+ pigs destroyed, damages over 200 million kip |
| **ປີ 2020** | ASF found in wild boar for the first time — spreads beyond human control |
| **ກຸມພາ 2025** | Vanghai village, Xaysomboun declared a **Red Zone** — proves recurrence 6 years later |
| **ມິຖຸນາ–ສິງຫາ 2026** | Houngsa, Xayaboury — 21 villages, **1,949 pigs lost (≈12.7× worse than 2019)**, wild boar deaths in Nam Poui National Protected Area, district-wide pork trade suspended |
| **ມີນາ 2026** | Ministry of Agriculture and Forestry confirms ASF remains an ongoing challenge; **still no approved vaccine in Laos** |

---

## 4. How It Works

### Hardware — the ear tag

| Component | Function |
|---|---|
| **ESP32 Mini** | Microcontroller — processes sensor data, handles Wi-Fi/BLE |
| **LM35** | Body temperature |
| **MPU6050** | Motion and activity level |
| **MAX30100** | Heart rate and blood oxygen (SpO₂) |
| **Li-Po 3.7V** | Battery |
| **Waterproof casing** | Top and lower cover for barn conditions |

### Power strategy
The tag sleeps almost all the time. Every 30 minutes it wakes, reads its sensors, sends one short message over Wi-Fi, and goes back to sleep. This is what makes the battery last without constant recharging.

### Data flow
```
Ear tag (4 sensors)
    → Wireless transmission (Wi-Fi / BLE)
        → Farm server / cloud platform
            → Pattern analysis + anomaly detection
                → Alert to farmer (app / SMS)
                    → Dashboard: pig status, health trends, early alerts
```

---

## 5. Why These Four Metrics

Each sensor choice is grounded in published, peer-reviewed research:

- **Temperature** — ASF studies found infected pigs showed rising core temperature during disease progression
- **Motion** — daily motion decreased by approximately 10% *before* clinical disease was detected
- **Heart rate** — a 2026 study using health-monitoring collars found abnormal pulse rate and HRV readings after infection, some occurring before clinical disease
- **SpO₂** — not a standalone ASF indicator, but an additional physiological signal that complements the others

**The key insight:** all three primary signals change *before* symptoms are visible to the human eye.

---

## 6. Cost

| # | Component | Prototype (qty 1) | Production (qty 100+) |
|---|---|---|---|
| 1 | ESP32 Mini | $7 | $4 |
| 2 | LM35 | $1 | $0.50 |
| 3 | MPU6050 | $6 | $1.50 |
| 4 | MAX30100 | $3 | $2 |
| 5 | Li-Po 3.7V | $3 | $2 |
| 6 | Other materials | $20 | $10 |
| | **Total per unit** | **$40** | **≈$20** |

Roughly **50% cost reduction at scale**, largely driven by the MPU6050 dropping ~4× in bulk.

> **Note:** production figures are market-research estimates from AliExpress/LCSC listings, not confirmed supplier quotes. Getting 2–3 real quotes before pitching would let you say "we've sourced pricing" rather than "we estimate."

---

## 7. Why a Vaccine Alone Isn't Enough

Even if Laos approves an ASF vaccine, continuous monitoring still matters:

- **The 14–28 day gap** — vaccinated pigs are not protected immediately; full immunity takes 2–4 weeks
- **Live-attenuated risks** — virus shedding from vaccinated animals, genetic stability concerns, safety issues in pregnant sows
- **Strain coverage** — current vaccines target genotype II; recombinant I/II strains are already emerging in Vietnam
- **Wild boar** — cannot be vaccinated, remain a permanent reintroduction source
- **WOAH's own guidance** requires post-vaccination monitoring to catch breakthrough infections

---

## 8. Beyond ASF — Other Uses

- **General health surveillance** — PRRS, respiratory disease, heat stress, parasites
- **Post-vaccination monitoring** — detect vaccine breakthrough cases once vaccines arrive
- **Growth and feeding management** — per-animal data for feed planning
- **Animal welfare tracking** — flag stress or poor environmental conditions
- **Other livestock** — adaptable to chickens, cattle, buffalo, goats
- **Research data** — long-term datasets for veterinary researchers and government agencies

---

## 9. Competitive Position

| | Existing pig farm apps | Swine Seeker |
|---|---|---|
| Model | Record-keeping (farmer types data in) | Automatic sensor monitoring |
| Data source | Manual entry | Continuous, hands-free |
| Detection | After symptoms are noticed | Before symptoms are visible |
| Target | Commercial farms, developed markets | Smallholder/medium farms in Laos |
| Cost | Subscription software | Low-cost hardware + service |

Existing livestock monitoring systems are collars or research equipment priced for large farms in developed countries. Swine Seeker is built around off-the-shelf components specifically for the Lao smallholder context.

---

## 10. Business Model

**Value created:** finding one sick pig early is cheaper than losing the herd.

**Revenue streams:**
- Hardware sales or rental (per animal or per set)
- Monthly/annual subscription for dashboard and alert service
- Installation, maintenance, and farmer training
- Partnerships with government agencies, NGOs, or disease surveillance programs

**Target market:** medium and large pig farms across Asia-Pacific, particularly ASF-affected countries — Laos, Vietnam, China, Philippines, Cambodia.

---

## 11. Dashboard Design Direction

Built for farmers first, professional enough to sell:

- **Traffic-light status** (green / amber / red) instead of charts — readable in 3 seconds
- **Pen map** — every pig is a numbered tile colored by status, grouped by pen, so a farmer can walk straight to the animal
- **Urgent alert panel** — shows the flagged pig with all three vitals and two actions: call the vet, or isolate
- **3-day trend view** — shows temperature climbing green → amber → red, making the early-warning story visible
- **Plain language** — "ອຸນຫະພູມສູງ" not "38.4°C anomaly detected"

Deliberately avoided: gradient hero banners, dense analytics dashboards, jargon like "insights" or "real-time telemetry," and multi-tab navigation — all of which assume a desk-bound user comfortable with software.

---

## 12. Safety and Honesty Positioning

- **Cheap where it counts** — four small sensors and one microcontroller per animal, on hardware that already exists
- **Safe by design** — every group is separated, and the tags are trusted with the least of all
- **Honest about limits** — it points at an animal. It does not diagnose, and the vet still decides.

---

## Key References

1. FAO — ASF Situation in Asia & Pacific: https://www.fao.org/animal-health/situation-updates/asf-in-asia-pacific/en
2. WOAH — African Swine Fever: https://www.woah.org/en/disease/african-swine-fever/
3. WOAH — ASF vaccine standard adopted (2025): https://www.woah.org/en/article/african-swine-fever-woah-vaccine-standard-adopted/
4. Matsumoto et al. — Retrospective investigation of the 2019 ASF epidemic, Oudomxay province, Lao PDR: https://www.frontiersin.org/journals/veterinary-science/articles/10.3389/fvets.2023.1277660/full
5. Hui et al. — Spatiotemporal Drivers of the ASF Epidemic in Lao PDR: https://pmc.ncbi.nlm.nih.gov/articles/PMC12016982/
6. Laotian Times — ASF Detected in Xaysomboun (2025): https://laotiantimes.com/2025/02/25/african-swine-fever-detected-in-xaysomboun-control-measures-enforced/
7. KPL — ASF Threatens Pig Farming in Laos: https://kpl.gov.la/En/detail.aspx?id=90696
8. Diep et al. — Safety and Efficacy of AVAC ASF LIVE vaccine: https://onlinelibrary.wiley.com/doi/10.1155/tbed/8623876
9. PMC — Knowledge, Attitudes, and Biosecurity Practices Among Small-Scale Pig Farmers in Lao PDR and Cambodia: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12846685/
10. Morelle et al. — Accelerometer-based detection of ASF infection in wild boar, *Proc. R. Soc. B* 290(2005), 2023
11. Layton et al. — Non-AI preliminary algorithm for prediction and detection of highly pathogenic ASF using health monitoring collars, *Animal Welfare* 35, 2026

---

*Document prepared as a project overview for the Swine Seeker pitch competition.*
