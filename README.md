# ABG

A simple command-line arterial / venous blood gas analyzer. Requires Python 3.

## Usage

```bash
./ABG 7.30 50 80 24                 # pH pCO2 pO2 HCO3 (positional)
./ABG --ph 7.30 --pco2 50 --po2 80 --hco3 24
./ABG                               # interactive prompts
./ABG 7.31 50 40 25 --vbg           # venous sample
./ABG 7.20 24 100 10 --na 140 --cl 100 --albumin 3.0   # adds anion gap / delta ratio
./ABG 7.47 30 60 22 --age 60 --fio2 0.21               # adds A-a gradient
```

`./ABG` works from Git Bash (shebang script) and from PowerShell / cmd (via `ABG.cmd`).

## What it reports

1. **pH status**: acidemia (< 7.35), alkalemia (> 7.45), or normal.
2. **Primary disorder**: respiratory or metabolic, acidosis or alkalosis, or a
   mixed disorder when both push pH the same way. With a normal pH and abnormal
   pCO2/HCO3, the side of 7.40 the pH sits on picks the primary process.
3. **Compensation** and whether it is adequate:
   - Metabolic acidosis: Winter's formula, pCO2 = 1.5 x HCO3 + 8 (+/- 2)
   - Metabolic alkalosis: pCO2 = 40 + 0.7 x (HCO3 - 24) (+/- 5)
   - Respiratory acidosis: HCO3 rises 1 (acute) or 3.5 (chronic) per 10 mmHg pCO2
   - Respiratory alkalosis: HCO3 falls 2 (acute) or 4 (chronic) per 10 mmHg pCO2

   A measured value outside the expected range is reported as a concurrent
   second disorder (e.g. "metabolic acidosis with concurrent respiratory acidosis").
4. **Oxygenation**: pO2 grading (normal / mild / moderate / severe hypoxaemia),
   with an optional A-a gradient. Venous pO2 is flagged as non-diagnostic.
5. **Sanity check**: HCO3 is recomputed from pH and pCO2 (Henderson-Hasselbalch)
   and a warning is printed if the entered value disagrees by more than 3 mEq/L.

Reference ranges: pCO2 35-45 mmHg, HCO3 22-26 mEq/L.

# MetAcid

A metabolic acidosis calculator that works through the standard workup. It
uses the same thresholds as `ABG` wherever the two tools overlap.

## Usage

```bash
./MetAcid 7.20 24 10 140 100 --albumin 3.0     # pH pCO2 HCO3 Na Cl (positional)
./MetAcid 18 140 112                           # HCO3 Na Cl only (no blood gas)
./MetAcid                                      # interactive prompts
./MetAcid 7.10 20 8 140 98 --osm 330 --glucose 90 --bun 14     # osmolar gap
./MetAcid 7.30 30 14 138 114 --k 3.1 --una 40 --uk 30 --ucl 100 # urine anion gap
./MetAcid 7.10 20 6 140 110 --weight 70                        # bicarbonate deficit
```

`./MetAcid` works from Git Bash and from PowerShell / cmd (via `MetAcid.cmd`).
Glucose, BUN and ethanol are in mg/dL, and albumin is in g/dL.

## What it reports

1. **Whether a metabolic acidosis is present**: either a low HCO3 (< 22) or a
   raised anion gap (which catches a HAGMA hidden by a metabolic alkalosis).
2. **Respiratory compensation** (needs pCO2): Winter's formula,
   pCO2 = 1.5 x HCO3 + 8 (+/- 2). A pCO2 outside that range is reported as a
   concurrent respiratory acidosis or alkalosis.
3. **Anion gap**: Na - (Cl + HCO3), normal ~8-12. With `--albumin` it is
   corrected by + 2.5 x (4.0 - albumin). A raised gap gets a **delta ratio**
   to detect a coexisting normal-AG acidosis or metabolic alkalosis.
4. **Osmolar gap** (`--osm --glucose --bun`, optional `--ethanol`): measured -
   (2 x Na + glucose/18 + BUN/2.8 [+ EtOH/3.7]). A gap > 10 is flagged, and a
   HAGMA with a raised osmolar gap is flagged as a possible toxic alcohol.
5. **Urine anion gap** (`--una --uk --ucl`): UNa + UK - UCl. It is only
   interpreted in a normal-AG acidosis: a negative gap suggests GI loss, and a
   positive gap suggests a renal cause. With `--k`, a positive gap is split
   into type 1 RTA (low K) and type 4 RTA (high K).
6. **Causes to consider** for each type of acidosis (GOLDMARK for a high AG),
   with suggested tests.
7. **Bicarbonate deficit** (`--weight`, optional `--target-hco3`, default 24):
   0.5 x weight x (target - HCO3). This is for reference only and is not a
   dosing instruction.

MetAcid also flags inconsistent pH/pCO2/HCO3 values, a missing albumin when
the anion gap is normal, and a low HCO3 that may be compensation for a
respiratory alkalosis.

These are teaching / convenience tools, not a substitute for clinical judgement.
