**BIRTH MODEL**

PRD Part 2 --- Delivery Note

All Fields · Dropdowns · Preference Cards · Note Generation

*Source: birth-model-unified-complete.html --- DN_FILL_OPTIONS, DN_PREF_OPTIONS, note generation functions*

**Dr. Anish J. Shah, MD, FACOG**

CEO & Founder, Birth Model, Inc. · March 2026

**1. Delivery Note Modal Layout**

Opens as full-screen overlay (#deliveryNoteOverlay). Click outside calls closeDeliveryNote(). Scroll position saved/restored on re-render.

**1.1 TOP BAR**

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Region**             **Content**
  ---------------------- ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Left                   Patient name (.dn-patient-name) · MRN (.dn-patient-mrn) · Type badge: .dn-type-cesarean \"CESAREAN\" or .dn-type-vaginal \"VAGINAL\"

  Right                  \"Delivery Note\" label (11px, rgba(255,255,255,0.7)) · Version badge \"vN SENT\" if noteHistory \> 0 · Close ✕ button

  isCesarean detection   \(1\) deliveryMethod.toLowerCase().includes(\"cesarean\"/\"c/s\"/\"cs\") OR (2) patient.status === \"cs\" OR (3) patient.substatus.toLowerCase().includes(\"c-section\"/\"cesarean\")
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**1.2 MISSING FIELDS BANNER**

  ------------------------------------------------------------------------------------------------------------
  **Condition**                              **Banner Text**
  ------------------------------------------ -----------------------------------------------------------------
  getDnMissingFields(patient).length === 0   Banner hidden

  1--5 missing fields                        All labels joined with \", \" + \" --- fill below to complete\"

  \> 5 missing fields                        First 4 labels + \"\...+ N more --- fill below to complete\"
  ------------------------------------------------------------------------------------------------------------

**1.3 PROVIDER STRIP (ALWAYS VISIBLE)**

\"Provider: {patient.provider}\" and \"Practice: {d.practice}\". bg #F9FAFB; font 11px, #6B7280; border-bottom #F3F4F6.

**1.4 COLLAPSIBLE SECTIONS**

  ------------------------------------------------------------------------------------------------------------------------------------------------
  **Section**        **Button Label**                                                     **State Key**                        **Default**
  ------------------ -------------------------------------------------------------------- ------------------------------------ -------------------
  Delivery Data      \"Delivery Data (N to fill)\" \| \"▶ Show\" / \"▼ Hide\"             dnState.dataCollapsed\[patientId\]   true (collapsed)

  Preference Card    \"⚙ {providerName}\'s Preference Card\" \| \"▶ Show\" / \"▼ Hide\"   dnState.prefCollapsed\[patientId\]   true (collapsed)

  Note Body          Always visible --- Edit/Done toggle                                  dnState.editMode\[patientId\]        false (read-only)
  ------------------------------------------------------------------------------------------------------------------------------------------------

**1.5 DATA SECTION --- SUBSECTIONS (HEADER CSS CLASS + FIELDS)**

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Subsection**                  **Header CSS**             **Fields**
  ------------------------------- -------------------------- -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Patient                         .dn-section-label.teal     Name · BMI · Prior C/S · MRN · GBS (fillable via gbs key) · DOB/Age · GTPAL

  Infant 1 Details                .dn-section-label.blue     MRN · Delivery Status (key: deliveryStatus) · GA · Nuchal Cord (key: nuchalCord) · Placenta Disposition · Sex · Apgar 1/5 · Presentation · Placenta Removal · Weight (g) · Delivery Time · Placenta Appearance · Delayed Cord Clamping · Cord Blood

  Infant 2 Details (twins only)   .dn-section-label.blue     Same fields as Infant 1 --- shown only if d.infant2 populated

  Operative (cesarean)            .dn-section-label.red      Col 1: Method (clickable) · Skin Incision · Fluid Color · Anesthesia · Uterine Incision · Sterilization · Indication. Col 2: QBL · Surgeon(s) · Practice. Col 3: Assistant (multitext)

  Delivery Details (vaginal)      .dn-section-label (blue)   Delivery Type (clickable) · Conditional forceps/vacuum/sequential sub-fields · Lacerations · Suture Type (hidden if intact perineum) · Shoulder Dystocia + Maneuvers (if Yes) · QBL · Clinician · Practice
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**1.6 NOTE EDIT MODE**

  ------------------------------------------------------------------------------------------------------------------------------------------
  **Element**        **Behavior**
  ------------------ -----------------------------------------------------------------------------------------------------------------------
  .dn-edit-btn       \"Edit\" / \"Done\" toggle via dnToggleEdit(). On exit (Done): saves innerHTML to dnState.editedContent\[patientId\]

  .dn-note-content   Read-only HTML when not editing. contenteditable=\"true\" + class \"editable\" when in edit mode

  Content priority   1st: dnState.editedContent\[patientId\] if exists. 2nd: regenerate fresh from note function. Manual edits always win.
  ------------------------------------------------------------------------------------------------------------------------------------------

**1.7 DIAGNOSES SECTION**

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Note Type**   **Left Column**                                                                                                           **Right Column**
  --------------- ------------------------------------------------------------------------------------------------------------------------- -----------------------------------------------------------------------------------------
  Cesarean        Preoperative Diagnoses: from d.preopDiagnoses → d.indication → d.csIndication (comma-separated, each = one .dn-dx-item)   Postoperative Diagnoses: from d.postopDiagnoses. Default text: \"Same as preoperative\"

  Vaginal         Admission Diagnoses: from d.admitDiagnoses → d.indication                                                                 Delivery Diagnoses: from d.deliveryDiagnoses
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**1.8 DISCLAIMER & ATTESTATION**

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Element**                **Text**
  -------------------------- -----------------------------------------------------------------------------------------------------------------------------------------
  Disclaimer                 \"This note was generated using Birth Model\'s Clinical Decision Support Software. The clinician has reviewed this note for accuracy.\"

  Attestation label          \"I have reviewed this note and it reflects the care provided\"

  Send button (first send)   Sign & Send --- enabled only when attested. noteHistory = 0.

  Send button (re-send)      Send Updated (vN+1) where N = count of prior sends --- enabled only when attested
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------

**2. DN_FILL_OPTIONS --- All Fillable Fields**

These are the exact field definitions from the DN_FILL_OPTIONS object in the HTML. Each key maps to an input rendered in the Delivery Data section.

**2.1 SHARED FIELDS (APPEAR IN BOTH CESAREAN AND VAGINAL NOTES)**

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Key**         **Label (exact)**       **Type**         **Choices (exact)**
  --------------------- ----------------------- ---------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  gbs                   GBS Status              select           Positive \| Negative \| Unknown \| Inadequately treated \| Adequately treated

  placentaDisposition   Placenta Disposition    select           Discarded \| To Pathology \| To Patient \| To Lab for Cord Blood Banking

  placentaRemoval       Placenta Removal        select           Spontaneous \| Manual Extraction \| Controlled Cord Traction

  placentaAppearance    Placenta Appearance     select           Intact with 3-vessel cord \| Intact with 2-vessel cord \| Intact with accessory lobe \| Fragmented --- explored \| Calcified \| Marginal cord insertion \| Velamentous cord insertion

  qbl                   QBL (mL)                text (numeric)   e.g. 300

  delayedCordClamping   Delayed Cord Clamping   select           30 seconds \| 60 seconds \| 90 seconds \| Not performed \| Cord milking performed

  cordBlood             Cord Blood              select           Collected for type and screen \| Collected for cord blood banking \| Not collected

  presentation          Presentation            select           Vertex \| Vertex, Occiput Anterior (OA) \| Vertex, Occiput Posterior (OP) \| Vertex, Left Occiput Anterior (LOA) \| Vertex, Right Occiput Anterior (ROA) \| Vertex, Left Occiput Transverse (LOT) \| Vertex, Right Occiput Transverse (ROT) \| Vertex, Left Occiput Posterior (LOP) \| Vertex, Right Occiput Posterior (ROP) \| Face \| Brow \| Compound \| Breech \| Frank Breech \| Complete Breech \| Footling Breech \| Transverse Lie

  nuchalCord            Nuchal Cord             select           No \| Yes --- x1 loose, reduced \| Yes --- x1 tight, reduced \| Yes --- x1, clamped and cut \| Yes --- x2 loose, reduced \| Yes --- x2 tight, reduced \| Body cord x1 \| Body cord x2

  deliveryStatus        Delivery Status         select           Living \| Stillborn \| Neonatal demise

  infantSex             Infant Sex              select           Female \| Male \| Ambiguous

  birthWeight           Birth Weight (g)        text (numeric)   e.g. 3200

  apgar1                Apgar 1 min             select           0 \| 1 \| 2 \| 3 \| 4 \| 5 \| 6 \| 7 \| 8 \| 9 \| 10

  apgar5                Apgar 5 min             select           0 \| 1 \| 2 \| 3 \| 4 \| 5 \| 6 \| 7 \| 8 \| 9 \| 10

  deliveryTime          Delivery Time           text             Format: MM/DD/YYYY HH:MM AM/PM
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**2.2 VAGINAL-SPECIFIC FIELDS**

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Key**               **Label (exact)**    **Type**      **Choices (exact)**
  --------------------------- -------------------- ------------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  operativeDelivery           Operative Delivery   select        Spontaneous Vaginal Delivery \| Forceps-Assisted Delivery \| Vacuum-Assisted Delivery \| Sequential (Vacuum then Forceps)

  operativeDeliveryTwinA      Twin A Delivery      select        Spontaneous Vaginal Delivery \| Forceps-Assisted Delivery \| Vacuum-Assisted Delivery \| Sequential (Vacuum then Forceps)

  operativeDeliveryTwinB      Twin B Delivery      select        Spontaneous Vaginal Delivery \| Forceps-Assisted Delivery \| Vacuum-Assisted Delivery \| Sequential (Vacuum then Forceps)

  lacerations                 Laceration Type      multiselect   Intact Perineum \| Periurethral Laceration \| Labial Laceration \| First Degree Perineal Laceration \| Second Degree Perineal Laceration \| Third Degree (3a --- \<50% EAS) \| Third Degree (3b --- \>50% EAS) \| Third Degree (3c --- IAS involvement) \| Fourth Degree Perineal Laceration \| Cervical Laceration \| Vaginal Sidewall Laceration \| Sulcal Tear \| Episiotomy --- Midline \| Episiotomy --- Mediolateral

  sutureType                  Suture Type          select        3-0 Vicryl \| 2-0 Vicryl \| 0 Vicryl \| 4-0 Vicryl \| 3-0 Chromic \| 2-0 Chromic \| 4-0 Chromic \| 4-0 Vicryl Rapide \| 3-0 Vicryl Rapide \| 3-0 Monocryl \| 2-0 Monocryl \| 4-0 Monocryl \| 3-0 PDS \| 2-0 PDS \| 3-0 Polysorb \| 2-0 Polysorb \| None required \| Other

  shoulderDystocia            Shoulder Dystocia    select        No \| Yes

  shoulderDystociaManeuvers   Maneuvers Used       multiselect   McRoberts maneuver \| Suprapubic pressure \| Rubin I \| Rubin II \| Woods screw \| Reverse Woods screw \| Delivery of posterior arm \| Gaskin maneuver (all fours) \| Zavanelli maneuver \| Intentional clavicle fracture
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

  ---------------------------------------------------------------------------------------------------
  📝 Fundal pressure is deliberately ABSENT from the shoulderDystociaManeuvers list. Do not add it.

  ---------------------------------------------------------------------------------------------------

  ----------------------------------------------------------------------------------------------------
  📝 sutureType field is hidden (display: none) when \"Intact Perineum\" is selected in lacerations.

  ----------------------------------------------------------------------------------------------------

**2.3 CONDITIONAL OPERATIVE FIELDS --- FORCEPS (WHEN OPERATIVEDELIVERY INCLUDES \"FORCEPS\" AND NOT \"SEQUENTIAL\")**

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Key**       **Label**            **Choices**
  ------------------- -------------------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  forcepsType         Forceps Type         Simpson forceps \| Tucker-McLane forceps \| Elliott forceps \| Kielland forceps \| Luikart forceps \| Barton forceps \| Kielland (rotation) \| Scanzoni maneuver (Tucker-McLane) \| Piper forceps (aftercoming head) \| Laufe divergent forceps \| Wrigley forceps (outlet) \| Neville-Barnes forceps \| Other

  forcepsIndication   Forceps Indication   Prolonged second stage \| Maternal exhaustion \| Non-reassuring fetal heart rate tracing \| Fetal bradycardia \| Maternal cardiac disease (shorten second stage) \| Maternal pushing contraindicated \| Other
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**2.4 CONDITIONAL OPERATIVE FIELDS --- VACUUM (WHEN OPERATIVEDELIVERY INCLUDES \"VACUUM\" AND NOT \"SEQUENTIAL\")**

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Key**      **Label**            **Choices**
  ------------------ -------------------- ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  vacuumCupType      Vacuum Cup Type      Silastic soft cup \| Rigid plastic cup \| Metal cup \| Kiwi OmniCup \| Kiwi ProCup \| Other

  vacuumIndication   Vacuum Indication    Prolonged second stage \| Maternal exhaustion \| Non-reassuring fetal heart rate tracing \| Fetal bradycardia \| Maternal cardiac disease (shorten second stage) \| Maternal pushing contraindicated \| Other

  vacuumPops         Number of Pop-offs   0 \| 1 \| 2 \| 3
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**2.5 CESAREAN-SPECIFIC OPERATIVE FIELDS**

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Key**     **Label**          **Type**           **Choices**
  ----------------- ------------------ ------------------ ----------------------------------------------------------------------------------------------------------------
  skinIncision      Skin Incision      select             Pfannenstiel \| Vertical midline \| Joel-Cohen \| Maylard \| Previous scar used

  uterineIncision   Uterine Incision   select             Low transverse \| Low vertical \| Classical (vertical) \| T-incision \| J-incision

  anesthesia        Anesthesia         select             Spinal \| Epidural \| Combined spinal-epidural (CSE) \| General anesthesia

  fluidColor        Fluid Color        select             Clear \| Meconium-stained \| Bloody \| Absent

  sterilization     Sterilization      select             None \| Bilateral tubal ligation \| Bilateral salpingectomy \| Unilateral salpingectomy \| Essure (historical)

  csIndication      CS Indication      text (multiline)   Free text --- ICD-10 diagnoses for C/S

  surgeons          Surgeon(s)         text               Provider name(s)

  csAssistant       Assistant          multitext          Can add multiple assistants
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------
