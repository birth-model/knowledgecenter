**BIRTH MODEL**

PRD Part 3 --- Labs, Immunizations & Genetic Screening

Prenatal Labs · Encounter Labs · Immunizations · LOINC Codes · Flag Logic

*Source: birth-model-unified-complete.html patient data · Birth_Model_All_Labs_and_Immunizations_for_Patients.xlsx*

**Dr. Anish J. Shah, MD, FACOG**

CEO & Founder, Birth Model, Inc. · March 2026

**1. Lab Display Architecture**

Labs render inside the patient handoff drawer using a templated HTML system. Labs are grouped into .lab-section subsections. The pVal(key) helper function reads from patient.prenatalLabs.

**1.1 LAB FLAG CSS CLASSES (.LV VARIANTS)**

  --------------------------------------------------------------------------------------------------
  **CSS Class**       **Color**                     **Meaning**
  ------------------- ----------------------------- ------------------------------------------------
  .lv (default)       var(\--text) dark gray        Normal value, no flag

  .lv.pos             var(\--green)                 Positive result (e.g., immunity confirmed)

  .lv.neg             var(\--muted) muted gray      Negative / not detected (favorable result)

  .lv.flag            var(\--red) #dc2626           Critical abnormal --- requires clinical action

  .lv.lo / .lv.warn   var(\--amber) #d97706         Below normal or borderline warning

  .lv.nil             var(\--soft), italic weight   Not available or pending

  .lv.bt-neutral      var(\--text), bold, 10px      Blood type display --- no color judgment
  --------------------------------------------------------------------------------------------------

**1.2 TREND DISPLAY CLASSES (INSIDE .LV-PREV SPAN)**

  ------------------------------------------------------------------------------------
  **CSS Class**   **Symbol**   **Color**            **Meaning**
  --------------- ------------ -------------------- ----------------------------------
  .arr-up         ↑            #ef4444 red          Value trended up from previous

  .arr-down       ↓            #0a7c52 green        Value trended down from previous

  .arr-eq         →            var(\--soft) muted   Value unchanged from previous
  ------------------------------------------------------------------------------------

**1.3 LAB GRID LAYOUTS**

  ------------------------------------------------------------------------------------------------------------------------------------------
  **CSS Class**   **Columns**   **nth-child border rule**                     **Used For**
  --------------- ------------- --------------------------------------------- --------------------------------------------------------------
  .lab-grid       3 columns     lab-item:nth-child(3n) → border-right: none   Blood Bank, CBC, Infectious Disease, Immunity, Immunizations

  .lab-grid-4     4 columns     lab-item:nth-child(4n) → border-right: none   Coagulation, Preeclampsia Labs
  ------------------------------------------------------------------------------------------------------------------------------------------

**2. Prenatal Labs (patient.prenatalLabs)**

Set during outpatient prenatal care; displayed in handoff drawer. Accessed via pVal(key) helper. Values can be a plain string or an object {value, date}.

**2.1 BLOOD BANK PRENATAL**

  ----------------------------------------------------------------------------------------------------------------------------------------------------------
  **Key**           **Name**            **NM Code**   **Normal**        **Abnormal / Flag Rule**    **Clinical Note**
  ----------------- ------------------- ------------- ----------------- --------------------------- --------------------------------------------------------
  blood_type        ABO/Rh Blood Type   5436          Any type          Rh Negative → .flag class   Rh-neg mothers: RhoGAM at 28w + postpartum if baby Rh+

  antibody_screen   Antibody Screen     5437          Negative → .neg   Positive → .flag            Positive = antibody ID + titers; risk of HDN
  ----------------------------------------------------------------------------------------------------------------------------------------------------------

**2.2 IMMUNITY STATUS**

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Key**     **Name**                **NM Code**   **Normal → CSS**   **Abnormal → CSS**                **Clinical Note**
  ----------- ----------------------- ------------- ------------------ --------------------------------- ---------------------------------------------------------------
  rubella     Rubella IgG             5441          Immune → .neg      Non-immune → .flag                MMR postpartum --- LIVE VACCINE, CI in pregnancy

  rubeola     Rubeola (Measles) IgG   24030         Immune → .neg      Non-immune → flag                 MMR postpartum

  mumps       Mumps IgG               ---           Immune → .neg      Non-immune → (no explicit flag)   MMR postpartum

  varicella   Varicella IgG           24031         Immune → .neg      Non-immune → .flag                Varicella vaccine postpartum; avoid exposure during pregnancy
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------

**2.3 INFECTIOUS DISEASE PRENATAL SCREENING**

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Key**                 **Display Label**          **NM Code**   **Normal → CSS**                     **Abnormal → CSS**                               **Critical Action**
  ----------------------- -------------------------- ------------- ------------------------------------ ------------------------------------------------ ----------------------------------------------------------------------------------------------
  hiv1st                  HIV (1st Tri)              5444          Nonreactive → .neg                   Reactive → (no explicit flag in code)            ART; consider C/S; infant prophylaxis

  hiv3rd                  HIV (3rd Tri)              5444          Nonreactive → .neg                   Reactive                                         Repeat at 36w or admission if high-risk

  hbsag                   HBsAg                      5443          Negative → .neg                      Positive                                         **⚠ Infant needs HBIg + HepB vaccine within 12hrs of birth**

  hepC                    HCV Ab                     5840          Negative → .neg                      Positive                                         Reactive → HCV RNA; infant testing at 18 months

  syphilis                RPR                        5442          Nonreactive → .neg                   Positive                                         **⚠ TREAT BEFORE DELIVERY --- prevent congenital syphilis**

  gonorrhea + chlamydia   GC/CT (combined display)   ---           Both Negative → \"Neg/Neg\" → .neg   Either Positive → show separate values → .flag   Treat immediately; prevents neonatal ophthalmia

  hsv                     HSV 1/2 IgG                ---           Negative → .neg                      Positive → .flag                                 Suppressive therapy at 36w; C/S if active lesions. Only rendered if pVal(\"hsv\") is truthy.

  urine_culture           Urine Culture              ---           No growth                            ≥100K CFU/mL                                     Treat ASB. GBS in urine = GBS+ status for delivery.

  gbs                     GBS (36-37w)               5452          Negative → .neg                      Positive                                         Intrapartum antibiotics (penicillin/ampicillin preferred; see GBS allergy logic)
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  📝 GBS display in the nurse card: p.gbs field + p.gbsDate field. p.antibiotics string is checked for coverage using regex: /gbs\|pcn\|ampicillin\|penicillin/. If positive GBS and no match → gbsStatus = \"needs-abx\" (flag); if match found → gbsStatus = \"covered\" (green).

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**2.4 GLUCOSE SCREENING**

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Key**           **Name**              **Timing**             **Normal**                                        **Abnormal**                                          **Note**
  ----------------- --------------------- ---------------------- ------------------------------------------------- ----------------------------------------------------- ------------------------------------------------------------------------------------
  glucose_screen    1-Hr Glucose Screen   24-28 weeks            Pass (institution threshold: 130/135/140 mg/dL)   Fail → 3-hr GTT needed; ≥200 diagnoses GDM directly   Display: \"Pass (124 mg/dL)\" or \"Fail (156 mg/dL) → GTT needed\"

  gtt_3hr           3-Hour GTT            If 1-hr screen fails   Normal                                            GDM                                                   Abnormal fasting = GDM. Otherwise 2 of 3 timed values abnormal.

  diabetes_status   Diabetes Status       ---                    No Diabetes / GDM Screen Negative                 GDM A1 \| GDM A2 \| Type 2 DM                         A1 = diet only. A2 = insulin/meds. Pregestational = pre-existing before pregnancy.

  hba1c             HbA1c                 ---                    \<5.7%                                            ≥5.7% (prediabetes) / ≥6.5% (diabetes)                ≥6.5% in early pregnancy = likely preexisting diabetes
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**2.5 HEMOGLOBINOPATHY, IRON STUDIES, THYROID**

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Key**                      **Name**                   **Normal**                                               **Abnormal**                                                   **Clinical Note**
  ---------------------------- -------------------------- -------------------------------------------------------- -------------------------------------------------------------- -----------------------------------------------------------
  hemoglobin_electrophoresis   Hgb Electrophoresis        AA (normal)                                              AS (trait) \| SS/SC (disease) \| elevated HbA2 (thalassemia)   Screen high-risk populations. Trait = genetic counseling.

  ferritin                     Ferritin                   ≥30 ng/mL                                                \<30 ng/mL → iron deficient                                    Low ferritin confirms iron deficiency.

  iron_serum                   Iron, Serum                60-170 µg/dL                                             \<60 µg/dL                                                     Used with ferritin for complete iron evaluation.

  tsh                          TSH (pregnancy-specific)   1st tri: 0.1-2.5 \| 2nd: 0.2-3.0 \| 3rd: 0.3-3.0 mIU/L   Outside trimester-specific range                               Ranges differ from non-pregnant normals.

  free_t4                      Free T4                    1st: 0.8-1.7 \| 2nd: 0.7-1.5 \| 3rd: 0.6-1.3 ng/dL       Outside trimester-specific range                               Order with TSH for thyroid dysfunction workup.
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**3. Encounter Labs (patient.labs and patient.preeLabs)**

Labs obtained at admission or during the L&D stay. Stored on patient.labs for CBC/coag/blood bank and patient.preeLabs for preeclampsia panel. All fields support trend display via prev/prevDate fields.

**3.1 BLOOD BANK ENCOUNTER (PATIENT.LABS)**

  -------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Key**                **Display**               **Normal**   **Flag Condition**   **Note**
  ---------------------------- ------------------------- ------------ -------------------- --------------------------------------------------------------------
  (blood type from prenatal)   Blood Type                Any ABO/Rh   Rh-negative          Must verify for potential transfusion.

  antibody_screen_encounter    Antibody Screen           Negative     Positive             Positive = antibody ID needed; risk of transfusion reaction + HDN.

  tsDate                       Last T&S (string field)   ---          ---                  Date of last type and screen. Displayed as patient.labs.tsDate.
  -------------------------------------------------------------------------------------------------------------------------------------------------------------

**3.2 CBC & HEMATOLOGY --- FIELD KEYS IN PATIENT.LABS**

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field**                   **Trend Fields**          **Flag Condition**   **Warn Condition**   **Normal (pregnancy)**
  --------------------------- ------------------------- -------------------- -------------------- ----------------------------------------------------------------------------
  wbc, wbcPrev, wbcPrevDate   wbcPrev + wbcPrevDate     ---                  wbc \> 15 → .warn    5,000-15,000/µL; up to 25,000 in active labor

  hgb, hgbPrev, hgbPrevDate   hgbPrev + hgbPrevDate     ---                  hgb \< 10 → .lo      ≥10 g/dL

  hct, hctPrev                paired with hgb display   ---                  ---                  ≥30%

  plt, pltPrev, pltPrevDate   pltPrev + pltPrevDate     plt \< 100 → .flag   plt \< 150 → .lo     ≥150,000/µL; \<100K = HELLP criterion; \<70K = epidural often CI

  labTime                     ---                       ---                  ---                  Timestamp of CBC draw (string, e.g. \"7:00 AM\"). Shown in section header.
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**3.3 COAGULATION --- FIELD KEYS IN PATIENT.LABS (GRID: .LAB-GRID-4)**

  ----------------------------------------------------------------------------------------------------------------------------------
  **Display**   **Field Keys**   **Trend Fields**              **Normal Range**
  ------------- ---------------- ----------------------------- ---------------------------------------------------------------------
  PT / INR      pt, inr          ptPrev, inrPrev, ptPrevDate   PT 10-13s; INR 0.8-1.2

  PTT           ptt              pttPrev, pttPrevDate          25-35 seconds

  Fibrinogen    fibrinogen       fibPrev (up/down arrow)       ≥200 mg/dL (pregnancy elevated: 300-600). Drop = DIC/abruption/AFE.

  D-Dimer       dDimer           --- (nil/Pending only)        Normally elevated in pregnancy; not reliable for PE/DVT diagnosis
  ----------------------------------------------------------------------------------------------------------------------------------

**3.4 PREECLAMPSIA LABS --- PATIENT.PREELABS (GRID: .LAB-GRID-4; CONDITIONAL: SHOWN IF P.PREE OR P.PREELABS)**

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Key**                              **Display**       **Flag Threshold**      **Trend Fields**                   **Normal**
  ------------------------------------------ ----------------- ----------------------- ---------------------------------- -----------------------------------------------
  mag, magPrev, magPrevTime                  Magnesium Level   ---                     up/down vs prev                    1.8-3.0 mg/dL baseline; monitor on MgSO4 drip

  bun, bunPrev, bunPrevTime                  BUN               ---                     arrow + prev                       ≤12 mg/dL (pregnancy)

  creat, creatPrev, creatPrevTime            Creatinine        creat \> 1.1 → .flag    arrow + prev                       \<0.9 mg/dL in pregnancy

  ast, astPrev, astPrevTime / alt, altPrev   AST / ALT         ast \> 70 → .flag       arrow on AST; alt shown with AST   ≤35 U/L each

  upCr, upCrPrev, upCrPrevTime               Uprot / Cr        upCr \> 0.3 → .flag     arrow + prev                       \<0.3 mg/mg; ≥0.3 = significant proteinuria

  uaProtein, uaProtPrev, uaProtPrevTime      UA Protein        truthy → .flag          arrow (always up)                  Negative/Trace; ≥1+ = quantify with P/C ratio

  egfr, egfrPrev, egfrPrevTime               eGFR              ---                     down=drop concern                  Monitor trajectory

  ldh, ldhPrev, ldhPrevTime                  LDH               ldh \> 600 → .flag      arrow + prev                       ≤250 U/L; \>600 = HELLP criterion

  uricAcid, uricAcidPrev                     Uric Acid         uricAcid \> 6 → .flag   arrow                              \<5.5 mg/dL; elevated = severity marker

  glucose                                    Glucose           ---                     --- (nil)                          ---
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------

**4. Immunizations --- patient.immunizations**

Stored on patient.immunizations. Displayed in two lab subsections (Pregnancy Vaccines and Postpartum/At-Risk) and in the standalone collapsible .immunization-section in the patient drawer.

**4.1 PREGNANCY VACCINES**

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Key**   **Name**                **Timing**                **Green Condition**                      **Flag Condition**                          **Note**
  --------------- ----------------------- ------------------------- ---------------------------------------- ------------------------------------------- -------------------------------------------------------------------------
  tdap            Tdap                    27-36w EACH pregnancy     status === \"Given\"                     status !== \"Given\" → .flag                Required every pregnancy for passive antibody transfer. Not just first.

  flu             Flu Vaccine             Any trimester, Oct-May    status === \"Given\"                     status !== \"Given\" → .flag                Injectable only. LAIV nasal spray contraindicated in pregnancy.

  covid           COVID-19 Vaccine        Any trimester             status === \"Up to date\" OR \"Given\"   other → neutral (no explicit flag)          mRNA vaccines (Pfizer, Moderna). Updated annually.

  rsv             RSV Vaccine (Abrysvo)   32-36w, Sept-Jan season   status === \"Given\"                     not given → infant nirsevimab alternative   Maternal RSV vaccine OR infant nirsevimab --- NOT both.
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**4.2 RHOGAM --- RH PROPHYLAXIS (PATIENT.IMMUNIZATIONS.RHOGAM)**

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Condition to Display**                                                        **Field**              **Normal**                    **Flag Condition**                      **Clinical Note**
  ------------------------------------------------------------------------------- ---------------------- ----------------------------- --------------------------------------- -----------------------------------------------------------------------------------
  pVal(\"bloodType\").includes(\"NEG\") OR patient.bloodType?.includes(\"NEG\")   immunizations.rhogam   status === \"Given\" → .neg   rhogam.needsPostpartum = true → .flag   Give at 28w + within 72hrs postpartum if baby Rh+. Also after sensitizing events.

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

  ----------------------------------------------------------------------------------------------------------------------
  📝 RhoGAM row is ONLY displayed for Rh-negative patients. For Rh-positive patients, this row does not appear at all.

  ----------------------------------------------------------------------------------------------------------------------

**4.3 POSTPARTUM/AT-RISK VACCINES --- DYNAMIC FALLBACK LOGIC**

These are computed from prenatal immunity labs. Check patient.immunizations first; if null, fall back to immunity status from patient.prenatalLabs:

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Field Key**   **Name**              **Fallback Logic (if patient.immunizations.X is null)**                              **Flag Condition**                                               **Note**
  --------------- --------------------- ------------------------------------------------------------------------------------ ---------------------------------------------------------------- ----------------------------------------------------
  mmr             MMR Vaccine           getRubellaImmune() AND getRubeolaImmune() → display \"Immune\"; else \"Needed PP\"   mmrStatus not \"Immune\" and not \"Given\" → .flag               LIVE VACCINE --- postpartum only. CI in pregnancy.

  varicella       Varicella Vaccine     getVaricellaImmune() → display \"Immune\"; else \"Needed PP\"                        varicellaVaxStatus not \"Immune\" and not \"Given\" → .flag      LIVE VACCINE --- postpartum only.

  hepB            Hepatitis B Vaccine   null → display \"Not indicated\"                                                     \"Not indicated\" → .neg (green); not immune + at-risk → .flag   3-dose series for high-risk patients only.
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**4.4 IMMUNIZATION ITEM CSS STATES (.IMM-ITEM --- IN STANDALONE .IMMUNIZATION-SECTION)**

  ----------------------------------------------------------------------------------------------------------------
  **CSS Class**             **Background**   **Border Color**   **Status Text Color**      **Meaning**
  ------------------------- ---------------- ------------------ -------------------------- -----------------------
  .imm-item.given           #f0fdf4          #86efac green      #16a34a green              Vaccine administered

  .imm-item.needed          #fef3c7          #fcd34d yellow     #d97706 amber (bold)       Due --- not yet given

  .imm-item.declined        #fef2f2          #fca5a5 red        #dc2626 red                Patient declined

  .imm-item.unknown         #f3f4f6          #d1d5db gray       #6b7280 gray               Status unknown

  .imm-item.rhogam-needed   #fef3c7          #f59e0b amber      #d97706 amber (bold 700)   RhoGAM overdue/needed
  ----------------------------------------------------------------------------------------------------------------

**5. Genetic Screening --- patient.geneticScreening**

Rendered as a .lab-section at the bottom of the lab panel. Only shown if patient.geneticScreening is truthy. Uses .lab-grid (3-col). Low Risk → .lv.neg; High Risk → .lv.flag.

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Key**                  **Name**               **Timing**    **Low Risk**                                  **High Risk / Flag**                     **Clinical Note**
  ------------------------ ---------------------- ------------- --------------------------------------------- ---------------------------------------- -------------------------------------------------------------------------
  nipt                     NIPT (Cell-Free DNA)   ≥10 weeks     Low Risk → .neg (shows fetalSex if present)   HIGH RISK: T21/T18/T13/Sex Chr → .flag   SCREENING only. Confirm with amnio/CVS. No result = low fetal fraction.

  first_trimester_screen   1st Trimester Screen   11-14 weeks   Low Risk → .neg                               Screen Positive → .flag                  Positive = genetic counseling, consider NIPT or CVS/amnio.

  quad_screen              Quad Screen            15-22 weeks   Low Risk → .neg                               Screen Positive → .flag                  High AFP = rule out NTD. Screen positive = genetic counseling.
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**6. LOINC Codes & NM External Codes Reference**

**6.1 PRENATAL LABS**

  ------------------------------------------------------------------------------------------------------------------------
  **Key**                      **LOINC Codes**                                               **NM Code**   **Priority**
  ---------------------------- ------------------------------------------------------------- ------------- ---------------
  blood_type                   34530-6, 883-9, 10331-7, 882-1, 78873-2                       5436          **essential**

  antibody_screen              890-4, 1007-4, 1250-0, 10360-6                                5437          **essential**

  rubella                      25514-1, 8014-3, 40667-8, 5334-8, 22496-4                     5441          **essential**

  rubeola                      20479-2, 5244-9, 21500-4, 35275-7, 40648-8                    24030         important

  mumps                        6477-4, 7966-5, 22415-4, 40737-9                              ---           supplemental

  varicella                    15410-4, 5403-1, 19162-7, 8046-5                              24031         important

  hiv_1st/3rd_tri              56888-1, 75622-1, 68961-2, 7917-8, 7918-6, 33866-5, 80387-4   5444          **essential**

  hep_b_surface_ag             5196-1, 7905-3, 75410-1, 63557-3                              5443          **essential**

  hep_c                        13955-0, 16128-1, 5199-5, 57006-9, 72376-7                    5840          important

  syphilis                     47238-1, 24110-9, 20507-0, 31147-2, 5292-8, 22461-8           5442          **essential**

  chlamydia                    21613-5, 43304-5, 45084-1, 45095-7, 45076-7, 21190-4          ---           **essential**

  gonorrhea                    24111-7, 43305-2, 45085-8, 45096-5, 45074-2, 21191-2          ---           **essential**

  urine_culture                630-4, 6463-4, 88262-1, 88261-3, 41857-4                      ---           important

  gbs                          11267-2, 86596-4, 580-1, 76588-4, 5098-9, 87921-3             5452          **essential**

  glucose_screen               1504-0, 20438-8, 1518-0                                       ---           **essential**

  gtt_3hr                      1558-6, 1556-0, 1557-8, 1507-3, 76629-5, 77145-1              ---           **essential**

  diabetes_status / hba1c      4548-4, 17856-6, 4549-2, 59261-8                              ---           **essential**

  hemoglobin_electrophoresis   4551-8, 4576-5, 4625-0, 4563-3, 71695-1, 50749-1              ---           important

  ferritin                     2276-4, 20567-4, 24373-3, 14723-1                             ---           supplemental

  tsh                          3016-3, 11580-8, 27975-2                                      ---           supplemental

  free_t4                      3024-7, 14920-3                                               ---           supplemental
  ------------------------------------------------------------------------------------------------------------------------

**6.2 ENCOUNTER LABS**

  ------------------------------------------------------------------------------------------------------
  **Key**                    **LOINC Codes**                               **NM Code**   **Priority**
  -------------------------- --------------------------------------------- ------------- ---------------
  wbc                        6690-2, 26464-8, 804-5                        ---           **essential**

  hemoglobin                 718-7, 20509-6, 30313-1                       ---           **essential**

  hematocrit                 4544-3, 20570-8, 32354-3                      ---           **essential**

  platelets                  777-3, 778-1, 26515-7                         ---           **essential**

  ast                        1920-8, 30239-8                               ---           important

  alt                        1742-6, 1743-4                                ---           important

  ldh                        2532-0, 14804-9                               ---           important

  creatinine                 2160-0, 14682-9, 38483-4                      ---           important

  bun                        3094-0, 6299-2, 14937-7                       ---           supplemental

  uric_acid                  3084-1, 14933-6                               ---           supplemental

  protein_creatinine_ratio   2890-2, 13801-6, 34366-5                      ---           important

  magnesium                  19123-9, 2601-3, 19124-7                      ---           important

  pt                         5902-2, 5964-2                                ---           important

  ptt                        3173-2, 14979-9                               ---           important

  inr                        6301-6, 34714-6                               ---           important

  fibrinogen                 3255-7, 30902-1, 48664-0                      ---           important

  d_dimer                    48065-7, 48066-5, 30240-6                     ---           supplemental

  covid_test                 94500-6, 95406-5, 94845-5, 96119-3, 94762-2   ---           important

  influenza_test             76078-5, 80382-5, 85477-8, 92142-9            ---           supplemental

  rsv_test                   92131-2, 40988-8, 88535-0                     ---           supplemental

  fetal_fibronectin          31993-9, 20407-3, 49245-4                     ---           important

  rom_status                 19295-5, 2753-9, 5802-4, 50595-8, 89321-4     ---           **essential**
  ------------------------------------------------------------------------------------------------------

**6.3 IMMUNIZATIONS LOINC / CVX CODES**

  --------------------------------------------------------------------------------
  **Key**               **LOINC / CVX Codes**                    **Priority**
  --------------------- ---------------------------------------- -----------------
  tdap_vaccine          30952-6, 30546-6, 11458-7                **essential**

  influenza_vaccine     46249-9, 30525-0                         important

  covid_vaccine         91375-6, 96766-1                         important

  rsv_vaccine           96895-8                                  important

  mmr_vaccine           30939-3                                  important

  varicella_vaccine     30942-7                                  supplemental

  hepatitis_b_vaccine   30943-5, 46250-7                         supplemental

  rhogam                LOINC: 11150-3, 890-4 · CVX: 90 (RhIG)   **essential**
  --------------------------------------------------------------------------------
