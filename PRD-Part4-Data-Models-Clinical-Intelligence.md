**BIRTH MODEL**

PRD Part 4 --- Patient Data Model & Clinical Intelligence

Complete Field Reference · Nurse Card (DOM class: nurse-card-v2) · Vital Alerts · Cervical Ripening · PPH · Medications

*Source: birth-model-unified-complete.html --- renderNurseCard(), PATIENTS array, medication data models*

**Dr. Anish J. Shah, MD, FACOG**

CEO & Founder, Birth Model, Inc. · March 2026

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **⚠ ENGINEERING NOTE: Throughout this document: the DOM CSS class for the nurse card is nurse-card-v2. The internal CSS prefix is nc3- (3rd-generation design). Do NOT rename the DOM class. Never reference \"Nurse Card v3\" in code --- only in plain-English descriptions.**

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**1. Complete Patient Field Reference**

**1.1 CORE IDENTITY**

  -----------------------------------------------------------------------------------------------------------
  **Field**   **Type**   **Example**           **Notes**
  ----------- ---------- --------------------- --------------------------------------------------------------
  id          number     1                     Unique patient ID

  room        string     \"0881-01\"           Must exactly match a value in ALL_ROOMS or ALL_ROOMS_HUNTLEY

  name        string     \"Maria Santos\"      Full display name

  age         number     27                    Age in years

  mrn         string     \"NWM-284519\"        Format NWM-XXXXXX for Prentice

  dob         string     \"06/15/1998\"        MM/DD/YYYY

  edd         string     \"03/08/2026\"        Estimated due date MM/DD/YYYY

  gtpal       string     \"G2P1001\"           Gravida/Term/Preterm/Ab/Living

  ga          string     \"39w2d\"             Gestational age display string

  section     string     \"L&D South\"         Must match a value in SECTIONS or SECTIONS_HUNTLEY

  race        string     \"Hispanic/Latina\"   Used in OB-Servability demographics breakdown
  -----------------------------------------------------------------------------------------------------------

**1.2 CLINICAL STATUS FIELDS**

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field**       **Type**         **Valid Values**                                                                                                                                                                        **Notes**
  --------------- ---------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- -------------------------------------------------------------------------------------------------------------------------------------
  status          string           \"labor\" \| \"active\" \| \"induction\" \| \"cs\" \| \"delivered\" \| \"postpartum\" \| \"antepartum\" \| \"triage\"                                                                   Primary status --- drives statusColorClass and acuity

  substatus       string           \"Induction\" \| \"IOL\" \| \"Pushing\" \| \"Scheduled C/S\" \| \"STAT\" \| \"Delivered Vaginal\" \| \"Delivered CS - Twins\" \| \"Procedure\" \| \"Fully Dilated\" \| \"Antepartum\"   Sub-status for granular display

  indication      string           \"Post-dates\" \| \"IUFD\" \| \"Cord prolapse\" \| etc.                                                                                                                                 Clinical indication; used in delivery note diagnoses auto-fill

  pushing         boolean          true / false                                                                                                                                                                            1:1 acuity; status-pushing color class

  pph             object \| null   {uterotonics, devices, bloodProducts} or null                                                                                                                                           Active PPH --- triggers status-pph (HIGHEST priority color). See Section 5.

  iufd            boolean          true / false                                                                                                                                                                            IUFD --- status-iufd color; 1:1 acuity (Loss)

  loss            boolean          true / false                                                                                                                                                                            Used alongside iufd for acuity loss check

  pree            boolean          true / false                                                                                                                                                                            Preeclampsia. Alone ≠ 1:1 acuity. With mag = High Risk on charge board.

  mag             boolean          true / false                                                                                                                                                                            Magnesium infusing --- 1:1 acuity; High Risk color with pree

  insulin         boolean          true / false                                                                                                                                                                            Insulin drip --- 1:1 acuity

  twins           boolean          true / false                                                                                                                                                                            Twin gestation --- 1:1 acuity; High Risk with pree on charge board

  triplets        boolean          true / false                                                                                                                                                                            Triplets --- High Risk on charge board regardless of pree/mag

  tolac           boolean          true / false                                                                                                                                                                            Trial of Labor After Cesarean --- status-vtol; adds to nurse card high risk counter

  pphRisk         string           \"Low\" \| \"Medium\" \| \"High\"                                                                                                                                                       Stored AWHONN admission risk (set by nurse at admission). NOT computed at render. Feeds nurse card high-risk counter when \"High\".

  convertedToCS   boolean          true / false                                                                                                                                                                            Was laboring patient converted to emergency C/S

  csReason        string           \"Cord prolapse - EMERGENCY\"                                                                                                                                                           Reason for C/S conversion
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**1.3 LABOR EXAM FIELDS**

  -----------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field**        **Type**         **Range / Example**                        **Notes**
  ---------------- ---------------- ------------------------------------------ ------------------------------------------------------------------------------
  dilation         number \| null   0--10 cm                                   ≥10 = 1:1 acuity \"Complete\"; ≥6 = Active stage in nurse card; \<6 = Latent

  effacement       number \| null   0--100%                                    ---

  station          number \| null   -3 to +4                                   Displayed as \"+N\" or \"-N\" or \"\--\"

  consistency      string           \"Soft\" \| \"Medium\" \| \"Firm\"         Cervical consistency

  position         string           \"Anterior\" \| \"Posterior\" \| \"Mid\"   Cervical OS position

  lastExam         string           \"1:00 PM\"                                Time of last cervical exam

  bishopScore      number \| null   0--13                                      Bishop score for IOL readiness assessment

  registeredAt     string           \"6:00 AM\" \| \"Yesterday 6:00 AM\"       \"Yesterday\" prefix used for long admissions

  completeAt       string \| null   \"1:30 PM\"                                Time reached 10cm

  pushingStarted   string \| null   \"2:15 PM\"                                Time active pushing began (2nd stage start)

  foleyDate        string \| null   \"Mar 5\"                                  Date of Foley/balloon catheter placement for IOL
  -----------------------------------------------------------------------------------------------------------------------------------------------------------

**1.4 GBS-SPECIFIC FIELDS**

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field**     **Type**   **Example**                                       **Notes**
  ------------- ---------- ------------------------------------------------- ---------------------------------------------------------------------------------------------------------------------
  gbs           string     \"Positive\" \| \"Negative\" \| \"Unknown\"       GBS culture result. Also accepts \"+\" as equivalent to \"Positive\" in nurse card GBS logic.

  gbsDate       string     \"Feb 18\"                                        Date of GBS culture result

  antibiotics   string     \"Clindamycin 900mg IV q8h (GBS, PCN allergy)\"   Freetext antibiotic string. Nurse card checks this for GBS coverage using regex: /gbs\|pcn\|ampicillin\|penicillin/
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**1.5 DELIVERY PREDICTION FIELDS**

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field**            **Type**           **Example**                                                            **Notes**
  -------------------- ------------------ ---------------------------------------------------------------------- ---------------------------------------------------------------------------------------------------------------
  est10cm              string             \"2:15 PM\"                                                            Birth Model predicted time to 10cm

  estDelivery          string \| object   \"2:45 PM\" \| \"STAT NOW\" \| { date: \"TMRW\", time: \"9:00 AM\" }   Predicted delivery time. Can be time string, \"STAT NOW\", or object with date+time for next-day predictions.

  confidence           string             \"±12 min\" \| \"Emergency\"                                           Confidence interval shown with prediction

  predictedDelivery    string             \"7:20 PM\"                                                            Post-delivery: what the model predicted (accuracy display)

  predictionVariance   string             \"+ 1 min\"                                                            Post-delivery: delta between predicted and actual
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**1.6 CERVICAL RIPENING --- THREE SEPARATE FIELDS (IMPORTANT DISTINCTION)**

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **⚠ ENGINEERING NOTE: Three different fields represent cervical ripening. They are NOT interchangeable. patient.cervicalRipening is the full structured object. patient.ripeningDetails is a simple summary. patient.ripening is a legacy string.**

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field**          **Type**   **Example**                                                                                                                **Used By**                                                    **Precedence**
  ------------------ ---------- -------------------------------------------------------------------------------------------------------------------------- -------------------------------------------------------------- -----------------------------------------------------------------
  cervicalRipening   object     { misoprostol: \[{dose, route, datetime}\], crib: {device, placedAt, removedAt}, dinoprostone: {dose, route, placedAt} }   Patient drawer .cervical-ripening section (full detail view)   Full structured data --- always prefer this

  ripeningDetails    object     {type: \"Misoprostol\", count: 2}                                                                                          Nurse card medInfo display                                     Simplified summary used when cervicalRipening detail not needed

  ripening           string     \"Misoprostol x2\"                                                                                                         Nurse card medInfo fallback                                    Legacy string; only used if ripeningDetails is also absent
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Nurse card precedence check: p.ripeningDetails \|\| p.ripening \|\| (p.pitocin && p.pitocin \> 0) \|\| p.cervicalRipening?.misoprostol \|\| p.cervicalRipening?.dinoprostone \|\| p.cervicalRipening?.crib

**2. Nurse Card --- renderNurseCard() (DOM class: nurse-card-v2)**

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  📝 The DOM class is nurse-card-v2. The CSS variable prefix is nc3- (design generation 3). These refer to the same component. Never use \"v3\" as a class or identifier in code.

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**2.1 GRID LAYOUTS BY PATIENT COUNT (DATA-COUNT ATTRIBUTE ON .NC-CARDS-AREA)**

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------
  **data-count**   **Grid Columns**    **Grid Rows**    **Special Layout**
  ---------------- ------------------- ---------------- ----------------------------------------------------------------------------------------------------------
  1                Single full-width   1                Max font sizes

  2                repeat(2, 1fr)      1                Side-by-side; vitals+labs shown in row when .unit-panel-open on body

  3                repeat(3, 1fr)      1                Three columns; condensed font

  4                repeat(2, 1fr)      repeat(2, 1fr)   2×2 grid

  5                repeat(6, 1fr)      repeat(2, 1fr)   Top row: children 1-3 each span 2 of 6 cols. Bottom row: child 4 = col 2-4; child 5 = col 4-6 (centered)
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------

  -------------------------------------------------------------------------------------------------------------------------------------------------------------
  📝 When body.unit-panel-open is active: cards condense (inset: 6px; gap: 6px). Font sizes and padding reduce proportionally across all data-count variants.

  -------------------------------------------------------------------------------------------------------------------------------------------------------------

**2.2 STATUS LABELS IN NURSE CARD (SEPARATE FROM 14-STATUS COLOR SYSTEM IN PART 1)**

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Condition**                                      **statusLabel**                     **statusClass**   **statusIcon**   **Note**
  -------------------------------------------------- ----------------------------------- ----------------- ---------------- -----------------------------------------------------------
  isDelivered (p.status === \"delivered\")           p.delivery?.type or \"Delivered\"   delivered         ✓                ---

  p.pushing                                          Pushing                             pushing           ⬆                ---

  p.status === \"cs\" AND p.substatus === \"STAT\"   STAT C/S                            stat              ⚡               ---

  p.status === \"cs\" (non-STAT)                     C-Section                           cs                ⚡               ---

  p.dilation \>= 10                                  Complete                            complete          ●                ---

  p.dilation \>= 6                                   Active                              active            ---              ---

  default (none above match)                         Latent                              latent            ---              Only labor stages shown here --- NOT conditions like PreE
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**2.3 HIGH RISK BADGE IN NURSE CARD (EARLY IN RENDERNURSECARD --- FOR THE RISK LABEL BADGE ONLY)**

  --------------------------------------------------------------------------------------
  **Condition**          **isHighRisk**   **riskReason label**   **Notes**
  ---------------------- ---------------- ---------------------- -----------------------
  p.pree                 true             PreE                   ---

  p.iufd                 true             IUFD                   ---

  p.tolac                true             TOLAC                  ---

  p.twins                true             Twins                  ---
  --------------------------------------------------------------------------------------

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  📝 This isHighRisk (with riskReason) is only for displaying the risk label badge text. It is NOT the isHighRisk used for status color assignment. There are two separate isHighRisk variables within renderNurseCard. See 2.4 for the one that drives status color.

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**2.4 HIGH RISK STATUS COLOR COUNTER IN NURSE CARD (LATER SCOPE IN RENDERNURSECARD --- DRIVES STATUSCOLORCLASS)**

This second isHighRisk in renderNurseCard determines whether to assign status-high-risk CSS class. Different formula from charge board (Part 1, Sec 3.2):

  -------------------------------------------------------------------------------------------------------------------------------------------------
  **Check**                                            **Effect**                               **Notes**
  ---------------------------------------------------- ---------------------------------------- ---------------------------------------------------
  p.warnings?.some(w =\> w.includes(\"CATEGORY 1\"))   isHighRisk = true (immediate)            Fetal tracing alert in warnings array

  p.pree                                               riskFactors += 1                         ---

  p.mag                                                riskFactors += 1                         ---

  p.iufd                                               isHighRisk = true AND riskFactors += 1   IUFD = direct flag + counts toward threshold

  p.twins \|\| p.triplets                              riskFactors += 1                         ---

  p.tolac                                              riskFactors += 1                         ---

  p.pphRisk === \"High\"                               riskFactors += 1                         Reads stored patient.pphRisk field from admission

  riskFactors \>= 2                                    isHighRisk = true                        Counter threshold
  -------------------------------------------------------------------------------------------------------------------------------------------------

**2.5 GBS ANTIBIOTIC COVERAGE LOGIC (NURSE CARD DISPLAY)**

  ----------------------------------------------------------------------------------------------------------------------------------------
  **GBS Status**                **p.antibiotics string check**                    **gbsStatus result**   **Display**
  ----------------------------- ------------------------------------------------- ---------------------- ---------------------------------
  Positive OR \"+\"             regex match: /gbs\|pcn\|ampicillin\|penicillin/   covered                Green --- covered

  Positive OR \"+\"             No match or p.antibiotics null/undefined          needs-abx              **⚠ Red --- needs antibiotics**

  Negative \| Unknown \| null   (any)                                             null                   Row not displayed
  ----------------------------------------------------------------------------------------------------------------------------------------

**2.6 PITOCIN / RIPENING DISPLAY IN NURSE CARD (MEDINFO FIELD)**

  --------------------------------------------------------------------------------------------------------------------------
  **Check Order**   **Condition**                                                      **medInfo Value Shown**
  ----------------- ------------------------------------------------------------------ -------------------------------------
  1st               p.pitocin && p.pitocin \> 0                                        \`Pit \${p.pitocin} mU\`

  2nd               p.ripeningDetails?.type === \"Misoprostol\"                        \`Miso #\${ripeningDetails.count}\`

  2nd               p.ripeningDetails?.type === \"Cervidil\"                           \"Cervidil\"

  2nd               p.ripeningDetails?.type === \"CRIB\" or \"Foley\" or \"Balloon\"   \"CRIB\"

  3rd               p.ripening (legacy string)                                         p.ripening value directly

  Fallback          Induction patient with nothing started yet                         Ready for IOL
  --------------------------------------------------------------------------------------------------------------------------

**3. Vital Alert Logic & Trends**

**3.1 PATIENT.VITALS OBJECT**

  ------------------------------------------------------------------------------------------------------------------------------------------
  **Field**   **Type**         **Example**   **Notes**
  ----------- ---------------- ------------- -----------------------------------------------------------------------------------------------
  bp          string           \"148/92\"    Parsed: systolic = parseInt(bp.split(\"/\")\[0\]); diastolic = parseInt(bp.split(\"/\")\[1\])

  hr          number           95            Heart rate bpm

  temp        number           99.2          Fahrenheit

  fhr         number \| null   145           Fetal heart rate bpm; null for delivered patients

  spo2        number           97            Pulse oximetry %
  ------------------------------------------------------------------------------------------------------------------------------------------

**3.2 TREND CALCULATION FROM VITALHISTORY (LAST 4 READINGS, INDEX 0 = MOST RECENT)**

  ------------------------------------------------------------------------------------------------------------
  **Vital**     **Up Condition**              **Down Condition**            **Trend arrow variable**
  ------------- ----------------------------- ----------------------------- ----------------------------------
  Systolic BP   current \> avgPrev3 + 8       current \< avgPrev3 − 8       bpTrend: \"↑\" \| \"↓\" \| \"→\"

  Temperature   current \> avgPrev3 + 0.3°F   current \< avgPrev3 − 0.3°F   tempTrend: \"↑\" \| \"↓\"

  Heart Rate    current \> avgPrev3 + 10      current \< avgPrev3 − 10      hrTrend: \"↑\" \| \"↓\"
  ------------------------------------------------------------------------------------------------------------

**3.3 BP ALERT CLASSES**

  ----------------------------------------------------------------------------------------------------------------------------------
  **Condition**                     **bpClass**   **bpRecheck message**   **Clinical Significance**
  --------------------------------- ------------- ----------------------- ----------------------------------------------------------
  systolic ≥160 OR diastolic ≥110   critical      \"Recheck 15min\"       Severe range hypertension --- acute management threshold

  systolic ≥140 OR diastolic ≥90    warning       \"Recheck 30min\"       Preeclampsia BP threshold

  Below both                        \"\"          \"\"                    Normal for pregnancy
  ----------------------------------------------------------------------------------------------------------------------------------

**3.4 TEMPERATURE ALERT CLASSES**

  ---------------------------------------------------------------------------------------------------------------
  **Condition**    **tempClass**   **tempRecheck**     **Clinical Significance**
  ---------------- --------------- ------------------- ----------------------------------------------------------
  temp ≥ 100.4°F   critical        \"Recheck 30min\"   Fever --- chorioamnionitis concern; hasSepsisRisk = true

  temp ≥ 99.5°F    warning         \"Recheck 1h\"      Low-grade --- monitor; hasSepsisRisk = true

  temp \< 99.5°F   \"\"            \"\"                Normal
  ---------------------------------------------------------------------------------------------------------------

**3.5 HR, SPO2, SEPSIS FLAG**

  --------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Vital / Flag**          **Condition**                                        **Class**              **Notes**
  ------------------------- ---------------------------------------------------- ---------------------- --------------------------------------------------------
  HR                        ≥ 120 bpm                                            critical (hrClass)     Tachycardia

  HR                        ≥ 100 bpm                                            warning (hrClass)      Borderline tachycardia

  SpO2                      \< 92%                                               critical (spo2Class)   Hypoxia

  SpO2                      \< 95%                                               warning (spo2Class)    Low-normal

  hasSepsisRisk (boolean)   (p.vitals?.temp \>= 100.4) OR (p.labs?.wbc \>= 15)   ---                    Displayed as sepsis risk badge on nurse card when true
  --------------------------------------------------------------------------------------------------------------------------------------------------------------

**4. Cervical Ripening Section**

Displayed as collapsible .cervical-ripening section in patient drawer. All three ripening objects (misoprostol, crib, dinoprostone) are within patient.cervicalRipening.

**4.1 PATIENT.CERVICALRIPENING OBJECT**

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Sub-key**    **Type**         **Shape**                                                                       **Notes**
  -------------- ---------------- ------------------------------------------------------------------------------- -----------------------------------------------------------------------------------------------------------------------
  misoprostol    array \| null    \[{ dose: \"25mcg\", route: \"Vaginal\", datetime: \"03/04/2026 6:00 AM\" }\]   Array of dose objects. Updated via formatDateTime(). Q4h dosing typical. Times updated to relative times at app init.

  crib           object \| null   { device: \"Cook Catheter\", placedAt: \"\...\", removedAt: null }              Single device entry. placedAt and removedAt tracked. Duration calculated if removedAt is null.

  dinoprostone   object \| null   { dose: \"10mg\", route: \"Vaginal insert\", placedAt: \"\...\" }               Cervidil. One at a time.
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**4.2 TOLAC CONTRAINDICATION BADGE**

  ------------------------------------------------------------------------------------------------------------------------------------
  **Agent**                       **Contraindicated in TOLAC?**   **Display**
  ------------------------------- ------------------------------- --------------------------------------------------------------------
  Misoprostol                     YES                             Shows .tolac-warning badge \"CI IN TOLAC\" on .ripening-agent-name

  Dinoprostone (Cervidil)         YES                             Shows .tolac-warning badge

  Cook Catheter / Foley Balloon   No --- mechanical only          No warning badge

  Oxytocin                        No (caution, not CI)            Not in cervical ripening section --- shown in medication panel
  ------------------------------------------------------------------------------------------------------------------------------------

**5. PPH Active Management --- patient.pph Object**

When patient.pph is truthy (non-null, non-false), the patient receives status-pph --- the HIGHEST priority color class. The pph object tracks interventions:

**5.1 PATIENT.PPH OBJECT STRUCTURE**

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Sub-array**   **Item Fields**              **Example Values**
  --------------- ---------------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  uterotonics     { drug, dose, type, time }   { drug: \"Oxytocin\", dose: \"40U in 1L LR\", type: \"infusing\", time: \"11:55 AM\" } · { drug: \"Methergine\", dose: \"0.2mg IM\", type: \"given\", time: \"12:00 PM\" } · { drug: \"Cytotec\", dose: \"800mcg PR\", type: \"given\", time: \"12:05 PM\" }

  devices         { type, status, time }       { type: \"Bakri Balloon\", status: \"Placed, 300mL\", time: \"12:15 PM\" } · { type: \"JADA System\", status: \"Active\", time: \"11:45 AM\" }

  bloodProducts   { type, units, time }        { type: \"pRBC\", units: 2, time: \"12:30 PM\" } · { type: \"FFP\", units: 2, time: \"12:45 PM\" }
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  📝 \"type\" field in uterotonics: \"infusing\" = currently running IV drip; \"given\" = bolus/IM dose already administered. Do not confuse with device \"type\" in the devices array.

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**5.2 PATIENT.WARNINGS ARRAY --- PPH CLINICAL ALERTS**

  ------------------------------------------------------------------------------------------------------------
  **Example String**                   **Trigger Context**
  ------------------------------------ -----------------------------------------------------------------------
  \"PPH --- QBL 1250 mL\"              Always present when pph active; QBL updated as tracked

  \"Bakri Balloon IN PLACE (300mL)\"   Bakri intrauterine balloon placed

  \"JADA System IN PLACE\"             JADA negative-pressure intrauterine device active

  \"MTP activated\"                    Massive Transfusion Protocol triggered

  \"Type & Screen pending\"            Crossmatch not yet complete

  \"Trend H/H closely\"                Serial Hgb/Hct monitoring instruction

  \"NO METHERGINE - preeclampsia\"     Methylergonovine is CI in hypertensive disorders --- warn prominently

  \"BP monitoring q15min\"             Severe range BP monitoring requirement

  \"Mag sulfate at bedside\"           Seizure prophylaxis readiness reminder
  ------------------------------------------------------------------------------------------------------------

**6. Medication Data Models**

**6.1 PATIENT.MAGSULFATE**

  -------------------------------------------------------------------------
  **Field**       **Type**   **Example**
  --------------- ---------- ----------------------------------------------
  loading         string     \"4g IV over 20 min\"

  maintenance     string     \"2g/hr IV\"

  started         string     \"6:50 AM\" (updated by timeOffset)
  -------------------------------------------------------------------------

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  📝 p.mag = true means MgSO4 is running. magSulfate object holds the specific protocol details. Do not conflate p.mag (boolean flag) with the magSulfate object (dose/rate/time data).

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**6.2 PATIENT.BPMEDS (ARRAY)**

  -------------------------------------------------------------------------------------
  **Field**       **Type**   **Example**
  --------------- ---------- ----------------------------------------------------------
  drug            string     \"Labetalol\"

  dose            string     \"200mg IV\"

  time            string     \"12:30 PM\"

  reason          string     \"BP 152/96\" (BP reading that triggered administration)
  -------------------------------------------------------------------------------------

**6.3 PATIENT.STEROIDS**

  ---------------------------------------------------------------------------------------
  **Field**   **Type**   **Example**                 **Notes**
  ----------- ---------- --------------------------- ------------------------------------
  drug        string     \"Betamethasone\"           Standard agent for lung maturity

  doses       string     \"12mg IM x2, 24h apart\"   Dosing schedule

  firstDose   string     \"Yesterday 11:45 AM\"      Set to relative time at app init

  complete    boolean    false                       True when both doses administered
  ---------------------------------------------------------------------------------------

**6.4 PATIENT.OXYHISTORY (ARRAY)**

  --------------------------------------------------------------------------------------------------------------------------------------------
  **Field**   **Type**            **Example**                                 **Notes**
  ----------- ------------------- ------------------------------------------- ----------------------------------------------------------------
  dose        number              8                                           Oxytocin dose in mU/min

  t           string              \"2:00 PM\"                                 Time of dose change (Q30min increments, updated by timeOffset)

  note        string (optional)   \"Increased for inadequate contractions\"   Reason for change
  --------------------------------------------------------------------------------------------------------------------------------------------

**6.5 PATIENT.LASTMED**

  -------------------------------------------------------------------------
  **Field**       **Type**   **Example**
  --------------- ---------- ----------------------------------------------
  name            string     \"Oxytocin\"

  dose            string     \"8 mU/min\"

  time            string     \"1:00 PM\"

  route           string     \"IV\"
  -------------------------------------------------------------------------

**7. ROM Duration & Urgency**

**7.1 PATIENT.ROM OBJECT**

  ---------------------------------------------------------------------------------------------------------------------------------------
  **Field**   **Type**         **Values**                                                               **Notes**
  ----------- ---------------- ------------------------------------------------------------------------ ---------------------------------
  type        string           \"AROM\" \| \"SROM\" \| \"PPROM\" \| \"Intact\" \| \"Intact until CS\"   Rupture type

  time        string \| null   \"1:00 PM\" or null                                                      null if membranes intact

  color       string           \"Clear\" \| \"Meconium\" \| \"Bloody\"                                  Amniotic fluid color at rupture
  ---------------------------------------------------------------------------------------------------------------------------------------

**7.2 ROMURGENCY CLASSES (COMPUTED IN RENDERNURSECARD)**

  -------------------------------------------------------------------------------------------------------------------------------------------
  **romHours value**                **romUrgency**          **Alert color**   **Applies when**
  --------------------------------- ----------------------- ----------------- ---------------------------------------------------------------
  \< 12                             ok                      ---               Normal

  12--18                            caution                 Amber / warning   Approaching prolonged ROM threshold

  \> 18                             urgent                  Red / critical    **⚠ Prolonged ROM --- significantly elevated infection risk**

  type === \"Intact\" (any hours)   null (not calculated)   ---               ROM not applicable --- row not shown
  -------------------------------------------------------------------------------------------------------------------------------------------

**8. Progress Note Interval Requirements**

patient.lastProgressNote stores the time of the most recent documented progress note (e.g. "4:45 PM"). The nurse card uses elapsed minutes from this time against the expected interval for the patient's current labor stage to flag overdue documentation.

**EXPECTED NOTE INTERVALS BY LABOR STAGE**

  ----------------------------------------------------------------------------------------------------------------------------------------------
  **Labor Stage**       **Interval**   **Overdue Trigger**   **Status Condition (p.dilation)**
  --------------------- -------------- --------------------- -----------------------------------------------------------------------------------
  Latent Labor          **Q4H**        \> 4 hours elapsed    p.dilation \< 6 (nurse card statusClass: latent)

  Active Labor          **Q2H**        \> 2 hours elapsed    p.dilation 6--9 (nurse card statusClass: active)

  Pushing / 2nd Stage   **Q1H**        \> 1 hour elapsed     p.pushing = true OR p.dilation = 10 (nurse card statusClass: pushing or complete)
  ----------------------------------------------------------------------------------------------------------------------------------------------

**9. History Arrays**

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Array Field**   **Item Shape**                                                   **Time Update Logic**
  ----------------- ---------------------------------------------------------------- ----------------------------------------------------------------------------------------------
  roomHistory       { time, room, reason }                                           Delivered patients: times relative to delivery time. Active: timeOffset at 90-min intervals.

  providerHistory   { provider, startTime, endTime }                                 endTime: null = current coverage

  nurseHistory      { date, startTime, nurse }                                       date = today or yesterday (ISO format); startTime = \"7:00 AM\" or \"7:00 PM\"

  cervixHistory     { time, dilation, effacement, station, consistency, position }   times updated via formatDateTime() at \~120-min intervals

  labHistory        { time, hgb, plt, wbc, \... }                                    time = \"Admission\" \| \"Prior day\" \| time string (240-min intervals)

  preeLabHistory    { time, ast, alt, ldh, creat, mag, plt }                         Index 0 = timeOffset(-120); index 1 = \"Prior day\"

  vitalHistory      { t, bp, hr, temp, spo2, fhr }                                   Updated at Q15min intervals via timeOffset

  oxyHistory        { dose, t, note }                                                Updated at Q30min intervals via timeOffset
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
