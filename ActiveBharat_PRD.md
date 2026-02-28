# ACTIVEBHARAT — Project Eklavya
## Production-Grade Product Requirements Document & Technical Architecture

**Version:** 1.0
**Classification:** Internal — Engineering & Product Leadership
**Date:** 2026-02-28
**Authors:** Principal SportsTech Architecture Team

---

# Table of Contents

1. [Industry Benchmark Analysis](#1-industry-benchmark-analysis)
2. [Athletic Metric Architecture](#2-athletic-metric-architecture)
3. [Technical Architecture](#3-technical-architecture)
4. [India-Specific Operational Realities](#4-india-specific-operational-realities)
5. [Periodization Engine](#5-periodization-engine)
6. [Nutrition Intelligence Layer](#6-nutrition-intelligence-layer)
7. [Governance & Ethical Safeguards](#7-governance--ethical-safeguards)
8. [Failure Mode Analysis](#8-failure-mode-analysis)
9. [5-Year Roadmap](#9-5-year-roadmap)

---

# 1. Industry Benchmark Analysis

## 1.1 WHOOP (Recovery Model)

**Data Collected:** Continuous PPG-based heart rate, HRV (RMSSD), skin temperature, blood oxygen (SpO2), 3-axis accelerometer for sleep staging and strain quantification.

**Metric Framework:** WHOOP's core innovation is its three-pillar model: **Strain** (cardiovascular load on a 0–21 scale derived from cumulative HR duration in zones), **Recovery** (0–100% composite of HRV, resting HR, respiratory rate, sleep performance), and **Sleep** (time-in-stage analysis mapped to sleep need prediction using prior-day strain).

**Blind Spot:** WHOOP is entirely physiology-driven — it has zero kinematic awareness. It cannot distinguish between a 5km run with poor form causing tibial stress and one with efficient biomechanics. It also requires a $30/month subscription with a dedicated wearable, making it inaccessible to 99% of Indian athletes. No sport-specific periodization — it treats a marathon runner and a javelin thrower identically.

**Technical Infrastructure:** Proprietary ASIC sensor, BLE streaming to mobile, cloud-based HRV trend modeling, cohort-level population analytics.

**Lessons for ActiveBharat:**
- **Adopt:** The Recovery Score concept — HRV-driven readiness is the gold standard for autoregulated training.
- **Reject:** The wearable-dependent model. ActiveBharat must derive HRV from smartphone rPPG (Remote Photoplethysmography) and camera-based vitals to eliminate hardware barriers.

---

## 1.2 Oura (HRV & Sleep Modeling)

**Data Collected:** Infrared PPG (finger-level, higher fidelity than wrist), skin temperature delta, 3D accelerometer, gyroscope. Nocturnal HRV is their key differentiator — measured during deep sleep windows for reduced sympathetic noise.

**Metric Framework:** Readiness Score (composite of HRV balance, body temperature deviation, resting HR trend, previous day's activity, sleep balance). Sleep Score (total sleep time, efficiency, latency, REM/deep proportions). Activity Score (daily movement goals with inactivity alerts).

**Blind Spot:** Oura is a passive monitoring tool — it does not prescribe training or provide exercise programming. It cannot assess movement quality. The ring form factor is culturally uncommon in rural India and the $300+ price point is prohibitive.

**Technical Infrastructure:** Titanium ring with Nordic nRF52 SoC, BLE 5.0, 7-day battery, cloud sync with trend detection algorithms.

**Lessons for ActiveBharat:**
- **Adopt:** Nocturnal HRV measurement philosophy. If we can capture even 2-minute resting HRV via phone camera upon waking, we approximate Oura's highest-signal data point.
- **Reject:** Hardware dependency. Ring-based sensing is irrelevant to our target demographic.

---

## 1.3 Strava (Community & Data Graph)

**Data Collected:** GPS traces, elevation, pace, cadence (with external sensors), heart rate (with paired devices), power (cycling). Route data is the primary asset.

**Metric Framework:** Relative Effort (HR-based training load using Banister's TRIMP model variant), Fitness/Freshness curves (exponentially weighted moving averages of training load), segment-based leaderboards, local legends.

**Blind Spot:** Strava is GPS-centric and fails indoors. It has no biomechanical analysis, no periodization awareness, and no scouting utility. The social layer can discourage novice athletes through unfavorable comparisons. Segment leaderboards are easily gamed with e-bikes and GPS drift.

**Technical Infrastructure:** Massive PostgreSQL + TimescaleDB backend, MapBox integration, FIT/TCX/GPX file parsing pipeline, OAuth-based ecosystem integrations.

**Lessons for ActiveBharat:**
- **Adopt:** The social graph + local leaderboard concept, but scoped to Block/District/State hierarchy instead of arbitrary GPS segments. The "Local Legend" mechanic is powerful for engagement.
- **Reject:** GPS-only tracking. Most Indian athletic training (cricket nets, kabaddi, wrestling) happens in confined spaces where GPS is meaningless.

---

## 1.4 Catapult Sports (Professional Athlete Monitoring)

**Data Collected:** 10Hz/18Hz GPS, tri-axial accelerometer (1000Hz for impact detection), gyroscope, magnetometer. Derives: PlayerLoad™ (triaxial accelerometer vector magnitude), high-speed running distance (>5.5 m/s), acceleration/deceleration counts, mechanical load.

**Metric Framework:** PlayerLoad™ is their proprietary accumulated accelerometer metric. Acute:Chronic Workload Ratio (ACWR) monitoring using rolling 7-day and 28-day exponentially weighted moving averages. Injury risk flags when ACWR > 1.5 (Gabbett's "danger zone").

**Blind Spot:** Enterprise-only ($50K+ per team annually). Requires dedicated hardware vests. No individual athlete self-service. No nutritional integration. Data interpretation requires a sports scientist.

**Technical Infrastructure:** Custom IMU hardware, base station receivers, OpenField cloud platform, API integrations with club medical systems.

**Lessons for ActiveBharat:**
- **Adopt:** ACWR modeling is essential for injury prevention. The Acute:Chronic ratio concept must be implemented using phone-derived accelerometer data and self-reported RPE (Rate of Perceived Exertion).
- **Reject:** The hardware and price model entirely. But the analytical framework is directly transferable.

---

## 1.5 Hudl (Video Analytics)

**Data Collected:** Game video (multiple angles), tagged plays, coach annotations. AI-driven automatic tagging for some sports (American football down/distance, basketball shot tracking).

**Metric Framework:** Play-level breakdown, formation recognition, tendency analysis. Athlete performance is contextualized within team tactical frameworks.

**Blind Spot:** Primarily team-sport focused and US/European market. No individual biomechanical analysis. Requires high-quality multi-camera setups. Zero presence in Indian sports. Processing requires cloud upload of large video files.

**Technical Infrastructure:** AWS-based video processing pipeline, computer vision models for sport-specific event detection, coach-facing web dashboard.

**Lessons for ActiveBharat:**
- **Adopt:** The concept of video-verified performance metrics. For scouting integrity, video evidence must accompany top-tier Bio-Passport entries.
- **Reject:** Multi-camera dependency. ActiveBharat must work with a single smartphone camera at variable quality levels (720p–1080p, 30–60fps).

---

## 1.6 Khelo India Ecosystem

**Data Collected:** Manual performance testing results during Khelo India Games (sprint times, jump distances, endurance test scores). Paper-based or basic digital entry. SAI centre reports.

**Metric Framework:** Event-specific performance thresholds for talent identification. Age-group normative tables. Selection is largely event-result driven with limited longitudinal tracking.

**Blind Spot:** Massive — no continuous monitoring between Games. Athletes are only measured during competition events, missing 99% of their training data. No biomechanical assessment. No digital infrastructure connecting grassroots → district → state → national levels. Data is siloed in individual SAI centres with no unified schema.

**Technical Infrastructure:** Fragmented. Mix of Excel sheets, basic web portals, and paper records. No API layer. No standardized data format across centres.

**Lessons for ActiveBharat:**
- **Adopt:** The hierarchical geographic structure (Block → District → State → National) is the correct organizational model.
- **ActiveBharat IS the missing digital layer.** The entire value proposition is providing the continuous data infrastructure that Khelo India lacks between annual events.

---

## 1.7 Competitive Positioning Matrix

```
┌──────────────────┬───────────┬──────────┬──────────┬───────────┬──────────┐
│     Capability   │  WHOOP    │  Strava  │ Catapult │ Khelo IN  │ Active   │
│                  │           │          │          │           │ Bharat   │
├──────────────────┼───────────┼──────────┼──────────┼───────────┼──────────┤
│ HRV Recovery     │  ■■■■■    │  ■□□□□   │  ■■■□□   │  □□□□□    │  ■■■■□  │
│ Kinematics       │  □□□□□    │  □□□□□   │  ■■□□□   │  □□□□□    │  ■■■■□  │
│ Scouting Pipeline│  □□□□□    │  □□□□□   │  ■■□□□   │  ■■□□□    │  ■■■■■  │
│ India Access     │  □□□□□    │  ■■□□□   │  □□□□□   │  ■■■□□    │  ■■■■■  │
│ Offline Support  │  ■■□□□    │  ■□□□□   │  ■■■□□   │  N/A      │  ■■■■■  │
│ No Hardware Req  │  □□□□□    │  ■■■□□   │  □□□□□   │  ■■■■■    │  ■■■■■  │
│ Cost (User)      │  $30/mo   │  Free/$  │  $50K/yr │  Free     │  Free   │
└──────────────────┴───────────┴──────────┴──────────┴───────────┴──────────┘
```

---

# 2. Athletic Metric Architecture

## 2.1 Tier 1 — Daily Activity Layer

### NEAT Modeling (Non-Exercise Activity Thermogenesis)

NEAT represents 15–50% of total daily energy expenditure and is the strongest predictor of metabolic health in sedentary populations. ActiveBharat estimates NEAT using phone accelerometer step-count data combined with a contextual multiplier.

**Formula:**
```
NEAT_kcal = (steps × 0.04) + (standing_minutes × 0.5) + (fidget_index × BW_kg × 0.001)
```

Where `fidget_index` is derived from accelerometer micro-movement variance during non-walking periods. Threshold: >300 NEAT kcal/day indicates adequate non-exercise movement.

### MET Scoring & Training Load

Metabolic Equivalent of Task (MET) values are used to normalize activity across modalities. 1 MET = 3.5 mL O₂/kg/min (resting metabolic rate).

**Training Load Calculation:**
```
Session_Load = MET_value × duration_hours × body_weight_kg
Daily_Load = Σ(Session_Load) for all sessions

ACWR = Acute_Load(7d_avg) / Chronic_Load(28d_EWMA)
```

**Acute:Chronic Workload Ratio (ACWR) Thresholds:**
- < 0.8: Undertrained (detraining risk)
- 0.8–1.3: Sweet spot (optimal adaptation zone)
- 1.3–1.5: Caution (elevated injury risk, ~2x baseline)
- > 1.5: Danger zone (~4x injury risk per Gabbett et al., 2016)

**MET Reference Table (India-Specific Activities):**

| Activity | MET Value |
|---|---|
| Walking (4 km/h, flat terrain) | 3.0 |
| Cricket batting (net practice) | 5.0 |
| Kabaddi (match intensity) | 9.5 |
| Wrestling (kushti, practice) | 8.0 |
| Sprint training (interval) | 14.0 |
| Yoga (Surya Namaskar flow) | 4.0 |
| Field hockey (match) | 8.5 |
| Cycling (rural commute, 15km/h) | 6.0 |

### Energy Availability (EA) Modeling

EA is the energy remaining for physiological function after exercise expenditure is subtracted. Critical for female athlete health (RED-S prevention).

```
EA = (Energy_Intake_kcal − Exercise_Energy_Expenditure_kcal) / Fat_Free_Mass_kg

Thresholds:
  > 45 kcal/kg FFM/day → Optimal
  30–45              → Adequate
  < 30               → LOW — RED-S risk flag (mandatory alert)
  < 25               → CRITICAL — suspend high-intensity training
```

**Measurement constraint:** EA requires dietary intake logging. Given the nutritional data gap (Section 6), EA will initially be estimated using simplified intake questionnaires with regional food templates, not precise calorie counting.

---

## 2.2 Tier 2 — Bio-Physiological Layer

### HRV Modeling

HRV is the gold-standard non-invasive marker of autonomic nervous system status. ActiveBharat uses two primary time-domain metrics:

**RMSSD (Root Mean Square of Successive Differences):**
```
RMSSD = √(Σ(RR_i+1 − RR_i)² / (N−1))
```
- Reflects parasympathetic (vagal) tone
- Higher RMSSD = better recovery status
- Normal range: 20–100ms (highly individual; must be baselined per athlete over 14+ days)
- Sampling: Minimum 60-second recording, ideally upon waking before standing

**SDNN (Standard Deviation of NN intervals):**
```
SDNN = √(Σ(RR_i − RR_mean)² / (N−1))
```
- Reflects total autonomic variability (sympathetic + parasympathetic)
- Useful for longer recordings (5-min or overnight)

**rPPG Measurement Protocol:**
ActiveBharat derives pulse rate variability using the smartphone front camera (Remote Photoplethysmography). The user places their fingertip over the camera+flash for 90 seconds upon waking.

**Accuracy constraints:**
- Camera sampling rate must be ≥30fps (most phones achieve this)
- Ambient light variability introduces noise — the flash LED provides consistent illumination through the fingertip
- Expected accuracy: ±5ms RR-interval error vs. ECG reference (acceptable for trend tracking, not clinical diagnosis)
- Skin pigmentation affects signal-to-noise ratio — darker skin requires longer flash exposure and adaptive gain control

### Readiness Score Algorithm

```
Readiness_Score = (
    w1 × HRV_deviation_from_baseline +
    w2 × RHR_deviation +
    w3 × Sleep_score +
    w4 × Subjective_wellness +
    w5 × ACWR_zone_penalty
) × 100

Where:
  w1 = 0.30 (HRV is primary driver)
  w2 = 0.15 (Resting HR)
  w3 = 0.25 (Sleep quality/duration)
  w4 = 0.15 (Self-reported: muscle soreness, mood, energy — 1-5 scale)
  w5 = 0.15 (ACWR penalty: 0 if in sweet spot, -0.5 if danger zone)

HRV_deviation = (today_RMSSD − 7d_rolling_mean) / 7d_rolling_SD
  → Positive = better than baseline → boosts readiness
  → Negative = worse than baseline → suppresses readiness
```

**Thresholds:**
- 80–100: Green — full training capacity, Olympic Module active
- 60–79: Amber — moderate volume, reduce intensity by 20%
- 40–59: Yellow — active recovery, mobility focus
- < 40: Red — mandatory rest day, flag if persists >3 days

### Sleep Stage Reliability on Mobile

**Critical limitation:** Smartphone accelerometer-based sleep staging (used when no wearable is present) has poor accuracy for REM vs. Light sleep differentiation. Studies show ~65% epoch-by-epoch agreement with PSG for consumer devices (de Zambotti et al., 2019).

**ActiveBharat approach:**
- Do NOT claim clinical sleep staging accuracy
- Report only: **Total Sleep Time**, **Sleep Efficiency** (time asleep / time in bed), **Restfulness** (movement-derived fragmentation index)
- Use self-reported sleep quality (1–5) as a complementary signal
- Reserve detailed sleep architecture analysis for athletes with connected wearables (Oura, Fitbit, Garmin via API)

### VO2 Max Estimation

True VO2 max requires lab testing (expired gas analysis). ActiveBharat estimates it via submaximal field tests:

**Cooper 12-Minute Run Test:**
```
VO2max_est = (distance_meters − 504.9) / 44.73
```

**20m Shuttle Run (Beep Test) — Léger equation:**
```
VO2max_est = 31.025 + (3.238 × speed_km/h) − (3.248 × age) + (0.1536 × speed × age)
```

**Resting HR Estimation (Uth et al., 2004) — least accurate but zero-effort:**
```
VO2max_est = 15.3 × (HR_max / HR_rest)
  where HR_max = 208 − (0.7 × age)
```

**Accuracy hierarchy:** Lab > Beep Test (±3.5 mL/kg/min) > Cooper (±5) > Resting estimate (±7)

ActiveBharat uses the resting estimate as a baseline on Day 1, then calibrates with field tests during the onboarding assessment phase.

---

## 2.3 Tier 3 — Kinematic & Performance Layer

### Sprint Mechanics Analysis

**Required frame rate:** Minimum 60fps for meaningful sprint biomechanics. At 30fps, each frame spans 33ms — a ground contact phase during max-velocity sprinting is ~80–100ms, giving only 2–3 frames of stance phase. This is insufficient to measure:
- Shin angle at touchdown (optimal: 90–95° from horizontal)
- Hip extension angle at toe-off
- Flight-to-contact ratio

**ActiveBharat approach:** Hybrid processing model.
```
┌──────────────────────────────────────────────────────────┐
│                    SPRINT ANALYSIS PIPELINE              │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Phone Camera (60fps)                                    │
│       │                                                  │
│       ▼                                                  │
│  LOCAL (MediaPipe BlazePose)         ASYNC CLOUD         │
│  ┌─────────────────────┐     ┌─────────────────────┐    │
│  │ 33-keypoint skeleton│     │ High-fidelity        │    │
│  │ Basic joint angles  │────▶│ biomechanical model  │    │
│  │ Stride count        │     │ Ground contact time  │    │
│  │ Instant feedback    │     │ Force vector estimate│    │
│  │ ~15ms latency       │     │ Gait asymmetry index │    │
│  └─────────────────────┘     │ ~30s processing time │    │
│                               └─────────────────────┘    │
│                                        │                  │
│                                        ▼                  │
│                               Detailed report pushed      │
│                               to athlete within 5 min     │
└──────────────────────────────────────────────────────────┘
```

### Joint Angle Precision Tolerance

| Measurement | Required Precision | MediaPipe Achievable | Gap |
|---|---|---|---|
| Knee flexion angle | ±3° | ±5–8° | Marginal |
| Hip extension | ±3° | ±6–10° | Significant |
| Ankle dorsiflexion | ±2° | ±8–12° | Too large for clinical |
| Shoulder rotation | ±5° | ±7–10° | Acceptable for screening |

**Mitigation:** ActiveBharat does NOT claim clinical-grade joint angle measurement. Instead, it uses **relative change tracking** — if an athlete's knee flexion angle at foot strike degrades by >10° over 4 weeks, this is flagged regardless of absolute accuracy. Trend detection is more robust than absolute measurement.

### Ground Reaction Force Approximation Without Force Plates

True GRF measurement requires lab-grade force plates ($10K+). ActiveBharat estimates vertical GRF during jumping using the impulse-momentum theorem:

```
Peak_GRF_estimate = body_mass × (jump_height × 2 × g)^0.5 / ground_contact_time

Where:
  jump_height = (flight_time² × g) / 8    [derived from video frame counting]
  g = 9.81 m/s²
  ground_contact_time = frames_in_contact / fps
```

**Accuracy:** ±15–20% vs. force plate. Acceptable for longitudinal tracking and population-level comparisons, NOT for research publication.

### Device Heterogeneity Challenge

The Indian smartphone market spans:
- **High-end (5%):** iPhone 13+, Samsung S-series — 60fps camera, A-series/Snapdragon 8-gen chips, reliable IMU
- **Mid-range (30%):** Redmi Note series, Realme, Samsung M-series — 30fps default (60fps burst mode available), Snapdragon 6xx/MediaTek Dimensity
- **Low-end (65%):** Sub-₹10K devices — 30fps only, MediaTek Helio G-series, limited RAM (3–4GB), thermal throttling during sustained camera use

**ActiveBharat tiered capability:**
```
Device Tier    │ Kinematic Mode        │ Model Size   │ Processing
───────────────┼───────────────────────┼──────────────┼──────────────
High           │ Full 33-point pose    │ 8MB          │ Real-time
Mid            │ 17-point lite pose    │ 3MB          │ Near-real-time
Low            │ Manual timing only    │ No CV model  │ Timer/stopwatch
               │ (video upload for     │              │ + async cloud
               │  async processing)    │              │
```

---

## 2.4 Tier 4 — Scouting & Bio-Passport Index (BPI)

### BPI Formula

The Bio-Passport Index is a composite longitudinal score designed to identify genuinely talented, consistently developing athletes — not one-time peak performers.

```
BPI = (
    α × Consistency_Score +
    β × Improvement_Slope +
    γ × Relative_Percentile +
    δ × Assessment_Confidence
) × Age_Correction_Factor × Verification_Multiplier

Where:
  α = 0.30  (Training consistency — most important for scouting)
  β = 0.30  (Rate of improvement)
  γ = 0.25  (Current performance relative to age-group peers)
  δ = 0.15  (Data quality / verification level)

  BPI range: 0–1000
```

**Component Definitions:**

**Consistency_Score (0–100):**
```
Consistency = (days_trained_in_period / expected_training_days) × 100
  adjusted for:
    - illness/injury deductions (verified medical flag)
    - seasonal training load variation (monsoon adjustment)

  Threshold: >80% = Elite consistency, <50% = Flag for dropout risk
```

**Improvement_Slope (0–100):**
```
Slope = linear_regression_slope(performance_metric, time) / expected_development_rate

  Expected development rates are age-group and sport-specific:
    e.g., 100m sprint improvement for males 14-16: ~0.3s/year
    e.g., Long jump improvement for females 13-15: ~15cm/year

  Slope > 1.0 → above-average development rate
  Slope > 2.0 → exceptional (top 2% flag for scout review)
```

**Relative_Percentile (0–100):**
```
Percentile within: same sport × same age-group × same region
  Calculated using z-score against regional normative database
  Scaled to 0–100 percentile rank
```

**Age_Correction_Factor:**
```
ACF accounts for biological maturation timing:
  Early maturer (PHV age < population_mean − 1SD): ACF = 0.90
  On-time maturer: ACF = 1.00
  Late maturer (PHV age > population_mean + 1SD): ACF = 1.10

  PHV = Peak Height Velocity, estimated via Mirwald equation:
  Male: Maturity_offset = −9.236 + (0.0002708 × leg_length × sitting_height) + ...
  Female: Maturity_offset = −9.376 + (0.0001882 × leg_length × sitting_height) + ...
```

This correction **boosts** late maturers who may be overlooked by conventional scouting that favors early physical developers — a well-documented bias in youth sport talent identification (Cobley et al., 2009).

**Verification_Multiplier:**
```
Level 0 (self-reported data only):        × 0.60
Level 1 (app-recorded, no video):         × 0.75
Level 2 (video-recorded, AI-verified):    × 0.90
Level 3 (video + coach-attested):         × 0.95
Level 4 (official competition result):    × 1.00
```

### Anti-Manipulation Safeguards

**Problem:** Athletes or coaches could fabricate training data to inflate BPI scores.

**Detection Methods:**

1. **Statistical anomaly detection:** If an athlete's improvement slope exceeds 3 SD above population mean without corresponding training volume increase → automatic flag
2. **Sensor fingerprinting:** Accelerometer/gyroscope data has device-specific noise signatures. If training sessions show data from multiple devices without account migration → flag
3. **Video liveness detection:** For Level 2+ verification:
   - Temporal consistency check (frames must show continuous motion, not spliced)
   - Background scene consistency (detect if video is played on a screen by checking for moiré patterns and refresh-rate artifacts)
   - Random challenge overlay: app displays a random number/color during recording that must be visible in frame
4. **Geographic consistency:** Training GPS data must be plausible (not teleporting between cities within hours)
5. **Physiological plausibility:** Heart rate data during claimed "sprint sessions" must show appropriate HR elevation. A claimed 11.0s 100m sprint with HR staying at 80bpm → flag

---

# 3. Technical Architecture

## 3.1 System Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐  │
│  │  Android App  │  │   iOS App    │  │    PWA (Low-end fallback)│  │
│  │  (Kotlin)     │  │   (Swift)    │  │    (React + Workbox)     │  │
│  └──────┬───────┘  └──────┬───────┘  └────────────┬─────────────┘  │
│         │                  │                       │                 │
│  ┌──────┴──────────────────┴───────────────────────┴──────────┐     │
│  │                    OFFLINE ENGINE                           │     │
│  │  IndexedDB (metrics) + SQLite (athlete profile) + OPFS     │     │
│  │  Delta-sync queue │ Conflict resolution (LWW + vector clk) │     │
│  └──────────────────────────┬────────────────────────────────┘     │
└─────────────────────────────┼───────────────────────────────────────┘
                              │ HTTPS / gRPC (when online)
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        API GATEWAY                                  │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  Kong / AWS API Gateway                                       │  │
│  │  Rate limiting: 100 req/min per user │ JWT auth │ WAF         │  │
│  └───────────────────────────────┬───────────────────────────────┘  │
│                                  │                                   │
│  ┌───────────┬───────────┬───────┴────┬──────────────┬───────────┐  │
│  │  Auth     │ Athlete   │ Training   │ Analytics    │ Scouting  │  │
│  │  Service  │ Profile   │ Engine     │ Service      │ Service   │  │
│  │  (Cognito)│ Service   │ Service    │ (Flink)      │           │  │
│  └───────────┴─────┬─────┴─────┬──────┴──────┬───────┴───────────┘  │
│                    │           │              │                       │
│  ┌─────────────────┴───────────┴──────────────┴───────────────────┐  │
│  │                    DATA LAYER                                   │  │
│  │  ┌────────────┐  ┌──────────────┐  ┌────────────────────────┐  │  │
│  │  │ PostgreSQL │  │ TimescaleDB  │  │ S3 / CloudFront        │  │  │
│  │  │ (Profiles, │  │ (Time-series │  │ (Video storage,        │  │  │
│  │  │  Auth,     │  │  metrics,    │  │  model artifacts,      │  │  │
│  │  │  Scouting) │  │  HRV, accel) │  │  food image dataset)   │  │  │
│  │  └────────────┘  └──────────────┘  └────────────────────────┘  │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │                    AI / ML LAYER                                │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐ │  │
│  │  │ Video        │  │ Anomaly      │  │ Periodization        │ │  │
│  │  │ Verification │  │ Detection    │  │ Recommendation       │ │  │
│  │  │ Queue (SQS)  │  │ (Isolation   │  │ Engine               │ │  │
│  │  │ + Lambda     │  │  Forest)     │  │ (Rule-based + ML)    │ │  │
│  │  └──────────────┘  └──────────────┘  └──────────────────────┘ │  │
│  └────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

## 3.2 Database Schema

### PostgreSQL — Core Relational Data

```sql
-- Athlete Profile (core identity)
CREATE TABLE athletes (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    phone_hash      VARCHAR(64) UNIQUE NOT NULL,  -- SHA-256 of phone number
    display_name    VARCHAR(100) NOT NULL,
    date_of_birth   DATE NOT NULL,
    gender          VARCHAR(10) CHECK (gender IN ('male', 'female', 'other')),
    height_cm       DECIMAL(5,1),
    weight_kg       DECIMAL(5,1),
    state_code      CHAR(2) NOT NULL,             -- IN-RJ, IN-MH, etc.
    district_id     INTEGER REFERENCES districts(id),
    block_id        INTEGER REFERENCES blocks(id),
    primary_sport   VARCHAR(50),
    equipment_tier  SMALLINT CHECK (equipment_tier BETWEEN 0 AND 3),
    onboarding_complete BOOLEAN DEFAULT FALSE,
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    updated_at      TIMESTAMPTZ DEFAULT NOW()
);

-- Geographic Hierarchy (for multi-tenant scouting)
CREATE TABLE states (
    id          SERIAL PRIMARY KEY,
    code        CHAR(2) UNIQUE NOT NULL,
    name_en     VARCHAR(50) NOT NULL,
    name_local  VARCHAR(100)
);

CREATE TABLE districts (
    id          SERIAL PRIMARY KEY,
    state_id    INTEGER REFERENCES states(id),
    name_en     VARCHAR(100) NOT NULL,
    name_local  VARCHAR(150),
    tier        SMALLINT CHECK (tier BETWEEN 1 AND 3)  -- Tier-1/2/3 classification
);

CREATE TABLE blocks (
    id          SERIAL PRIMARY KEY,
    district_id INTEGER REFERENCES districts(id),
    name_en     VARCHAR(100) NOT NULL,
    name_local  VARCHAR(150)
);

-- Bio-Passport Index (scouting record)
CREATE TABLE bio_passport (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    athlete_id      UUID REFERENCES athletes(id) ON DELETE CASCADE,
    sport           VARCHAR(50) NOT NULL,
    bpi_score       DECIMAL(6,1) NOT NULL,          -- 0-1000
    consistency     DECIMAL(5,1),
    improvement     DECIMAL(5,1),
    percentile      DECIMAL(5,1),
    confidence      DECIMAL(3,2),
    age_correction  DECIMAL(3,2),
    verification    SMALLINT CHECK (verification BETWEEN 0 AND 4),
    computed_at     TIMESTAMPTZ DEFAULT NOW(),
    valid_until     TIMESTAMPTZ,
    UNIQUE(athlete_id, sport, computed_at)
);
CREATE INDEX idx_bpi_scouting ON bio_passport(sport, bpi_score DESC, verification);

-- Training Sessions
CREATE TABLE training_sessions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    athlete_id      UUID REFERENCES athletes(id),
    session_type    VARCHAR(30) NOT NULL,  -- 'sprint', 'endurance', 'strength', etc.
    sport           VARCHAR(50),
    started_at      TIMESTAMPTZ NOT NULL,
    ended_at        TIMESTAMPTZ,
    duration_min    SMALLINT,
    rpe             SMALLINT CHECK (rpe BETWEEN 1 AND 10),  -- Rate of Perceived Exertion
    training_load   DECIMAL(8,1),     -- MET × duration × weight
    acwr_at_time    DECIMAL(4,2),
    video_ref       VARCHAR(255),      -- S3 key for verification video
    verification    SMALLINT DEFAULT 0,
    device_fingerprint VARCHAR(64),
    gps_lat         DECIMAL(9,6),
    gps_lon         DECIMAL(9,6),
    created_at      TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_sessions_athlete_time ON training_sessions(athlete_id, started_at DESC);
```

### TimescaleDB — Time-Series Metrics

```sql
-- High-frequency biometric data (hypertable)
CREATE TABLE biometrics (
    athlete_id      UUID NOT NULL,
    ts              TIMESTAMPTZ NOT NULL,
    metric_type     VARCHAR(20) NOT NULL,  -- 'rmssd', 'rhr', 'spo2', 'hrv_sdnn'
    value           DECIMAL(10,3) NOT NULL,
    source          VARCHAR(20),           -- 'rppg', 'wearable_oura', 'manual'
    quality_score   DECIMAL(3,2),          -- 0.00-1.00 signal quality
    PRIMARY KEY (athlete_id, ts, metric_type)
);
SELECT create_hypertable('biometrics', 'ts', chunk_time_interval => INTERVAL '7 days');

-- Continuous aggregate for daily summaries (auto-materialized)
CREATE MATERIALIZED VIEW daily_biometrics
WITH (timescaledb.continuous) AS
SELECT
    athlete_id,
    time_bucket('1 day', ts) AS day,
    metric_type,
    AVG(value) AS avg_value,
    MIN(value) AS min_value,
    MAX(value) AS max_value,
    COUNT(*) AS sample_count
FROM biometrics
GROUP BY athlete_id, time_bucket('1 day', ts), metric_type;

-- Kinematic snapshots (post-processed, not raw frames)
CREATE TABLE kinematic_results (
    id              UUID DEFAULT gen_random_uuid(),
    session_id      UUID NOT NULL,
    athlete_id      UUID NOT NULL,
    ts              TIMESTAMPTZ NOT NULL,
    metric          VARCHAR(30),  -- 'knee_angle_strike', 'hip_ext_toe_off', 'stride_length'
    value           DECIMAL(8,3),
    unit            VARCHAR(10),  -- 'degrees', 'meters', 'seconds', 'N'
    confidence      DECIMAL(3,2),
    PRIMARY KEY (athlete_id, ts, metric)
);
SELECT create_hypertable('kinematic_results', 'ts');
```

### Data Retention Policy

| Data Type | Hot Storage | Warm (Compressed) | Cold (Archive) |
|---|---|---|---|
| Raw biometrics | 90 days | 1 year | 5 years (S3 Glacier) |
| Daily aggregates | Forever | — | — |
| Session summaries | Forever | — | — |
| Video (verification) | 30 days | 6 months | Delete |
| BPI snapshots | Forever | — | — |
| Kinematic results | 1 year | 5 years | Delete |

## 3.3 Offline-First Architecture

### Delta-Sync Strategy

```
┌─────────────────────────────────────────────────┐
│              SYNC STATE MACHINE                  │
│                                                  │
│  OFFLINE ──┬── Queue mutations in IndexedDB     │
│            │   (max queue: 500 operations)       │
│            │                                     │
│  ONLINE ───┤── 1. Push queued mutations          │
│   detected │   2. Pull server changes since      │
│            │      last_sync_timestamp             │
│            │   3. Resolve conflicts (LWW)        │
│            │   4. Update last_sync_timestamp      │
│            │                                     │
│  CONFLICT ─┴── Last-Writer-Wins for metrics      │
│                Server-wins for BPI/scouting      │
│                Manual resolution for profile     │
└─────────────────────────────────────────────────┘
```

**IndexedDB Schema (Client-Side):**
```javascript
const DB_SCHEMA = {
  athletes: '++id, phone_hash, synced',
  pendingSessions: '++localId, athleteId, startedAt, synced',
  biometrics: '++id, athleteId, ts, metricType, synced',
  syncQueue: '++id, table, operation, payload, createdAt',
  modelCache: 'modelName, version, blob, downloadedAt'
};
```

**Sync priority order:**
1. Biometric readings (small payloads, high value)
2. Session completions (medium payloads)
3. Video uploads (large payloads, background transfer only on Wi-Fi)
4. Profile updates (rare, small)

### Model Compression for Edge Inference

| Model | Full Size | Quantized (INT8) | Pruned + Quantized | Target |
|---|---|---|---|---|
| BlazePose Full | 12.3 MB | 3.8 MB | 2.9 MB | High-end |
| BlazePose Lite | 5.1 MB | 1.9 MB | 1.4 MB | Mid-range |
| rPPG signal extractor | 2.2 MB | 0.8 MB | 0.6 MB | All devices |
| Food image classifier | 18 MB | 6.1 MB | 4.5 MB | High/Mid |

## 3.4 Security & Anti-Spoofing

### Video Verification Pipeline

```
┌──────────────────────────────────────────────────────────┐
│              VIDEO VERIFICATION QUEUE                     │
│                                                          │
│  1. Upload video to S3 (pre-signed URL, max 60s, 720p+) │
│                      │                                    │
│  2. SQS message ─────┤                                   │
│                      ▼                                    │
│  3. Lambda: Frame extraction (1fps for liveness,          │
│             10fps for performance analysis)               │
│                      │                                    │
│  4. Liveness checks: │                                    │
│     ├── Moiré pattern detection (screen playback)        │
│     ├── Temporal coherence (no frame splicing)           │
│     ├── Challenge overlay verification                    │
│     ├── Background consistency (SSIM across frames)      │
│     └── Human presence confidence (pose detection)       │
│                      │                                    │
│  5. Performance extraction:                               │
│     ├── Sprint time (first-movement to finish-line)      │
│     ├── Jump height (flight time method)                 │
│     ├── Form quality score (joint angle analysis)        │
│     └── Rep counting (for strength exercises)            │
│                      │                                    │
│  6. Result: VERIFIED / FLAGGED / REJECTED                │
│     └── Written to training_sessions.verification        │
└──────────────────────────────────────────────────────────┘
```

### Fraud Anomaly Detection

Using Isolation Forest on athlete training data vectors:

```python
# Pseudocode: Anomaly detection for training data integrity
features = [
    'session_frequency_7d',
    'avg_session_duration',
    'improvement_rate_30d',
    'rpe_hr_correlation',      # RPE should correlate with HR
    'device_switch_frequency',
    'gps_variance',
    'time_of_day_entropy',     # Consistent training times = normal
    'video_verification_ratio'
]

model = IsolationForest(contamination=0.05, n_estimators=200)
model.fit(population_training_data)

# Per-athlete scoring
anomaly_score = model.decision_function(athlete_features)
if anomaly_score < threshold:
    flag_for_manual_review(athlete_id)
```

---

# 4. India-Specific Operational Realities

## 4.1 Device Diversity

India has 600M+ smartphones across 200+ models. ActiveBharat must function on devices with:
- **RAM:** 3GB minimum (excludes ~8% of market)
- **Storage:** App + models must fit in <100MB (many users have <2GB free)
- **Camera:** Rear 720p minimum for kinematic analysis
- **Chipset:** Minimum Snapdragon 450 / MediaTek Helio G35 equivalent

**Mitigation:** Progressive capability detection at first launch. The app probes device capabilities and enables/disables features accordingly. No feature should crash on an unsupported device — graceful degradation is mandatory.

## 4.2 Rural Connectivity

Per TRAI data, Tier-3 and rural areas experience:
- Average download speed: 5–12 Mbps (vs. 30+ in Tier-1)
- Frequent disconnections (2G fallback during peak hours)
- Data cost sensitivity: ₹150/month for 1.5GB/day is standard

**Architecture Adjustments:**
- All metric collection must work 100% offline
- Sync payload budget: <2MB per daily sync (compressed)
- Video upload: only on Wi-Fi, with resume capability (chunked multipart upload)
- App assets: aggressive caching, no CDN-dependent rendering
- API responses: Protobuf instead of JSON (30–50% size reduction)

## 4.3 Lighting Issues in Video Capture

Rural training grounds often have:
- No artificial lighting (outdoor morning/evening training)
- Harsh direct sunlight causing overexposure
- Inconsistent shadows affecting pose detection accuracy

**Mitigation:**
- Adaptive exposure guidance (on-screen prompt: "Move so the sun is behind the camera")
- MediaPipe confidence score threshold: reject frames with pose confidence <0.6
- Allow athletes to mark "indoor" vs "outdoor" sessions for model parameter adjustment
- Low-light mode: increase ISO and reduce frame rate to 30fps to maintain exposure

## 4.4 Female Athlete Participation

Cultural barriers include:
- Privacy concerns with video recording in rural settings
- Family resistance to athletic participation post-puberty
- Lack of female coaches in many regions
- Period tracking stigma

**ActiveBharat approach:**
- **Privacy mode:** Video processing can happen entirely on-device with no cloud upload. Results are synced as numeric data only.
- **Female-only leaderboards** and community spaces
- **Menstrual cycle integration** (optional, athlete-controlled): Track cycle phase and correlate with HRV/readiness data. Literature shows luteal phase HRV suppression of 10–15% — the readiness algorithm must account for this to avoid false "low readiness" flags.
- **RED-S screening:** Automated Energy Availability monitoring with sensitive, non-alarmist messaging

## 4.5 Data Privacy Compliance (DPDP Act 2023)

- All athlete data is classified as **personal data** under the Digital Personal Data Protection Act
- Athletes under 18: **Verifiable parental consent** required before data collection
- **Data localization:** All primary storage must be on Indian servers (AWS Mumbai / GCP Mumbai)
- **Right to erasure:** Athletes can request complete data deletion; system must support cascade delete across PostgreSQL, TimescaleDB, S3, and CDN caches within 72 hours
- **No Aadhaar linkage:** Due to Supreme Court privacy rulings, ActiveBharat will NOT integrate with Aadhaar. Identity verification uses phone OTP + optional DigiLocker sports certificate upload.
- **Data fiduciary registration** required before launch

## 4.6 Regional Language UI

India has 22 scheduled languages. ActiveBharat launch plan:

| Phase | Languages | Coverage |
|---|---|---|
| MVP | English, Hindi | ~55% |
| Phase 2 | Tamil, Telugu, Kannada, Malayalam, Bengali, Marathi | ~85% |
| Phase 3 | Gujarati, Punjabi, Odia, Assamese | ~93% |
| Phase 4 | Remaining scheduled languages | ~98% |

**Implementation:** i18n framework with server-managed string bundles (not compiled into the app). Language detection from device locale with manual override. All sports science terminology retains English technical terms with vernacular explanations.

---

# 5. Periodization Engine

## 5.1 Structure

```
MACROCYCLE (16–52 weeks)
  │
  ├── MESOCYCLE 1: General Preparation (4–6 weeks)
  │     ├── Microcycle 1: Anatomical Adaptation (low intensity, high volume)
  │     ├── Microcycle 2: Base Building
  │     └── ...
  │
  ├── MESOCYCLE 2: Specific Preparation (4–6 weeks)
  │     ├── Progressive overload: +5-10% volume/week
  │     └── Sport-specific skill integration
  │
  ├── MESOCYCLE 3: Pre-Competition (2–4 weeks)
  │     ├── Intensity peaks, volume tapers
  │     └── Competition simulation
  │
  ├── MESOCYCLE 4: Competition (1–4 weeks)
  │     └── Maintenance volume, peak performance
  │
  └── MESOCYCLE 5: Transition/Recovery (1–2 weeks)
        └── Active recovery, cross-training
```

## 5.2 Autoregulation Algorithm

```python
def generate_daily_training(athlete):
    readiness = compute_readiness_score(athlete)
    acwr = compute_acwr(athlete)
    phase = get_current_mesocycle_phase(athlete)

    # HRV-triggered autoregulation
    if readiness < 40:
        return RecoverySession(
            type='mobility_and_breathing',
            duration_min=20,
            rpe_target=2
        )

    if readiness < 60:
        return ModifiedSession(
            base=phase.planned_session,
            intensity_modifier=0.75,    # reduce intensity 25%
            volume_modifier=0.80,       # reduce volume 20%
            note='Readiness below threshold. Reduced load today.'
        )

    # ACWR guardrails
    if acwr > 1.5:
        return DeloadSession(
            type='active_recovery',
            duration_min=30,
            rpe_target=3,
            alert='ACWR in danger zone. Mandatory deload.'
        )

    if acwr < 0.8 and phase.allows_load_increase:
        return AugmentedSession(
            base=phase.planned_session,
            volume_modifier=1.15,  # increase volume 15%
            note='Underloaded. Adding volume to maintain adaptation.'
        )

    # Progressive overload logic
    planned = phase.planned_session
    week_in_meso = get_week_in_mesocycle(athlete)

    if week_in_meso <= 3:
        # Standard progressive overload: +5-10% per week
        overload = 1.0 + (0.05 * (week_in_meso - 1))
        planned.volume *= overload
    elif week_in_meso == 4:
        # Deload week: reduce to 60% of week 3 volume
        planned.volume *= 0.60
        planned.intensity *= 0.85

    return planned

def check_deload_triggers(athlete):
    triggers = []

    # 3+ consecutive days of declining HRV
    hrv_trend = get_hrv_trend(athlete, days=3)
    if all(hrv_trend[i] > hrv_trend[i+1] for i in range(len(hrv_trend)-1)):
        triggers.append('hrv_declining_3d')

    # Resting HR elevated >10% above baseline
    rhr = get_latest_rhr(athlete)
    rhr_baseline = get_rhr_baseline(athlete)
    if rhr > rhr_baseline * 1.10:
        triggers.append('rhr_elevated')

    # Self-reported soreness >= 4/5 for 2+ days
    soreness = get_soreness_history(athlete, days=2)
    if all(s >= 4 for s in soreness):
        triggers.append('persistent_soreness')

    # Performance decline >5% in primary metric
    perf_trend = get_performance_trend(athlete, metric='primary', weeks=2)
    if perf_trend.pct_change < -0.05:
        triggers.append('performance_decline')

    if len(triggers) >= 2:
        force_deload(athlete, reason=triggers)
        return True
    return False
```

## 5.3 Growth Spurt & Puberty Edge Cases

For athletes aged 11–17, the periodization engine must account for:
- **Peak Height Velocity (PHV):** During PHV (typically age 12–14 for girls, 14–16 for boys), bone growth outpaces muscular adaptation. High-impact plyometrics should be volume-capped during PHV period.
- **PHV detection:** Monthly height/weight logging. If height increase >0.8cm/month → PHV flag → reduce plyometric volume by 30%, increase mobility work.
- **Relative Age Effect (RAE):** An athlete born in January vs December of the same year can differ by 10–15% in physical capacity. BPI age-correction accounts for this using birth-quarter adjustment.

---

# 6. Nutrition Intelligence Layer

## 6.1 Why Global Food Databases Fail in India

MyFitnessPal, USDA FoodData Central, and similar databases contain <500 entries for Indian regional foods out of 300,000+ total entries. Missing categories include:

- **Regional preparations:** Bajra roti, Jowar bhakri, Ragi mudde, Sattu sharbat, Pakhala bhaat
- **Cooking method variation:** Dal fry vs. dal tadka vs. dal makhani have dramatically different fat content (5g vs. 8g vs. 22g per serving) but are often listed as generic "dal"
- **Street food:** Vada pav, Kachori, Poha, Upma — commonly consumed but uncatalogued with preparation-specific macros
- **Regional variation:** "Sambar" in Tamil Nadu vs. Karnataka uses different base ingredients, yielding 15–25% calorie variation

## 6.2 India Food Dataset Strategy

### Phase 1: Rajasthan Pilot Dataset (Year 1)

**Approach:** Partner with 50 athletes in Jodhpur/Jaipur pilot program. Athletes photograph every meal for 8 weeks using in-app camera.

**Pipeline:**
```
Photo → Dietitian labeling (remote, via dashboard)
  → Portion estimation (plate-size reference card provided)
  → Macro assignment (protein/carb/fat/fiber per serving)
  → Regional food taxonomy tagging
  → Quality review (2nd dietitian cross-validation)

Target: 2,000 unique food items, 10,000 labeled images
Accuracy target: ±15% calorie estimation (acceptable for non-clinical use)
```

### Phase 2: Crowd-Sourced Expansion (Year 2–3)

- Community-contributed food labels reviewed by regional nutrition moderators
- Consensus mechanism: 3 independent labels must agree within 20% before item is accepted into the database
- Gamification: Athletes earn BPI confidence boosts for verified food contributions

### Phase 3: AI-Assisted Estimation (Year 3+)

- Image-based food recognition model trained on India-specific dataset
- Volume estimation using depth sensing (available on mid-range phones via LiDAR or stereo camera)
- Known limitation: Accuracy for mixed dishes (thali, biryani) is inherently lower (~±25%) than single-item recognition (~±12%)

## 6.3 Nutritional Scoring

```
Meal_Score = (
    protein_adequacy × 0.35 +
    energy_sufficiency × 0.25 +
    micronutrient_coverage × 0.25 +
    hydration_index × 0.15
)

Where:
  protein_adequacy = actual_protein / target_protein
    target_protein = 1.6 g/kg/day for strength athletes
                     1.2 g/kg/day for endurance athletes
                     0.8 g/kg/day for general fitness

  energy_sufficiency = actual_kcal / TDEE
    TDEE = BMR × activity_factor
    Mifflin-St Jeor BMR:
      Male: (10 × weight_kg) + (6.25 × height_cm) - (5 × age) + 5
      Female: (10 × weight_kg) + (6.25 × height_cm) - (5 × age) - 161
```

**Low-cost dietary intervention alerts:**
- Iron deficiency risk: If diet score shows <50% RDA iron for 7+ days → suggest jaggery, green leafy vegetables, soaked chana
- Protein gap: If protein_adequacy <0.7 → suggest affordable sources: eggs (₹7 each), soy chunks (₹60/kg), milk, peanuts
- Hydration: If training in >35°C ambient temp and no water logging → push hydration reminder every 20 minutes

---

# 7. Governance & Ethical Safeguards

## 7.1 Data Ownership

**Principle: Athletes own their data. Period.**

- Athletes can export all their data in standard formats (CSV, JSON) at any time
- Athletes can delete their account and all associated data permanently
- No federation, coach, or government body can access raw athlete data without explicit, revocable consent from the athlete (or guardian for minors)
- Consent is granular: an athlete can share BPI scores with a federation without sharing HRV data

## 7.2 Federation Access Model

```
ACCESS TIERS:

Coach (athlete-granted):
  ├── View training sessions
  ├── View readiness scores
  ├── View BPI score
  └── Cannot: export data, share with third parties

District Federation:
  ├── View anonymized aggregate statistics
  ├── View BPI leaderboards (athlete-consented only)
  └── Cannot: view individual biometrics, training details

State Federation:
  ├── View district-level aggregates
  ├── Request identified data for national selection (requires athlete consent)
  └── Cannot: bulk export, monetize data

National (SAI):
  ├── Anonymized population-level analytics
  ├── Talent pipeline metrics (consented athletes only)
  └── Cannot: access without state federation endorsement
```

## 7.3 AI Bias Against Rural Athletes

**Risk:** Models trained on urban athlete data may establish norms that systematically disadvantage rural athletes who have different training modalities, nutrition profiles, and baseline fitness markers.

**Mitigation:**
- Separate normative tables by region tier (Tier-1, 2, 3)
- BPI percentile rankings are calculated within-region first, then nationally
- Training environment tags (gym, outdoor ground, home) influence exercise recommendations — the system must not default to gym-based exercises for athletes without gym access
- Regular bias audits: quarterly analysis of BPI distributions across regions, genders, and socioeconomic proxies (device tier as proxy)

## 7.4 Monetization Ethics

- Athlete data is NEVER sold to third parties
- Revenue model: Government contracts (SAI/state federations), premium coaching tools, sports brand partnerships (aggregate insights only, never individual data)
- No dark patterns: free tier must be fully functional for an individual athlete's training and BPI tracking

---

# 8. Failure Mode Analysis

| # | Failure Mode | Impact | Prevention Strategy | Monitoring Metric |
|---|---|---|---|---|
| 1 | **rPPG HRV measurement gives systematically wrong readings on dark skin tones** | Readiness scores biased, incorrect training load → overtraining injuries | Skin-tone-stratified validation study before launch. Adaptive gain control. Mandatory accuracy threshold: RMSSD error <8ms across all skin tones | Error rate by skin tone bucket (Fitzpatrick scale), monthly audit |
| 2 | **Mass data spoofing by coaching academies inflating athlete BPIs** | Scouting system loses credibility, genuine athletes overlooked | Video verification requirement for BPI >700. Anomaly detection. Coach-linked statistical analysis (if all athletes under one coach show similar anomalies → flag the coach) | Anomaly flag rate per coach, per region |
| 3 | **Server breach exposing minor athlete biometric data** | DPDP Act violation, massive reputation damage, potential criminal liability | Encryption at rest (AES-256), in transit (TLS 1.3). Biometric data stored with athlete_id as UUID (no PII in biometric tables). SOC2 Type II compliance. Annual penetration testing | Security audit score, incident response time |
| 4 | **Device thermal throttling causes app crash during video recording** | Athletes lose trust, stop recording verification videos | Thermal monitoring API: if device temp >42°C, reduce camera resolution and processing. Never run CV model + video recording simultaneously on low-end devices | Crash rate during video recording, by device model |
| 5 | **Political misuse — state government pressures to access opposition-district athlete data** | Violation of athlete trust, potential for discriminatory funding | Data access requires athlete consent + federation endorsement + audit log. All access requests logged immutably. Whistleblower mechanism for unauthorized access attempts | Access audit log review (monthly), consent revocation rate |
| 6 | **Periodization engine recommends inappropriate load during growth spurt** | Injury to adolescent athlete (e.g., Osgood-Schlatter, Sever's disease) | PHV detection algorithm, automatic load reduction during growth periods, mandatory parental notification of training modifications | Injury reports correlated with PHV-flag athletes |
| 7 | **Food database errors lead to significant calorie miscalculation** | Athletes under/over-eat, RED-S risk, weight management issues | Dual-dietitian labeling consensus, community correction mechanism, explicit accuracy disclaimers (±15%) in UI | Food item correction/dispute rate |
| 8 | **Offline queue grows too large, sync fails catastrophically** | Data loss for rural athletes with extended offline periods | Queue size cap (500 ops), queue compression, priority-based sync (biometrics first), local backup to device storage | Queue overflow incidents, data loss reports |
| 9 | **Leaderboard gaming via GPS spoofing for location-based rankings** | Unfair rankings, athlete disengagement | GPS consistency checks (cell tower triangulation cross-reference), device-level GPS spoofing detection (mock location API flag on Android) | GPS anomaly detection rate |
| 10 | **Model drift in kinematic analysis as MediaPipe updates** | Inconsistent form scores over time, athletes confused by changing metrics | Pin ML model versions per athlete cohort. Only migrate to new model version after parallel-run validation showing <5% metric deviation. Version stored per session | Model version distribution, metric drift monitoring |

---

# 9. 5-Year Roadmap

## Year 1 — Data Infrastructure & Pilot (Budget: ₹8–12 Cr)

**Focus:** Build the data layer. Prove the technology works.

**KPIs:**
- 5,000 pilot athletes across 3 states (Rajasthan, Haryana, Odisha)
- rPPG HRV accuracy validated: RMSSD error <8ms vs. chest-strap reference
- 2,000 food items in regional database
- Offline-first architecture handling 7-day offline periods
- Basic BPI v1 generating scores for pilot athletes

**Infrastructure:**
- 2 backend engineers, 2 mobile engineers, 1 ML engineer, 1 sports scientist
- AWS Mumbai: 3× m5.xlarge (backend), 1× p3.xlarge (ML training), S3 + CloudFront
- Monthly cloud cost estimate: ₹3–5 Lakh

**Milestones:**
- Q1: Core app (Android) with offline metric collection
- Q2: rPPG validation study, Rajasthan food pilot
- Q3: Periodization engine v1 (rule-based)
- Q4: BPI v1 launch, first scouting reports

---

## Year 2 — District-Level Talent Detection (Budget: ₹15–20 Cr)

**Focus:** Scale to districts. Integrate with Khelo India district trials.

**KPIs:**
- 500,000 registered athletes
- 50 districts with active scouting coverage
- Video verification pipeline processing 10,000 videos/month
- Partnership with 3 State Sports Councils
- iOS app launch

**Infrastructure:**
- Team grows to 25 (eng: 12, product: 3, sports science: 4, ops: 6)
- Multi-AZ deployment, auto-scaling groups
- Monthly cloud cost: ₹15–25 Lakh

**Key deliverables:**
- District-level leaderboards and talent reports
- Coach dashboard (web)
- Video verification at scale
- 5,000+ food items in database (5 state coverage)

---

## Year 3 — State Integration (Budget: ₹25–35 Cr)

**Focus:** Become the official data platform for state-level sports bodies.

**KPIs:**
- 5 million registered athletes
- 15 states with official partnerships
- BPI scores used in state-level selection trials (at least 5 states)
- Kinematic analysis accuracy within ±5° for primary joint angles
- PWA launch for ultra-low-end devices

**Infrastructure:**
- Team: 60+ across engineering, sports science, operations, government relations
- Dedicated ML training cluster
- Monthly cloud cost: ₹40–60 Lakh

---

## Year 4 — National High Performance Pipeline (Budget: ₹40–50 Cr)

**Focus:** SAI integration. ActiveBharat BPI becomes part of national selection criteria.

**KPIs:**
- 25 million registered athletes
- All 28 states + 8 UTs with some level of coverage
- SAI API integration for talent pipeline data sharing
- Wearable integrations (Oura, Garmin, Fitbit, Noise, boAt) for enhanced biometrics
- Multilingual support in 15+ languages
- BPI data cited in national team selection for at least 3 sports

---

## Year 5 — Olympic Integration (Budget: ₹60–80 Cr)

**Focus:** ActiveBharat data directly informing Olympic preparation.

**KPIs:**
- 100 million registered athletes
- India's Olympic contingent athletes tracked on platform
- International sports science publication using ActiveBharat dataset
- API ecosystem: 50+ third-party coach/federation integrations
- Data-driven evidence of improved national sporting outcomes (measurable improvement in medal tally correlation with BPI-identified athletes)

**Long-term infrastructure:**
- Multi-region deployment (Mumbai + Hyderabad)
- Dedicated sports science research division
- Open dataset initiative (anonymized, for sports science research)

---

# Appendix A: Glossary

| Term | Definition |
|---|---|
| ACWR | Acute:Chronic Workload Ratio — ratio of recent (7-day) training load to longer-term (28-day) average |
| BPI | Bio-Passport Index — ActiveBharat's composite scouting score |
| EWMA | Exponentially Weighted Moving Average |
| FFM | Fat-Free Mass (lean body mass) |
| GRF | Ground Reaction Force — force exerted by ground on the body during contact |
| HRV | Heart Rate Variability — beat-to-beat variation in heart rate intervals |
| LWW | Last-Writer-Wins — conflict resolution strategy for distributed systems |
| MET | Metabolic Equivalent of Task — energy cost relative to resting |
| NEAT | Non-Exercise Activity Thermogenesis |
| PHV | Peak Height Velocity — point of fastest height growth during puberty |
| RED-S | Relative Energy Deficiency in Sport |
| RMSSD | Root Mean Square of Successive Differences (HRV metric) |
| RPE | Rate of Perceived Exertion (Borg scale, 1-10) |
| rPPG | Remote Photoplethysmography — camera-based pulse measurement |
| SDNN | Standard Deviation of NN intervals (HRV metric) |
| TRIMP | Training Impulse — HR-based training load quantification |

---

*End of Document*
*Total sections: 9 | Schemas: 6 | Formulas: 12 | Pseudocode blocks: 4 | Architecture diagrams: 5*
*Classification: Internal — Engineering & Product Leadership*
