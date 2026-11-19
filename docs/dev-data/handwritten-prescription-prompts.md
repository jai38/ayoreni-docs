# Handwritten prescription image prompts

Generated from `myhealthapp-backend/src/scripts/seed-dev/handwritten-templates.ts` by `npm run seed:prompts`. Do not edit by hand: change the template file and regenerate, so each image keeps matching its seeded record.

## How to use

1. Use **Gemini Nano Banana Pro** (Gemini 3 Pro Image) in the Gemini app or AI Studio. If a medicine name comes out misspelt after two tries, try the newest GPT image model for that one prompt.
2. Paste one prompt per chat. Ask for portrait 3:4. Regenerate until every medicine line, number and date matches the prompt exactly: the seeded record expects exactly that text.
3. Save each image as `HW-01.png` ... `HW-20.png` (PNG or JPEG only; convert WebP or HEIC first). Keep each file under 15 MB and under 8000 px per side.
4. Load them:
   - Local: `npm run seed:dev -- --target dev --reset --images <folder>`
   - AWS dev: upload the files to the dev bucket under `seed-assets/` (see `docs/dev-data/README.md`), then run the dev seed task again. Every seeded document for that template then shows the image.
5. Images are also good real-upload tests: upload one through the app in dev and compare Gemini's draft with the record in the table below.

| ID | Scenario | What it tests | Date on paper | Medicines |
|---|---|---|---|---|
| HW-01 | Clean, neat GP prescription | Happy path: every field readable, three common medicines. | 14/10/2026 | 3 |
| HW-02 | Classic doctor cursive | Extraction under heavy cursive; at least one medicine should be flagged uncertain. | 3.9.26 | 3 |
| HW-03 | No date anywhere | Missing date: record keeps date unknown and flags it; timeline must not invent one. | (none) | 3 |
| HW-04 | Ambiguous date order | Day-month vs month-day: 04/07/25 could be 4 July or 7 April; must be flagged uncertain. | 04/07/25 | 2 |
| HW-05 | Long chronic list, elderly | Many medicines on one page (8 lines), chronic refills, insulin units, a weekly-only medicine. | 21-08-2026 | 8 |
| HW-06 | Steroid taper | One medicine whose dose changes over time; must not become a simple daily reminder. | 2/11/2026 | 3 |
| HW-07 | Hindi and English mixed | Devanagari instructions next to English medicine names; names must be kept as written. | १२/६/२०२४ (12/6/2024) | 3 |
| HW-08 | Partly illegible line | A medicine name nobody can read: normalized name must stay empty, never guessed. | 9/1/23 | 2 |
| HW-09 | Eye clinic drops, both eyes | Ophthalmic route, per-eye directions (RE/LE/BE), frequent drops. | 29 Feb 2024 | 2 |
| HW-10 | Hospital discharge summary, two pages | Multi-page document; IV and IM medicines given in hospital vs medicines to take home. | DOA 12/03/2025  DOD 18/03/2025 | 5 |
| HW-11 | Old, faded prescription from 2003 | Very old date, faded ink, old brand (Rantac); timeline far in the past. | 17-5-2003 | 2 |
| HW-12 | Doctor wrote a future date | Date after today (writing slip); must be flagged, not accepted as fact. | 05/01/2027 | 3 |
| HW-13 | Government hospital OPD slip | Tiny slip, printed form fields, generic names only, OPD number. | 08/08/2022 | 2 |
| HW-14 | Skin clinic, topical only | Creams and ointments: topical route, "apply" frequencies, body-site directions. | 11/4/2026 | 3 |
| HW-15 | Weekly methotrexate | A weekly-only medicine next to daily ones; must never be scheduled daily. | 19.10.2026 | 3 |
| HW-16 | Pregnancy (antenatal) visit | Pregnancy context for later safety checks; vaginal route; "till 12 weeks" durations. | 01-09-2026 | 3 |
| HW-17 | Crossed-out and corrected dose | A dose changed by hand on the same page; the final dose must win. | 07/07/2026 | 2 |
| HW-18 | Carbon copy, very faint | Low-contrast carbon-copy text; extraction may fail or flag everything. | 22/12/2019 | 2 |
| HW-19 | Very short, two words | Minimum content: one medicine, no diagnosis, no clinic. | 2/2/21 | 1 |
| HW-20 | Insulin and injection chart | Sliding-scale style units and mixed insulin; units not milligrams. | 15/05/2026 | 2 |

## HW-01: Clean, neat GP prescription

Tests: Happy path: every field readable, three common medicines.

Expected record: date 2026-10-14; doctor Dr. R. K. Sharma, MBBS, MD (Med); diagnosis Viral fever; 3 medicine(s).

```text
A realistic smartphone photo of a handwritten Indian doctor's prescription. Portrait, 3:4.
Paper and handwriting: Neat, upright blue ballpoint handwriting on a white A5 letterhead pad.
Header: a printed clinic letterhead at the top (clinic name, the doctor's name and degrees, a made-up address, and "Reg. No. SAMPLE-0000"). Clinic: "City Care Clinic, Bengaluru". Doctor: "Dr. R. K. Sharma, MBBS, MD (Med)".
Handwritten on the page, in this order:
- Date: "14/10/2026"
- Patient line: "Mr. Rahul Verma  34/M"
- Diagnosis / complaint: "Viral fever"
- A large "Rx" symbol, then these medicines, each on its own line, exactly as written:
1. Tab. Dolo 650  —  1-1-1  —  3 days  (after food)
2. Tab. Cetzine 10mg  —  0-0-1  —  5 days
3. Cap. Becosules Z  —  1-0-0  —  15 days
- Advice at the bottom: "Plenty of fluids. Review if fever > 3 days."
- A scribbled doctor's signature at the bottom right.
Photo: Phone photo, flat on a wooden table, good daylight, slight shadow of the phone at the bottom.
Keep every medicine name, number and date exactly as given (spelling, dashes, fractions). Do not add any other medicines, logos of real hospitals or real brands of paper. All people and clinics are fictional. No real QR codes or barcodes.
```

## HW-02: Classic doctor cursive

Tests: Extraction under heavy cursive; at least one medicine should be flagged uncertain.

Expected record: date 2026-09-03; doctor Dr. B. N. Ghosh, MBBS, MD; diagnosis URTI; 3 medicine(s), 1 flagged unclear.

```text
A realistic smartphone photo of a handwritten Indian doctor's prescription. Portrait, 3:4.
Paper and handwriting: Fast, slanted, joined-up black ink cursive typical of a busy OPD; letters run together, the third line is barely legible.
Header: a printed clinic letterhead at the top (clinic name, the doctor's name and degrees, a made-up address, and "Reg. No. SAMPLE-0000"). Clinic: "Ghosh Clinic, Salt Lake". Doctor: "Dr. B. N. Ghosh, MBBS, MD".
Handwritten on the page, in this order:
- Date: "3.9.26"
- Patient line: "Smt. Rekha Das  52/F"
- Diagnosis / complaint: "URTI"
- A large "Rx" symbol, then these medicines, each on its own line, exactly as written:
1. Azithral 500  —  1-0-0  —  3d  (1 hr before food)
2. Montair LC  —  0-0-1  —  10d
3. Ascoril LS syp  —  10ml TDS  —  5d   [this line should be hard to read]
- A scribbled doctor's signature at the bottom right.
Photo: Handheld phone photo at a slight angle, indoor tube-light, faint glare at the top right.
Keep every medicine name, number and date exactly as given (spelling, dashes, fractions). Do not add any other medicines, logos of real hospitals or real brands of paper. All people and clinics are fictional. No real QR codes or barcodes.
```

## HW-03: No date anywhere

Tests: Missing date: record keeps date unknown and flags it; timeline must not invent one.

Expected record: date unknown (flagged uncertain); doctor Dr. M. A. Qureshi, MBBS; diagnosis Acute gastroenteritis; 3 medicine(s).

```text
A realistic smartphone photo of a handwritten Indian doctor's prescription. Portrait, 3:4.
Paper and handwriting: Plain white paper with a purple rubber stamp of the clinic name; blue ink; no date field filled.
Header: no printed header; instead a slightly smudged rubber stamp with the clinic name and "Reg. No. SAMPLE-0000". Clinic: "Qureshi Dawakhana". Doctor: "Dr. M. A. Qureshi, MBBS".
Handwritten on the page, in this order:
- Date: leave the date blank (no date anywhere on the page)
- Patient line: "Farhan  28 yrs"
- Diagnosis / complaint: "Acute gastroenteritis"
- A large "Rx" symbol, then these medicines, each on its own line, exactly as written:
1. Tab Norflox-TZ  —  1-0-1  —  5 days
2. ORS  —  after each loose motion  —  till better
3. Tab Ondem 4mg  —  SOS  —  3 days  (for vomiting)
- A scribbled doctor's signature at the bottom right.
Photo: Photo on a bedsheet, warm yellow bulb light, paper slightly curled.
Keep every medicine name, number and date exactly as given (spelling, dashes, fractions). Do not add any other medicines, logos of real hospitals or real brands of paper. All people and clinics are fictional. No real QR codes or barcodes.
```

## HW-04: Ambiguous date order

Tests: Day-month vs month-day: 04/07/25 could be 4 July or 7 April; must be flagged uncertain.

Expected record: date 2025-07-04 (flagged uncertain); doctor Dr. Priya Nair, MBBS, DCH; diagnosis Fever with cold; 2 medicine(s).

```text
A realistic smartphone photo of a handwritten Indian doctor's prescription. Portrait, 3:4.
Paper and handwriting: Paediatric letterhead with a small cartoon logo; neat but small handwriting in blue; weight written next to the name.
Header: a printed clinic letterhead at the top (clinic name, the doctor's name and degrees, a made-up address, and "Reg. No. SAMPLE-0000"). Clinic: "Sunrise Children Hospital, Kochi". Doctor: "Dr. Priya Nair, MBBS, DCH".
Handwritten on the page, in this order:
- Date: "04/07/25"
- Patient line: "Baby Aadhya Menon  3 yrs / 13 kg"
- Diagnosis / complaint: "Fever with cold"
- A large "Rx" symbol, then these medicines, each on its own line, exactly as written:
1. Calpol (250) syp  —  5 ml 6 hrly  —  SOS fever  (only if temp > 100)
2. Septilin syp  —  2.5 ml BD  —  15 days
- A scribbled doctor's signature at the bottom right.
Photo: Top-down photo, good light, a child's toy partly visible at one edge.
Keep every medicine name, number and date exactly as given (spelling, dashes, fractions). Do not add any other medicines, logos of real hospitals or real brands of paper. All people and clinics are fictional. No real QR codes or barcodes.
```

## HW-05: Long chronic list, elderly

Tests: Many medicines on one page (8 lines), chronic refills, insulin units, a weekly-only medicine.

Expected record: date 2026-08-21; doctor Dr. Harleen Kaur, MD (Endo); diagnosis Type 2 diabetes, hypertension, CAD, hypothyroidism; 8 medicine(s).

```text
A realistic smartphone photo of a handwritten Indian doctor's prescription. Portrait, 3:4.
Paper and handwriting: Hospital letterhead, two columns; tidy but compressed blue handwriting to fit 8 lines; ticks beside some lines.
Header: a printed clinic letterhead at the top (clinic name, the doctor's name and degrees, a made-up address, and "Reg. No. SAMPLE-0000"). Clinic: "Fortis OPD, Mohali". Doctor: "Dr. Harleen Kaur, MD (Endo)".
Handwritten on the page, in this order:
- Date: "21-08-2026"
- Patient line: "Sardar Gurpreet Singh  78/M"
- Diagnosis / complaint: "T2DM, HTN, CAD, Hypothyroid"
- A large "Rx" symbol, then these medicines, each on its own line, exactly as written:
1. Glycomet GP1  —  1-0-1  —  cont.
2. Inj Lantus  —  14 U at bedtime  —  cont.
3. Telma 40  —  1-0-0  —  cont.
4. Ecosprin 75  —  0-1-0  —  cont.
5. Atorva 20  —  0-0-1  —  cont.
6. Thyronorm 75mcg  —  1-0-0 empty stomach  —  cont.
7. Uprise D3 60K  —  once a week  —  8 weeks
8. Shelcal 500  —  0-1-0  —  3 months
- Advice at the bottom: "FBS/PPBS, HbA1c after 3 months. Next visit with reports."
- A scribbled doctor's signature at the bottom right.
Photo: Scanned-looking photo, slightly overexposed, top edge cut off a little.
Keep every medicine name, number and date exactly as given (spelling, dashes, fractions). Do not add any other medicines, logos of real hospitals or real brands of paper. All people and clinics are fictional. No real QR codes or barcodes.
```

## HW-06: Steroid taper

Tests: One medicine whose dose changes over time; must not become a simple daily reminder.

Expected record: date 2026-11-02; doctor Dr. Lata Kulkarni, MD (Chest); diagnosis Asthma exacerbation; 3 medicine(s).

```text
A realistic smartphone photo of a handwritten Indian doctor's prescription. Portrait, 3:4.
Paper and handwriting: Blue ink; the taper is written as a small staircase diagram with arrows; one word crossed out and rewritten.
Header: a printed clinic letterhead at the top (clinic name, the doctor's name and degrees, a made-up address, and "Reg. No. SAMPLE-0000"). Clinic: "Ruby Hall Clinic, Pune". Doctor: "Dr. Lata Kulkarni, MD (Chest)".
Handwritten on the page, in this order:
- Date: "2/11/2026"
- Patient line: "Ms. Sneha Kulkarni  41/F"
- Diagnosis / complaint: "Acute exacerbation of asthma"
- A large "Rx" symbol, then these medicines, each on its own line, exactly as written:
1. Tab Wysolone 20  —  2-0-0 x5d, 1½-0-0 x5d, 1-0-0 x5d, ½-0-0 x5d  —  20 days  (after breakfast)
2. Foracort 200 inhaler  —  2 puffs BD  —  cont.  (rinse mouth)
3. Asthalin inh  —  2 puffs SOS  —  SOS
- A scribbled doctor's signature at the bottom right.
Photo: Phone photo held in hand, thumb visible at the edge, mild motion blur.
Keep every medicine name, number and date exactly as given (spelling, dashes, fractions). Do not add any other medicines, logos of real hospitals or real brands of paper. All people and clinics are fictional. No real QR codes or barcodes.
```

## HW-07: Hindi and English mixed

Tests: Devanagari instructions next to English medicine names; names must be kept as written.

Expected record: date 2024-06-12; doctor Dr. Ramesh; diagnosis Joint pain; 3 medicine(s).

```text
A realistic smartphone photo of a handwritten Indian doctor's prescription. Portrait, 3:4.
Paper and handwriting: Plain paper with a blue stamp; instructions in Hindi (Devanagari), medicine names in English capitals.
Header: no printed header; instead a slightly smudged rubber stamp with the clinic name and "Reg. No. SAMPLE-0000". Clinic: "Shree Sai Polyclinic, Indore". Doctor: "Dr. Ramesh".
Handwritten on the page, in this order:
- Date: "१२/६/२०२४ (12/6/2024)"
- Patient line: "श्रीमती कमला देवी  65 वर्ष"
- Diagnosis / complaint: "जोड़ों में दर्द (joint pain)"
- A large "Rx" symbol, then these medicines, each on its own line, exactly as written:
1. Zerodol-SP  —  1-0-1  —  5 दिन  (खाने के बाद)
2. Pan 40  —  1-0-0  —  5 दिन  (खाली पेट)
3. Shelcal  —  0-1-0  —  1 माह
- A scribbled doctor's signature at the bottom right.
Photo: Photo on a steel plate tray, harsh overhead light, small water stain on the paper.
Keep every medicine name, number and date exactly as given (spelling, dashes, fractions). Do not add any other medicines, logos of real hospitals or real brands of paper. All people and clinics are fictional. No real QR codes or barcodes.
```

## HW-08: Partly illegible line

Tests: A medicine name nobody can read: normalized name must stay empty, never guessed.

Expected record: date 2023-01-09; doctor Dr. S.K.; diagnosis Cough for one week; 2 medicine(s), 1 flagged unclear.

```text
A realistic smartphone photo of a handwritten Indian doctor's prescription. Portrait, 3:4.
Paper and handwriting: Torn half-sheet of a notepad, pencil, first medicine name is a scribble that cannot be read.
Header: no header at all; plain paper. Doctor: "Dr. S.K.".
Handwritten on the page, in this order:
- Date: "9/1/23"
- Patient line: "Mohd. Imran  45"
- Diagnosis / complaint: "c/o cough 1 wk"
- A large "Rx" symbol, then these medicines, each on its own line, exactly as written:
1. Syp. ??? (illegible) 5ml  —  TDS  —  ?   [this line should be hard to read]
2. Tab Cetzine  —  HS  —  5 days
- A scribbled doctor's signature at the bottom right.
Photo: Low-light phone photo, grainy, slightly out of focus.
Keep every medicine name, number and date exactly as given (spelling, dashes, fractions). Do not add any other medicines, logos of real hospitals or real brands of paper. All people and clinics are fictional. No real QR codes or barcodes.
```

## HW-09: Eye clinic drops, both eyes

Tests: Ophthalmic route, per-eye directions (RE/LE/BE), frequent drops.

Expected record: date 2024-02-29; doctor Dr. Kavita Rao, MS (Ophth); diagnosis Bacterial conjunctivitis, both eyes; 2 medicine(s).

```text
A realistic smartphone photo of a handwritten Indian doctor's prescription. Portrait, 3:4.
Paper and handwriting: Eye-hospital letterhead with an eye diagram where the doctor marked both eyes; neat blue handwriting.
Header: a printed clinic letterhead at the top (clinic name, the doctor's name and degrees, a made-up address, and "Reg. No. SAMPLE-0000"). Clinic: "Lotus Eye Care, Chennai". Doctor: "Dr. Kavita Rao, MS (Ophth)".
Handwritten on the page, in this order:
- Date: "29 Feb 2024"
- Patient line: "Mr. Venkatesh Pillai  59/M"
- Diagnosis / complaint: "Bacterial conjunctivitis BE"
- A large "Rx" symbol, then these medicines, each on its own line, exactly as written:
1. Moxicip e/d  —  1 drop 4 times BE  —  7 days
2. Refresh Tears  —  1 drop 6 times BE  —  1 month
- A scribbled doctor's signature at the bottom right.
Photo: Bright photo, slight perspective skew.
Keep every medicine name, number and date exactly as given (spelling, dashes, fractions). Do not add any other medicines, logos of real hospitals or real brands of paper. All people and clinics are fictional. No real QR codes or barcodes.
```

## HW-10: Hospital discharge summary, two pages

Tests: Multi-page document; IV and IM medicines given in hospital vs medicines to take home.

Expected record: date 2025-03-18; doctor Dr. S. Venkataraman, MD, DM (Cardio); diagnosis NSTEMI, after PTCA to LAD; 5 medicine(s).

```text
A realistic smartphone photo of a handwritten Indian doctor's prescription. Portrait, 3:4.
Paper and handwriting: Two handwritten pages on hospital discharge forms with printed headings; page 1 history and in-hospital treatment, page 2 discharge medicines.
Header: a printed clinic letterhead at the top (clinic name, the doctor's name and degrees, a made-up address, and "Reg. No. SAMPLE-0000"). Clinic: "Manipal Hospital, Old Airport Road". Doctor: "Dr. S. Venkataraman, MD, DM (Cardio)".
Handwritten on the page, in this order:
- Date: "DOA 12/03/2025  DOD 18/03/2025"
- Patient line: "Mr. Ashok Rao  67/M  IP No. SAMPLE-0001"
- Diagnosis / complaint: "NSTEMI, s/p PTCA to LAD"
- A large "Rx" symbol, then these medicines, each on its own line, exactly as written:
1. Tab Ecosprin 75  —  0-1-0  —  lifelong
2. Tab Clopilet 75  —  0-1-0  —  1 year
3. Tab Atorva 40  —  0-0-1  —  cont.
4. Tab Metolar XR 25  —  1-0-0  —  cont.
5. Tab Pan 40  —  1-0-0  —  1 month
- Advice at the bottom: "Cardiac rehab. Review in cardiology OPD after 1 week. Report chest pain immediately."
- A scribbled doctor's signature at the bottom right.
Photo: Two separate photos (one per page) placed side by side, paper stapled at the corner.
Keep every medicine name, number and date exactly as given (spelling, dashes, fractions). Do not add any other medicines, logos of real hospitals or real brands of paper. All people and clinics are fictional. No real QR codes or barcodes.
```

## HW-11: Old, faded prescription from 2003

Tests: Very old date, faded ink, old brand (Rantac); timeline far in the past.

Expected record: date 2003-05-17; doctor Dr. Anita Deshmukh, MBBS, DGO; diagnosis Acidity; 2 medicine(s).

```text
A realistic smartphone photo of a handwritten Indian doctor's prescription. Portrait, 3:4.
Paper and handwriting: Yellowed paper with brown foxing spots, faded blue fountain-pen ink, folded into four with deep creases.
Header: a printed clinic letterhead at the top (clinic name, the doctor's name and degrees, a made-up address, and "Reg. No. SAMPLE-0000"). Clinic: "Deshmukh Nursing Home, Nagpur". Doctor: "Dr. Anita Deshmukh, MBBS, DGO".
Handwritten on the page, in this order:
- Date: "17-5-2003"
- Patient line: "Mrs. Meera Joshi  29/F"
- Diagnosis / complaint: "Acidity"
- A large "Rx" symbol, then these medicines, each on its own line, exactly as written:
1. Tab Rantac 150  —  1-0-1  —  7 days
2. Cap Becosules  —  1-0-0  —  1 month
- A scribbled doctor's signature at the bottom right.
Photo: Photo of the unfolded paper, creases casting shadows, one corner missing.
Keep every medicine name, number and date exactly as given (spelling, dashes, fractions). Do not add any other medicines, logos of real hospitals or real brands of paper. All people and clinics are fictional. No real QR codes or barcodes.
```

## HW-12: Doctor wrote a future date

Tests: Date after today (writing slip); must be flagged, not accepted as fact.

Expected record: date 2027-01-05 (flagged uncertain); doctor Dr. J. Fernandes, MS (Ortho); diagnosis Low back pain; 3 medicine(s).

```text
A realistic smartphone photo of a handwritten Indian doctor's prescription. Portrait, 3:4.
Paper and handwriting: Letterhead with a spine illustration; confident blue handwriting; the year is clearly 2027.
Header: a printed clinic letterhead at the top (clinic name, the doctor's name and degrees, a made-up address, and "Reg. No. SAMPLE-0000"). Clinic: "Fernandes Ortho Clinic, Panaji". Doctor: "Dr. J. Fernandes, MS (Ortho)".
Handwritten on the page, in this order:
- Date: "05/01/2027"
- Patient line: "Mr. Joseph D'Souza  47/M"
- Diagnosis / complaint: "Low back pain"
- A large "Rx" symbol, then these medicines, each on its own line, exactly as written:
1. Zerodol SP  —  1-0-1  —  5 days
2. Pregeb M 75  —  0-0-1  —  1 month
3. Pan 40  —  1-0-0  —  5 days
- A scribbled doctor's signature at the bottom right.
Photo: Good photo, slight rotation.
Keep every medicine name, number and date exactly as given (spelling, dashes, fractions). Do not add any other medicines, logos of real hospitals or real brands of paper. All people and clinics are fictional. No real QR codes or barcodes.
```

## HW-13: Government hospital OPD slip

Tests: Tiny slip, printed form fields, generic names only, OPD number.

Expected record: date 2022-08-08; doctor none; diagnosis Fever; 2 medicine(s), 1 flagged unclear.

```text
A realistic smartphone photo of a handwritten Indian doctor's prescription. Portrait, 3:4.
Paper and handwriting: Small grey recycled-paper OPD slip with printed Marathi and English form headings; doctor name field left blank; dispensary rubber stamp.
Header: a small printed government OPD slip with form fields (Name, Age/Sex, OPD No., Date, Rx). Clinic: "Govt. District Hospital, Nashik". No doctor name anywhere.
Handwritten on the page, in this order:
- Date: "08/08/2022"
- Patient line: "Ganesh Pawar  M/38  OPD No. SAMPLE-4471"
- Diagnosis / complaint: "Fever"
- A large "Rx" symbol, then these medicines, each on its own line, exactly as written:
1. T. PCM 500  —  1-1-1  —  3 days
2. T. CPM  —  0-0-1  —  3 days   [this line should be hard to read]
- A scribbled doctor's signature at the bottom right.
Photo: Photo with the slip held between fingers against a dark background.
Keep every medicine name, number and date exactly as given (spelling, dashes, fractions). Do not add any other medicines, logos of real hospitals or real brands of paper. All people and clinics are fictional. No real QR codes or barcodes.
```

## HW-14: Skin clinic, topical only

Tests: Creams and ointments: topical route, "apply" frequencies, body-site directions.

Expected record: date 2026-04-11; doctor Dr. Arvind Joshi, MD (Derm); diagnosis Tinea corporis; 3 medicine(s).

```text
A realistic smartphone photo of a handwritten Indian doctor's prescription. Portrait, 3:4.
Paper and handwriting: Modern minimalist letterhead, green ink, a small body outline with the rash area shaded.
Header: a printed clinic letterhead at the top (clinic name, the doctor's name and degrees, a made-up address, and "Reg. No. SAMPLE-0000"). Clinic: "Kaya Skin Centre, Hyderabad". Doctor: "Dr. Arvind Joshi, MD (Derm)".
Handwritten on the page, in this order:
- Date: "11/4/2026"
- Patient line: "Ms. Ananya Reddy  23/F"
- Diagnosis / complaint: "Tinea corporis"
- A large "Rx" symbol, then these medicines, each on its own line, exactly as written:
1. Lulifin oint  —  LA OD  —  4 weeks  (on rash + 2 cm around)
2. Candid-B  —  LA BD  —  2 weeks  (not on face)
3. Tab Cetzine  —  HS  —  10 days
- A scribbled doctor's signature at the bottom right.
Photo: Clean, well-lit photo.
Keep every medicine name, number and date exactly as given (spelling, dashes, fractions). Do not add any other medicines, logos of real hospitals or real brands of paper. All people and clinics are fictional. No real QR codes or barcodes.
```

## HW-15: Weekly methotrexate

Tests: A weekly-only medicine next to daily ones; must never be scheduled daily.

Expected record: date 2026-10-19; doctor Dr. T. Reddy, DM (Rheum); diagnosis Rheumatoid arthritis; 3 medicine(s).

```text
A realistic smartphone photo of a handwritten Indian doctor's prescription. Portrait, 3:4.
Paper and handwriting: Blue ink; the word WEEKLY underlined twice and boxed; neat handwriting.
Header: a printed clinic letterhead at the top (clinic name, the doctor's name and degrees, a made-up address, and "Reg. No. SAMPLE-0000"). Clinic: "Medanta, Gurugram". Doctor: "Dr. T. Reddy, DM (Rheum)".
Handwritten on the page, in this order:
- Date: "19.10.2026"
- Patient line: "Mrs. Lakshmi Iyer  56/F"
- Diagnosis / complaint: "Rheumatoid arthritis"
- A large "Rx" symbol, then these medicines, each on its own line, exactly as written:
1. Tab Folitrax 7.5  —  ONCE A WEEK - SUNDAY only  —  cont.  (NOT DAILY)
2. Tab Folvite 5mg  —  Monday only  —  cont.
3. Tab HCQS 200  —  1-0-1  —  cont.
- A scribbled doctor's signature at the bottom right.
Photo: Photo with a pill strip lying on the paper, partly covering the bottom margin.
Keep every medicine name, number and date exactly as given (spelling, dashes, fractions). Do not add any other medicines, logos of real hospitals or real brands of paper. All people and clinics are fictional. No real QR codes or barcodes.
```

## HW-16: Pregnancy (antenatal) visit

Tests: Pregnancy context for later safety checks; vaginal route; "till 12 weeks" durations.

Expected record: date 2026-09-01; doctor Dr. Anita Deshmukh, MBBS, DGO; diagnosis Early pregnancy; 3 medicine(s).

```text
A realistic smartphone photo of a handwritten Indian doctor's prescription. Portrait, 3:4.
Paper and handwriting: Letterhead with a mother-and-child logo; blue ink.
Header: a printed clinic letterhead at the top (clinic name, the doctor's name and degrees, a made-up address, and "Reg. No. SAMPLE-0000"). Clinic: "Deshmukh Nursing Home, Nagpur". Doctor: "Dr. Anita Deshmukh, MBBS, DGO".
Handwritten on the page, in this order:
- Date: "01-09-2026"
- Patient line: "Mrs. Pooja Shah  29/F  G2P1, 8 wks"
- Diagnosis / complaint: "Early pregnancy, threatened miscarriage"
- A large "Rx" symbol, then these medicines, each on its own line, exactly as written:
1. Susten 200  —  0-0-1 PV  —  till 12 wks
2. Duphaston  —  1-0-1  —  till 12 wks
3. Folvite 5  —  1-0-0  —  3 months
- Advice at the bottom: "Bed rest. USG after 2 weeks."
- A scribbled doctor's signature at the bottom right.
Photo: Photo with the paper on a hospital file folder.
Keep every medicine name, number and date exactly as given (spelling, dashes, fractions). Do not add any other medicines, logos of real hospitals or real brands of paper. All people and clinics are fictional. No real QR codes or barcodes.
```

## HW-17: Crossed-out and corrected dose

Tests: A dose changed by hand on the same page; the final dose must win.

Expected record: date 2026-07-07; doctor Dr. Neha Agarwal, MD (Psych); diagnosis Generalised anxiety disorder; 2 medicine(s).

```text
A realistic smartphone photo of a handwritten Indian doctor's prescription. Portrait, 3:4.
Paper and handwriting: The strength "5" is crossed out and "10" written above with the doctor's initials beside it.
Header: a printed clinic letterhead at the top (clinic name, the doctor's name and degrees, a made-up address, and "Reg. No. SAMPLE-0000"). Clinic: "Mind Matters Clinic, Saket". Doctor: "Dr. Neha Agarwal, MD (Psych)".
Handwritten on the page, in this order:
- Date: "07/07/2026"
- Patient line: "Mr. Tarun Malhotra  31/M"
- Diagnosis / complaint: "GAD"
- A large "Rx" symbol, then these medicines, each on its own line, exactly as written:
1. Tab Nexito 10 (5 struck through)  —  0-0-1  —  3 months
2. Tab Lonazep 0.25  —  SOS, max 1/day  —  2 weeks
- A scribbled doctor's signature at the bottom right.
Photo: Photo under a desk lamp, warm tint.
Keep every medicine name, number and date exactly as given (spelling, dashes, fractions). Do not add any other medicines, logos of real hospitals or real brands of paper. All people and clinics are fictional. No real QR codes or barcodes.
```

## HW-18: Carbon copy, very faint

Tests: Low-contrast carbon-copy text; extraction may fail or flag everything.

Expected record: date 2019-12-22; doctor Dr. P. Subramanian, MS (ENT); diagnosis Otitis externa, left ear; 2 medicine(s), 2 flagged unclear.

```text
A realistic smartphone photo of a handwritten Indian doctor's prescription. Portrait, 3:4.
Paper and handwriting: Thin pink carbon-copy paper; the writing is a faint blue-purple impression with smudges.
Header: a printed clinic letterhead at the top (clinic name, the doctor's name and degrees, a made-up address, and "Reg. No. SAMPLE-0000"). Clinic: "Subramanian ENT Clinic, Madurai". Doctor: "Dr. P. Subramanian, MS (ENT)".
Handwritten on the page, in this order:
- Date: "22/12/2019"
- Patient line: "Revathi S.  44/F"
- Diagnosis / complaint: "Otitis externa LE"
- A large "Rx" symbol, then these medicines, each on its own line, exactly as written:
1. Ciplox-D ear drops  —  3 drops TDS LE  —  7 days   [this line should be hard to read]
2. Combiflam  —  SOS  —  3 days   [this line should be hard to read]
- A scribbled doctor's signature at the bottom right.
Photo: Photo on a white surface, slightly blurred.
Keep every medicine name, number and date exactly as given (spelling, dashes, fractions). Do not add any other medicines, logos of real hospitals or real brands of paper. All people and clinics are fictional. No real QR codes or barcodes.
```

## HW-19: Very short, two words

Tests: Minimum content: one medicine, no diagnosis, no clinic.

Expected record: date 2021-02-02; doctor none; diagnosis none; 1 medicine(s).

```text
A realistic smartphone photo of a handwritten Indian doctor's prescription. Portrait, 3:4.
Paper and handwriting: Back of a pharmacy bill; two lines of black ink.
Header: no header at all; plain paper. No doctor name anywhere.
Handwritten on the page, in this order:
- Date: "2/2/21"
- Patient line: "Arjun"
- No diagnosis written
- A large "Rx" symbol, then these medicines, each on its own line, exactly as written:
1. Forcan 150 stat  —  stat  —  single dose
- A scribbled doctor's signature at the bottom right.
Photo: Quick, slightly tilted photo.
Keep every medicine name, number and date exactly as given (spelling, dashes, fractions). Do not add any other medicines, logos of real hospitals or real brands of paper. All people and clinics are fictional. No real QR codes or barcodes.
```

## HW-20: Insulin and injection chart

Tests: Sliding-scale style units and mixed insulin; units not milligrams.

Expected record: date 2026-05-15; doctor Dr. Harleen Kaur, MD (Endo); diagnosis Uncontrolled type 2 diabetes; 2 medicine(s).

```text
A realistic smartphone photo of a handwritten Indian doctor's prescription. Portrait, 3:4.
Paper and handwriting: Letterhead with a hand-drawn table: columns Breakfast / Lunch / Dinner with unit numbers.
Header: a printed clinic letterhead at the top (clinic name, the doctor's name and degrees, a made-up address, and "Reg. No. SAMPLE-0000"). Clinic: "Fortis OPD, Mohali". Doctor: "Dr. Harleen Kaur, MD (Endo)".
Handwritten on the page, in this order:
- Date: "15/05/2026"
- Patient line: "Mr. Sanjay Gupta  61/M"
- Diagnosis / complaint: "Uncontrolled T2DM"
- A large "Rx" symbol, then these medicines, each on its own line, exactly as written:
1. Inj Human Mixtard 30/70  —  18 U before breakfast, 10 U before dinner  —  cont.
2. Glycomet 500  —  1-0-1  —  cont.
- Advice at the bottom: "Check sugar before each insulin dose. If < 70, take 3 glucose tablets."
- A scribbled doctor's signature at the bottom right.
Photo: Photo with a glucometer lying beside the paper.
Keep every medicine name, number and date exactly as given (spelling, dashes, fractions). Do not add any other medicines, logos of real hospitals or real brands of paper. All people and clinics are fictional. No real QR codes or barcodes.
```
