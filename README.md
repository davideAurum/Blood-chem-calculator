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

This is a teaching / convenience tool, not a substitute for clinical judgement.
