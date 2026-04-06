**BIRTH MODEL**

PRD Part 1 --- Platform Overview

Hospitals · Roles · Acuity · Status Colors · App Shell

*Source: birth-model-unified-complete.html --- SECTIONS, roles modal, getAcuity(), CSS status classes*

**Dr. Anish J. Shah, MD, FACOG**

CEO & Founder, Birth Model, Inc. · March 2026

**1. Product Overview**

Birth Model is a real-time Labor & Delivery clinical intelligence platform. It integrates with Epic and Oracle Health via SMART on FHIR, enriching EHR data with predictive models and presenting it in role-optimized interfaces. Live at Northwestern Medicine Prentice Women\'s Hospital and Huntley Hospital.

**1.1 HOSPITALS & SECTIONS**

  --------------------------------------------------------------------------------------------------------------------------------------------------
  **Hospital**                                       **Unit**        **const SECTIONS Array (exact)**
  -------------------------------------------------- --------------- -------------------------------------------------------------------------------
  Northwestern Medicine Prentice Women\'s Hospital   Prentice L&D    \[\"L&D South\", \"L&D North\", \"Triage\", \"Antepartum\", \"OR/Recovery\"\]

  Northwestern Medicine Huntley Hospital             Huntley L&D     \[\"Triage\", \"L&D\", \"OR/Recovery\"\]
  --------------------------------------------------------------------------------------------------------------------------------------------------

**1.2 PRENTICE ROOM NUMBERS (ALL_ROOMS)**

  ---------------------------------------------------------------------------------------
  **Section**            **Rooms**                                            **Count**
  ---------------------- ---------------------------------------------------- -----------
  L&D South              0881-01 through 0896-01                              16 rooms

  L&D North              0861-01 through 0876-01                              16 rooms

  Triage                 0159-01 through 0172-01, 0175-01, 0175-02, 0175-03   17 rooms

  Antepartum             0961-01 through 0997-01                              33 rooms

  OR/Recovery            OR 1--5-10, PHR 1--8                                 13 rooms
  ---------------------------------------------------------------------------------------

**1.3 HUNTLEY ROOM NUMBERS (ALL_ROOMS_HUNTLEY)**

  ---------------------------------------------------------------------------------------------------
  **Section**            **Rooms**                                                        **Count**
  ---------------------- ---------------------------------------------------------------- -----------
  L&D                    3107-1, 3108-1, 3109-1, 3110-1, 3111-1, 3112-1, 3113-1, 3114-1   8 rooms

  Triage                 Triage 1, Triage 2, Triage 3, NST 1, NST 2, NST 3                6 rooms

  OR/Recovery            OR 1, OR 2, Recovery 1, Recovery 2                               4 rooms
  ---------------------------------------------------------------------------------------------------

**1.4 USER ROLES --- EXACT FROM ROLE MODAL (SELECTROLE VALUES)**

  -------------------------------------------------------------------------------------------------------------------------------------------------
  **Button Label**            **selectRole() value**   **Sub-label**       **CSS class**                           **Body class added on login**
  --------------------------- ------------------------ ------------------- --------------------------------------- --------------------------------
  Student                     student                  ---                 role-btn                                ---

  Resident                    resident                 ---                 role-btn                                ---

  Nurse                       nurse                    ---                 role-btn                                (nurse card view)

  Huntley Charge Nurse        huntley                  ---                 role-btn                                huntley-view + census-full

  Charge Nurse                charge                   ---                 role-btn                                charge-view + prentice-tab-all

  L&D Operations              clerk                    ---                 role-btn                                ---

  Motherboard                 motherboard              ---                 role-btn                                motherboard-view + census-full

  Huntley Attending           huntleyAttending         Dr. Rachel Guild    role-btn attending-role                 huntley-attending-view

  Attending                   attending                Dr. Amber Watters   role-btn attending-role                 attending-view

  Clinician (CNM, PA, APRN)   clinician                Simone Rowen, CNM   role-btn attending-role                 attending-view

  TV Screen Login             tvScreen                 Display Mode        role-btn (dark gradient inline style)   (TV mode)
  -------------------------------------------------------------------------------------------------------------------------------------------------

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  📝 TV Screen Login button has inline style: background: linear-gradient(135deg, #1e3d3c 0%, #2d5a58 100%); border: 2px solid #abdfe2. The attending-role buttons show two lines: role-main (14px, 600) + role-sub (11px, muted). Border: 1px solid #c7d2fe; background: #eef2ff.

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**1.5 LOGGED_IN_USER --- DEMO PERSONAS (EXACT FROM HTML)**

  ------------------------------------------------------------------------------------------------------------------
  **Role Key**         **Display Name / Config**
  -------------------- ---------------------------------------------------------------------------------------------
  nurse                Jennifer Smith

  provider (default)   Dr. Patel

  resident             Dr. Sarah Chen, PGY-3, Northwestern Memorial Hospital

  attending            Dr. Amber Watters, Prentice Women\'s Hospital. Practices: NASOG, NM Maternal Fetal Medicine

  huntleyAttending     Dr. Rachel Guild, Huntley Hospital. Practice: Northwestern Medicine RMG Huntley Call Group

  clinician            Simone Rowen, CNM, Erie Family Health, Prentice Women\'s Hospital
  ------------------------------------------------------------------------------------------------------------------

**2. Acuity System --- getAcuity() Function**

getAcuity(p) returns { level, reason, color, isOverride }. Evaluated in strict priority order. All conditions return 1:1 except the final default which returns 1:2.

  -------------------------------------------------------------------------------------------------------------------
  **Priority**             **Condition (exact from HTML)**   **level returned**      **reason**           **color**
  ------------------------ --------------------------------- ----------------------- -------------------- -----------
  0 (override check)       if (p.ratioOverride)              p.ratioOverride value   Manual Override      gray

  1                        if (p.pushing)                    **1:1**                 Pushing              red

  2                        if (p.dilation \>= 10)            **1:1**                 10cm                 red

  3                        if (p.status === \"cs\")          **1:1**                 C-Section            red

  4                        if (p.status === \"delivered\")   **1:1**                 Recovering           orange

  5                        if (p.mag)                        **1:1**                 Magnesium            blue

  6                        if (p.insulin)                    **1:1**                 Insulin drip         orange

  7                        if (p.iufd \|\| p.loss)           **1:1**                 Loss Patient         purple

  8                        if (p.twins \|\| p.triplets)      **1:1**                 Multiple Gestation   blue

  9                        if (p.bmi \> 40)                  **1:1**                 BMI \>40             orange

  Default (none matched)   ---                               1:2                     null                 green
  -------------------------------------------------------------------------------------------------------------------

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  📝 Preeclampsia (p.pree) alone WITHOUT magnesium is NOT automatically 1:1. Explicit HTML comment: \"Note: Preeclampsia without mag is NOT automatically 1:1 per the reference.\"

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Manual ratio toggle: togglePatientRatio(patientId, event) --- if no current override → sets to opposite of calculated. If override exists → clears it. Re-renders via renderSections() + renderStaffing().

**3. Status Color System --- All 14 Statuses**

All 14 status CSS classes share the same hex values across every component. The isHighRisk formula used to determine status-high-risk DIFFERS by component --- see the warning below.

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **⚠ ENGINEERING NOTE: isHighRisk is calculated by TWO different formulas depending on which component renders the card. See Section 3.2 for the charge board formula and Section 3.3 for the nurse card formula. These produce different results for the same patient in edge cases.**

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**3.1 STATUS COLOR PRIORITY CHAIN (APPLIES TO ALL COMPONENTS)**

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Priority**   **Condition**                                                                                                 **CSS Class Assigned**   **Label**
  -------------- ------------------------------------------------------------------------------------------------------------- ------------------------ --------------------------
  1              if (p.pph)                                                                                                    status-pph               PPH

  2              else if (p.iufd)                                                                                              status-iufd              IUFD

  3              else if (isHighRisk && !isDelivered) --- see Sec 3.2/3.3 for formula                                          status-high-risk         High Risk

  4              else if (isDelivered && isCSDelivery) --- delivery.type includes \"C/S\" or \"CS\"                            status-delivered-cs      Delivered C/S

  5              else if (isDelivered) --- any non-CS delivery                                                                 status-delivered-vag     Delivered Vaginal

  6              else if (p.status === \"postpartum\" \|\| p.section === \"OR/Recovery\")                                      status-postpartum        Postpartum / OR Recovery

  7              else if (p.pushing)                                                                                           status-pushing           Pushing

  8              else if (p.tolac)                                                                                             status-vtol              VTOL (TOLAC)

  9              else if (p.status === \"induction\" \|\| p.substatus === \"Induction\" \|\| p.substatus?.includes(\"IOL\"))   status-induction         Induction (IOL)

  10             else if (p.status === \"cs\" \|\| p.substatus?.includes(\"Scheduled C/S\"))                                   status-scheduled-cs      Scheduled C-Section

  11             else if (p.status === \"antepartum\")                                                                         status-antepartum        Antepartum

  12             else if (p.status === \"labor\" \|\| p.status === \"active\")                                                 status-laboring          Laboring

  13             else if (p.section === \"Triage\" \|\| p.status === \"triage\")                                               status-observe           Observe / Triage

  14             else if (p.substatus?.includes(\"Procedure\"))                                                                status-procedure         Procedure

  Default        else                                                                                                          status-observe           Observe (default)
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**3.2 ISHIGHRISK FORMULA --- CHARGE BOARD, MB CARD, HANDOFF, SNIPPETS (MOST COMPONENTS)**

Used in: renderPtCard(), motherboard rendering, attending grid, handoff drawer, patient snippets. Simple boolean computed inline:

  --------------------------------------------------------------------------------------------------------------
  **Component context**         **Formula (exact JavaScript)**
  ----------------------------- --------------------------------------------------------------------------------
  renderPtCard (charge board)   const isHighRisk = (p.pree && p.mag) \|\| p.triplets \|\| (p.twins && p.pree);

  MB card render                const isHighRisk = (p.pree && p.mag) \|\| p.triplets \|\| (p.twins && p.pree);

  Attending grid render         const isHighRisk = (p.pree && p.mag) \|\| p.triplets \|\| (p.twins && p.pree);

  Handoff / snippet render      const isHighRisk = (p.pree && p.mag) \|\| p.triplets \|\| (p.twins && p.pree);
  --------------------------------------------------------------------------------------------------------------

**3.3 ISHIGHRISK FORMULA --- NURSE CARD ONLY (RENDERNURSECARD)**

The nurse card uses a riskFactors counter instead of the simple formula. This is a different calculation path than all other components.

  -------------------------------------------------------------------------------------------------------------------------------
  **Condition checked**                                **Effect**                            **Threshold**
  ---------------------------------------------------- ------------------------------------- ------------------------------------
  p.warnings?.some(w =\> w.includes(\"CATEGORY 1\"))   isHighRisk = true immediately         ---

  p.pree                                               riskFactors++                         ---

  p.mag                                                riskFactors++                         ---

  p.iufd                                               isHighRisk = true AND riskFactors++   Immediate flag + counter

  p.twins \|\| p.triplets                              riskFactors++                         ---

  p.tolac                                              riskFactors++                         ---

  p.pphRisk === \"High\"                               riskFactors++                         Reads stored patient.pphRisk field

  riskFactors \>= 2                                    isHighRisk = true                     Counter threshold
  -------------------------------------------------------------------------------------------------------------------------------

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **⚠ ENGINEERING NOTE: A patient with TOLAC + pphRisk=\"High\" (but no pree/mag/triplets) will show status-high-risk on the NURSE CARD but status-vtol on the CHARGE BOARD. This is an existing inconsistency in the codebase --- document but do not silently correct without coordinating with engineering.**

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**3.4 ISDELIVERED --- CONSISTENT ACROSS ALL COMPONENTS**

isDelivered = p.status === \"delivered\". This formula is identical in every render function.

**3.5 ISCSDELIVERY --- TWO VARIANTS (SAME LOGICAL RESULT, SLIGHTLY DIFFERENT IMPLEMENTATION)**

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Component**                                    **isCSDelivery check**
  ------------------------------------------------ ------------------------------------------------------------------------------------------------------------------------------------------
  Charge board, MB, attending, handoff, snippets   p.delivery?.type?.includes(\"C/S\") \|\| p.delivery?.type?.includes(\"CS\")

  Nurse card (renderNurseCard)                     p.delivery?.type?.includes(\"C/S\") \|\| p.delivery?.type?.includes(\"CS\") \|\| p.delivery?.type?.toLowerCase().includes(\"c-section\")
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  📝 The nurse card adds the .toLowerCase().includes(\"c-section\") check. Functionally the same result for all current patient data, but engineers adding new delivery types should be aware both checks exist.

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**3.6 STATUS COLOR HEX VALUES & BACKGROUND COLORS**

  -------------------------------------------------------------------------------------------------------------------------------------------------------
  **CSS Class**          **Border Hex**   **Background Hex**   **Trigger**                                                                 **Priority**
  ---------------------- ---------------- -------------------- --------------------------------------------------------------------------- --------------
  status-pph             #ff170f          #fad8d8              p.pph truthy                                                                1 (HIGHEST)

  status-iufd            #808080          #e8e8e8              p.iufd = true                                                               2

  status-high-risk       #00e6ff          #d5f5fa              isHighRisk && !isDelivered (formula differs by component --- see 3.2/3.3)   3

  status-delivered-cs    #c000c0          #f5d8f5              isDelivered && isCSDelivery                                                 4

  status-delivered-vag   #00a201          #d5f5d8              isDelivered (non-CS)                                                        5

  status-postpartum      #c04001          #f5e2d5              p.status === \"postpartum\" OR p.section === \"OR/Recovery\"                6

  status-pushing         #4169e1          #dce5f9              p.pushing = true                                                            7

  status-vtol            #c0c0ff          #e5e5fc              p.tolac = true                                                              8

  status-induction       #d4b68b          #f5ead8              p.status === \"induction\" OR substatus Induction/IOL                       9

  status-scheduled-cs    #dda0dd          #f5e0f5              p.status === \"cs\" OR substatus \"Scheduled C/S\"                          10

  status-antepartum      #22b2aa          #d9f4f2              p.status === \"antepartum\"                                                 11

  status-laboring        #fff002          #fdfad5              p.status === \"labor\" OR \"active\"                                        12

  status-observe         #d4d400          #f9f8dc              Triage section OR p.status === \"triage\" OR default fallback               13/Default

  status-procedure       #4b0382          #ebe0f5              p.substatus?.includes(\"Procedure\")                                        14
  -------------------------------------------------------------------------------------------------------------------------------------------------------

**3.7 BORDER THICKNESS BY COMPONENT**

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **CSS Selector**                              **Element**                               **Border Rule**
  --------------------------------------------- ----------------------------------------- -----------------------------------------------------------------------------------
  .pt-card.status-\*                            Charge Board patient card                 border: 2px solid {hex}; border-left: 4px solid {hex}; background: {bg}

  body.motherboard-view .mb-card.status-\*      Motherboard card                          border: 2px solid {hex}; background: {bg}

  .attending-patients-grid .mb-card.status-\*   Attending grid card                       border: 2px solid {hex}; background: {bg}

  .attending-section-body .mb-card.status-\*    Attending section card                    border: 2px solid {hex}; background: {bg}

  body.laborist-board-mode .mb-card.status-\*   Laborist board card                       border: 2px solid {hex}; background: {bg}

  .nurse-card-v2.mb-card.status-\*              Nurse card (DOM class is nurse-card-v2)   border: 2px solid {hex}; background: {bg}

  .unit-patient-item.status-\*                  Unit panel list item                      border-left: 3px solid {hex}; background: {bg}

  .patient-snippet.status-\*                    Patient snippet                           border: 2px solid {hex}; background: {bg}

  Drawer header (inline style)                  panel-header                              style=\"background:{bg}; border-left: 4px solid {hex};\" --- via getStatusColor()
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**4. PPH Risk Score --- Attending / Postpartum Card**

This is the PPH risk scoring used specifically in the attending card and postpartum badge display. It is computed at render time from live patient data --- it is NOT the same as patient.pphRisk (which is the stored AWHONN admission risk field). See Section 5 for the distinction.

  --------------------------------------------------------------------------------------------------------------
  **Check (exact order)**        **Points Added**
  ------------------------------ -------------------------------------------------------------------------------
  if (p.pree)                    riskFactors += 1

  if (p.mag)                     riskFactors += 1

  if (p.twins \|\| p.triplets)   riskFactors += 1

  if (p.pph)                     riskFactors += 2

  if (p.qbl \>= 500)             riskFactors += 1

  if (p.qbl \>= 1000)            riskFactors += 2 (cumulative with the \>=500 check: total +3 if qbl \>= 1000)

  if (p.pitocin \> 0)            riskFactors += 1

  if (p.bmi \>= 35)              riskFactors += 1

  if (p.bmi \>= 40)              riskFactors += 1 (cumulative with \>=35 check: total +2 if bmi \>= 40)
  --------------------------------------------------------------------------------------------------------------

  -------------------------------------------------------------------------------------------------------
  **riskFactors Total**                   **pphRisk Output**   **CSS class**   **Display**
  --------------------------------------- -------------------- --------------- --------------------------
  0                                       Low                  (none)          No badge or default text

  ≥ 1                                     Medium               warning         Amber warning badge

  ≥ 3 OR p.pph truthy OR p.qbl \>= 1000   High                 critical        Red critical badge
  -------------------------------------------------------------------------------------------------------

**5. patient.pphRisk Field vs Render-Time PPH Score --- Clarification**

  -----------------------------------------------------------------------------------------------------------------
  **⚠ ENGINEERING NOTE: There are TWO separate PPH risk concepts. Conflating them causes incorrect UI behavior.**

  -----------------------------------------------------------------------------------------------------------------

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Concept**                 **Field / Variable**                                          **Source**                                                                                                         **Used By**                                                                                                                                          **Writes Back?**
  --------------------------- ------------------------------------------------------------- ------------------------------------------------------------------------------------------------------------------ ---------------------------------------------------------------------------------------------------------------------------------------------------- --------------------------------------------------
  AWHONN Admission PPH Risk   patient.pphRisk (string: \"Low\" \| \"Medium\" \| \"High\")   Set by nurse at admission using AWHONN stratification tool                                                         \(1\) Nurse card high-risk counter: pphRisk=\"High\" adds +1 to riskFactors in renderNurseCard (2) Displayed as stored risk level in handoff views   No --- read-only at render time

  Render-Time PPH Score       Local pphRisk variable (computed)                             Calculated from current clinical data (pree/mag/twins/pph/qbl/pitocin/bmi) inside attending card render function   Attending card PPH risk badge only                                                                                                                   Never --- local variable, discarded after render
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**6. App Shell**

**6.1 HEADER**

  ------------------------------------------------------------------------------------------------------------------------------------
  **Element**         **Spec**
  ------------------- ----------------------------------------------------------------------------------------------------------------
  Height              56px fixed

  Background          #F1F6F6

  Border-bottom       1px solid #D1E0E0

  Left                Birth Model logo img (height 28px) · Unit name · Hospital name with dropdown toggle (toggleHospitalDropdown())

  Center              Context-dependent title/tabs. flex-1, justify right, margin-right: 16px

  Right               Date + time display (updateDateTime()) · Role badge · User menu button

  Hospital dropdown   .header-hospital-dropdown --- toggles .open class; shows switchToMotherboard() option
  ------------------------------------------------------------------------------------------------------------------------------------

**6.2 ROLE MODAL**

  -----------------------------------------------------------------------------------------------------------------------------------------------
  **Property**    **Value**
  --------------- -------------------------------------------------------------------------------------------------------------------------------
  Title           \"Select Your Role\"

  Subtitle        \"Choose how you\'d like to view the L&D unit\"

  Grid            4-column CSS grid, 12px gap

  On selection    selectRole(role) → sets currentRole → localStorage(\"birthmodel_role\") → hides modal → shows appContainer → renderSections()

  Auto-login      On page load: reads localStorage(\"birthmodel_role\") → selectRole(savedRole, true) if found
  -----------------------------------------------------------------------------------------------------------------------------------------------

**6.3 USER MENU**

  -------------------------------------------------------------------------------------------------------------------------
  **Property**     **Value**
  ---------------- --------------------------------------------------------------------------------------------------------
  Trigger          .user-menu-btn click → toggles .open on .user-menu

  Width            240px; right-aligned; 8px border-radius

  Header section   Label \"SIGNED IN AS\" (11px uppercase) · User name (14px bold) · Role description (12px)

  Items            Switch Role (reopens modal) · Motherboard profile switcher (hidden unless motherboard-view) · Sign Out
  -------------------------------------------------------------------------------------------------------------------------
