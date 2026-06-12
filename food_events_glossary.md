# Food Events – Pilot Data Glossary

## 1. The experiment

Two mice, B61 and B62, female B6.CAST-Cdh23 littermates (DOB 2025-08-07), were implanted with an Abbott Libre 3 continuous glucose monitor (CGM) that samples interstitial glucose every 5 minutes. B61 carries sensor CGM1, B62 carries sensor CGM2.

The mice are on an inverted light cycle: lights are OFF (dark, active phase) from 09:00 to 21:00, and ON (light, rest phase) from 21:00 to 09:00. All sessions were run during the dark active phase.

The arena is a 30.48 cm acrylic square open field. It has two FED3 pellet feeders mounted on opposite walls and a hydrogel water source. Pose is tracked with DeepLabCut (I'm working on moving things to SLEAP) and condense into a single CGM-anchored body centroid per frame at about 30 FPS. The CGM is the most reliable tracked body part, hence why I picked it as the anchor.

<img width="1394" height="952" alt="calibration_preview" src="https://github.com/user-attachments/assets/fb821832-6ccb-43f7-8493-c59c97fbd7f4" />



## 2. Sessions day-by-day


<img width="4444" height="2300" alt="image" src="https://github.com/user-attachments/assets/4645791c-79b2-4a96-b17c-3327a1cb29a3" />

| Date | Day | Feeders | Special manipulation | Source | Mice |
|---|---|---|---|---|---|
| 2026-04-20 | 1 | empty | 3 h fast + 1 sucrose pellet | hand-coded | B61, B62 |
| 2026-04-21 | 2 | sucrose | mice were sated | FED3 | B61, B62 |
| 2026-04-22 | 3 | sucrose | mice were sated | FED3 | B61, B62 |
| 2026-04-23 | 4 | grain | mice were sated | FED3 | B61, B62 |
| 2026-04-24 | 5 | grain + sucrose | mice were sated | FED3 | B61, B62 |
| 2026-04-25 | 6 | empty | 1 h fast + live crickets + grain + sucrose | hand-coded | B61, B62 |
| 2026-04-26 | 7 | grain + sucrose | 14 h fast | FED3 | B61 only |

**Day 1 (baseline):** The animal fasted for 3 hours, then a single 20 mg sucrose pellet was placed in the arena by hand. The FED3s were on but empty, so the timing of the sucrose pellet delivery was annotated using Datavyu.

**Days 2-5 and 7:** Standard two-feeder FED3 sessions: the mouse retrieved 20 mg pellets from two FED3s on opposite walls, with food events taken from the FED3. The sessions differ only in what each feeder held.

**Day 6 (cricket day):** Each mouse was given 8 food items in a fixed order: three crickets, a grain pellet, three more crickets, and a sucrose pellet (C1, C2, C3, grain, C4, C5, C6, sucrose), one about every 15 minutes, with a roughly 30-minute gap before the grain. FED3s were on but empty, so the timing of the food events was coded using Datavyu.

<img width="3261" height="2005" alt="image" src="https://github.com/user-attachments/assets/428bfeeb-f4b8-4327-8db0-c731e6c9508c" />


**Day 7 (grain-sucrose fasted):** Standard two-FED3 session with grain and sucrose available simultaneously from opposite walls (like in Day 5), but run after a 14-hour fast. B61 only, because B62's CGM sensor detached the day before.

> **Note:** on Days 1 and 6, the mice sometimes carried a food item away from where it was delivered and ate it somewhere else. Because of this, the <u>approach</u> path is measured from the localization frame (when the food item was deposited in the arena) to the frame of closest approach to the item and the <u>carry</u> is in a different column as the distance from the food item to the eating location (`grab_frame_minus_item_cm`: grab frame sits far from the item).


## 3. Conventions, units, and idiosyncrasies

- Since interstitial glucose lags by 10 to 15 minutes, I included lag columns (+5, +15, +30 min) to capture the response to a food event.
- For every session, I mapped the arena corners on a 30.48 cm square so that the pose data was axis-aligned and comparable across sessions.
- When a quantity is a fixed standard and not a measurement (like the 20mg pellets), I put it in a column that ends with `in _standard`.

## 4. The three deliverables

Given that pose, behavior and glucose operate at different timescales, I wanted to represent the data at three different resolutions:

- **S31 – PER FOOD EVENT**: One row per food acquisition. On days 2, 3, 4, 5, and 7, a row is one FED3 pellet retrieval. On days 1 and 6, a row is one hand-delivered item (the sucrose pellet on Day 1, each cricket or single pellet on Day 6). This is an event-level table that pairs each food event with the glucose response, the energy content, the approach path, and the location.
- **S32 – PER GLUCOSE READOUT:** One row per each 5-minute CGM Libre sample, with locomotion, occupancy, and retrieval events aggregated over the 2.5 min before and after.
- **S33 – PER FRAME:** One row per frame (~30 fps). Here you'll find the per-frame position, speed, ROI, and glucose interpolated to the frame.

## 5. Columns per deliverable

### 5a. S31 — columns per food event

| Column | Source and how it is derived | Units |
|---|---|---|
| `pellet_num_global` | [FED3] All removal events from both feeder logs, merged, sorted by retrieval time, numbered 1..N. Hand-coded days: items in grab order. | count |
| `pellet_num_in_fed` | [FED3] As above but counted within each feeder (cumulative per fed). Hand-coded days: counted within each food type. | count |
| `fed` | [FED3] Which feeder unit logged the removal (fed001 or fed002). Hand-coded days: the literal value hand_delivered. | label |
| `fed_content` | [Constant / session config] What that feeder was loaded with this session, declared in the config and cross-checked against the FED log filename. Hand-coded days: the item type. | label |
| `video_date, video_timestamp` | [Sync] The event mapped to the burned-in video clock; date and time of day. | date / time |
| `retrieval_pi_corrected_iso` | [FED3 + Sync] The feeder RTC timestamp of the removal, corrected to the Pi master clock via the learned per-session RTC-to-Pi offset. Hand-coded days: the DataVyu grab time. | ISO datetime |
| `video_clock_iso` | [Sync] The same event mapped to the burned-in video clock. | ISO datetime |
| `chunk_num, frame_index_in_chunk` | [Sync] The recording chunk and within-chunk frame index that contains the event, from the chunk-to-global-frame pairing table. | frame |
| `chunk_first_global_frame, chunk_last_global_frame` | [Sync] The global frame span of that chunk. Blank on hand-coded days, which use per-chunk frames only. | frame |
| `was_timed_out` | [FED3] The firmware Timed_out flag on the event row. False on hand-coded days. | boolean |
| `in_video_gap` | [Sync] True if the event time falls in a recording gap with no video frames. | boolean |
| `minutes_in_session` | [Sync] (event time minus session start) / 60. | minutes |
| `kcal_per_pellet` | [Constant] Looked up from the pellet-energy table by fed_content (sucrose 0.0728, grain 0.067). Cricket blank: not measured. | kcal |
| `cumulative_kcal_session` | [Constant] Running sum of kcal_per_pellet over events up to and including this one. | kcal |
| `glucose_at_retrieval_mg_dl` | [CGM] Linear interpolation of the 5-minute interstitial series to the event timestamp. Blank if the event is outside CGM coverage. | mg/dL |
| `prev_pellet_fed` | [FED3] The fed value of the previous event in retrieval order. Hand-coded days: previous food type. | label |
| `is_fed_switch` | [FED3] True if fed differs from prev_pellet_fed. Hand-coded days: food-type switch. | boolean |
| `time_since_prev_retrieval_s, ..._same_fed_s` | [FED3] Differences of consecutive event timestamps, overall and restricted to the same feeder/content. | seconds |
| `time_to_next_retrieval_s, ..._same_fed_s` | [FED3] Same, looking forward to the next event. | seconds |
| `time_at_current_fed_before_retrieval_s` | [DLC + Calibration] Time the centroid spent inside this feeder's ROI during the window before the retrieval. Blank on hand-coded days. | seconds |
| `travel_time_s` | [DLC + Sync] Duration from leaving the previous feeder ROI to entering the current one (frames to seconds at 30 fps). Hand-coded days: time from item localization to closest approach. | seconds |
| `travel_distance_cm` | [DLC] Sum of frame-to-frame centroid displacements (in canonical cm) over that travel segment. | cm |
| `mean_speed_during_travel_cm_s` | [DLC] travel_distance_cm divided by travel_time_s. | cm/s |
| `travel_t_leave_frame, travel_t_enter_frame` | [DLC + Calibration] The frames where the centroid left the previous ROI and entered the current ROI, found from the per-frame ROI sequence. | frame |
| `time_in_fed001/fed002/hydrogel/elsewhere_during_travel_s` | [DLC + Calibration] Count of travel-segment frames whose centroid is in each ROI, divided by 30. FED3 days. | seconds |
| `time_in_drinking_bout_during_travel_s` | [DLC + Calibration] Time in detected drinking bouts (both ears inside the hydrogel ROI) during travel. Inferred, not observed. | seconds |
| `retrieval_position_dist_cm` | [DLC + Calibration / Localizer] Distance from the centroid to the target at the event frame: the feeder on FED3 days, the localized item on hand-coded days. | cm |
| `position_quality` | [DLC] Provenance/reliability flag for the event position: tracked vs in-gap on FED3 days; hand_localized or hand_localized_carry on hand-coded days. | label |

### 5b. S32 — columns per 5-minute CGM sample

#### Shared for all days

| Column | Source and how it is derived | Units |
|---|---|---|
| `cgm_sample_iso (cgm_sample_pi_iso)` | [CGM] The timestamp of the Libre sample. | ISO datetime |
| `glucose_mg_dl` | [CGM] The measured glucose value at that sample. Raw, not interpolated. | mg/dL |
| `glucose_mg_dl_minus_prev_sample` | [CGM] This sample minus the previous sample (local slope). | mg/dL |
| `minutes_in_session` | [Sync] (sample time minus session start) / 60. | minutes |
| `light_dark_phase` | [Constant] Wall-clock hour compared to the inverted light schedule (dark_active 09:00 to 21:00). | label |
| `mean_speed_cm_s, max_speed_cm_s` | [DLC] Mean and max centroid speed over the window's tracked frames. | cm/s |
| `total_travel_distance_cm` | [DLC] Sum of centroid displacements (canonical cm) over the window. | cm |
| `mean_X_cm, mean_Y_cm` | [DLC] Mean centroid position over the window. | cm |
| `frac_near_wall` | [DLC + Calibration] Fraction of tracked frames whose centroid is within a quarter-arena of a wall. | fraction |
| `n_video_frames_in_bin` | [Sync] Number of video frames whose timestamp falls in the window. | count |
| `n_tracked_frames_in_bin` | [DLC] Of those, the number with a valid centroid. | count |
| `kcal_in_bin (kcal_retrieved_in_bin)` | [FED3 or DataVyu + Constant] Energy of the items acquired in the window. | kcal |

#### For hand-coded-days

| Column | Source and how it is derived | Units |
|---|---|---|
| `items_acquired` | [DataVyu] Count of coded grabs whose time falls in the window (all types). | count |
| `crickets_acquired, grain_acquired, sucrose_acquired` | [DataVyu] The same, split by item type. | count |

#### For FED3 days

| Column | Source and how it is derived | Units |
|---|---|---|
| `n_pellets_retrieved` | [FED3] Removal events in the window. Retrieval, a proxy for intake. | count |
| `n_pellets_fed001, n_pellets_fed002` | [FED3] The same, per feeder. | count |
| `n_pellets_timed_out` | [FED3] Firmware timeouts in the window. | count |
| `n_fed_switches` | [FED3] Changes of feeder between consecutive retrievals in the window. | count |
| `time_in_fed001/fed002/hydrogel/elsewhere_s` | [DLC + Calibration] Tracked frames per ROI in the window, divided by 30. | seconds |
| `n_fed001/fed002/hydrogel_roi_entries_in_bin` | [DLC + Calibration] Debounced ROI-entry transitions in the window. | count |
| `time_in_drinking_bout_s, n_drinking_bouts_in_bin` | [DLC + Calibration] Detected drinking bouts (ears in hydrogel ROI). Inferred. | seconds / count |
| `time_since_last_pellet_at_bin_start_s` | [FED3 + Sync] Time from the previous retrieval to the window start. | seconds |
| `bin_overlaps_chunk_gap, bin_video_coverage_pct, n_distinct_video_chunks_in_bin` | [Sync] Coverage and data-quality flags for the window. | mixed |

### 5c. S33 — columns per video frame

#### Shared for all days

| Column | Source and how it is derived | Units |
|---|---|---|
| `frame_index` | [Sync] Global frame index across the session. | frame |
| `chunk_num, frame_index_in_chunk` | [Sync] The recording chunk and within-chunk frame index (0-based). | frame |
| `pi_iso (video_iso)` | [Sync] The frame's timestamp on the master (or video) clock. | ISO datetime |
| `x_cm, y_cm` | [DLC + Calibration] The CGM-anchored centroid mapped to canonical cm through the calibration homography. | cm |
| `cx_px, cy_px` | [DLC] The raw centroid in image pixels (hand-coded-day tables). | pixels |
| `speed_cm_s` | [DLC] Frame-to-frame centroid displacement times 30, divided by px_per_cm. | cm/s |
| `glucose_mg_dl_interp` | [CGM] The 5-minute series linearly interpolated to this frame's time. | mg/dL |
| `light_dark_phase` | [Constant] Frame hour vs the inverted light schedule. | label |
| `is_pellet_retrieval` | [FED3 or DataVyu] True on the frames at a retrieval (FED3) or a coded grab (hand-coded). | boolean |

#### For hand-coded-days

| Column | Source and how it is derived | Units |
|---|---|---|
| `is_eating` | [DataVyu] True inside a coded eat onset-to-offset interval. Observed, not inferred. | boolean |
| `current_item, current_food_type` | [DataVyu] The item being eaten on that frame. | label |

#### For FED3 days

| Column | Source and how it is derived | Units |
|---|---|---|
| `roi` | [DLC + Calibration] Which ROI polygon the centroid falls in (fed001, fed002, hydrogel, elsewhere). | label |
| `pellet_fed, kcal_retrieved_at_frame` | [FED3 + Constant] The feeder and energy if a retrieval occurs on this frame. | label / kcal |
| `glucose_mg_dl_measured, is_cgm_sample_frame` | [CGM] The measured sample value on frames that coincide with a real Libre sample. | mg/dL / boolean |
| `in_chunk_gap, in_video_gap` | [Sync] The frame falls in a recording gap. | boolean |
| `has_tracking, tracking_status` | [DLC] Whether a valid centroid exists for the frame and its status label. | boolean / label |
| `centroid_method` | [DLC] How the centroid was formed on this frame: cgm_anchored or clustered_no_cgm. | label |
| `n_bodyparts_used` | [DLC] Number of keypoints that contributed to the centroid. | count |
| `is_drinking_posture, is_drinking_bout, is_nose_lost, is_nose_lost_at_fed` | [DLC + Calibration] Posture and occlusion inferences with thresholds. Inferred. | boolean |
