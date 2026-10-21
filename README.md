# IEL MCQ Practice

A static multiple-choice practice site for first-year **International and European Law (IEL)** students.
No server, no database and no build step: three files published on GitHub Pages.

| File | Role |
|---|---|
| `index_v2.html` | The whole application (markup, styles, logic). |
| `questions_v2.js` | The question bank and the module configuration. Loaded by the page. |
| `questions_v2.csv` | The same bank in spreadsheet form, for editing and review. |
| `README_v2.md` | This file. |
| `REVIEW_v2.md` | Source mapping and the list of points to verify before each exam. |

---

## 1. What the site does

* **Pick a module** → **pick how much material** → **pick a set size** → answer.
* Questions **and answer positions are reshuffled every session**, so the position of the correct answer carries no information.
* **Four options, exactly one correct answer.** At least one wrong option in each newly written question is a deliberate near-miss of the correct one.
* **Every question has an explanation**, shown in the review screen together with a "why your answer was wrong" note.
* **Set sizes:** 10, 20, 30, 50, everything, plus an *Exam format* button that matches the real paper for the selected exam. Options larger than the filtered pool are disabled automatically.
* **Cumulative exams:** choosing *Exam 2* also serves everything from Exam 1, which matches how the programme examines.
* Flagging, a question palette, free navigation, progress bar, keyboard shortcuts (`←` `→`, `1`–`4`, `F`), per-topic performance breakdown, "retry the ones I missed", dark mode, and automatic session saving on the device.

### Current contents

| Module | Questions | Exam 1 pool | Exam 2 pool | Exam 3 pool |
|---|---|---|---|---|
| Introduction to Law | 312 | 312 | — | — |
| Private Law | 117 | 66 | 117 | 117 |
| International Law | 94 | 34 | 94 | 94 |
| **Total** | **523** | | | |

Exam 3 currently returns the same pool as Exam 2 because no exam-3-only material (tort, property, public international law, human rights) has been taught yet. As soon as it is, tag it `"examStage": 3` and the Exam 3 pool grows on its own.

---

## 2. Publishing on GitHub Pages

1. Put the files in the repository root.
2. **Rename `index_v2.html` to `index.html`** (GitHub Pages serves `index.html` as the home page). Keep the original `index.html` from the previous project in an `archive/` folder if you want to preserve it.
3. If you rename the HTML file, make sure the script tag still matches the bank filename:
   ```html
   <script src="questions_v2.js"></script>
   ```
4. Commit and push to the default branch.
5. **Settings → Pages → Build and deployment → Deploy from a branch**, select the branch and the `/ (root)` folder, then save.
6. The site appears at `https://<username>.github.io/<repository>/` after a minute or two.

> **Opening the file locally:** browsers block `file://` script loading in some configurations. If the page says the bank did not load, run a local server instead:
> `python3 -m http.server` and open `http://localhost:8000`.

---

## 3. Adding and editing questions

### 3.1 The record format

```js
{
  "id": "PRIV-118",                       // must be unique
  "module": "private",                     // intro | private | international | <new>
  "subject": "Private Law",
  "topic": "Remedies",                     // shown as a chip; drives the topic filter
  "subtopic": "Specific performance",       // optional
  "examStage": 3,                           // the exam this material first appears in
  "cumulative": true,                       // later exams also draw on it
  "difficulty": "medium",                  // easy | medium | hard
  "sourceType": "course material",
  "sourceRef": "Private Law — Remedies",   // shown in the review screen
  "legacySource": "",                       // internal traceability only, never displayed
  "stem": "…question text…",
  "choices": ["correct answer", "near-miss distractor", "distractor", "distractor"],
  "correct": 0,                             // ALWAYS 0 — the app shuffles
  "explanation": "…why the correct answer is correct…",
  "status": "new",
  "version": "2.0"
}
```

**Rules that keep the bank consistent**

1. `correct` is **always `0`**. The correct answer is written first and the app shuffles the four options at runtime, so students never see an order pattern.
2. Exactly **four** choices, all non-empty and all different.
3. At least one distractor should be a **plausible near-miss** — the distinction the question is actually testing (for example *disapplication* vs *annulment*, *sufficient* vs *adequate*, *dispatch* vs *receipt*, *void* vs *voidable*).
4. Never refer inside a question to the notes, a workshop, a lecture, a week number, a slide deck or a teacher. Students take the modules in different orders, and the questions must stand alone.
5. Do not reuse the fictional states from the open-ended class exercises. Invent a fresh fictional state where a scenario is needed.
6. Paraphrase the textbooks; do not quote them at length.

### 3.2 Adding a few questions

Open `questions_v2.js`, copy any object inside `window.QUESTION_BANK`, edit it, give it a new `id`. Save and reload. **Nothing else needs to change** — every total in the interface is counted at runtime.

### 3.3 Adding many questions from the spreadsheet

Edit `questions_v2.csv` (columns: `id, module, subject, topic, subtopic, exam_stage, cumulative, difficulty, source_type, source_ref, stem, choice_correct, choice_wrong_1, choice_wrong_2, choice_wrong_3, explanation, legacy_source, status, version`), then regenerate the JavaScript:

```python
import csv, json
rows = list(csv.DictReader(open('questions_v2.csv', encoding='utf-8')))
bank = [{
    "id": r["id"], "module": r["module"], "subject": r["subject"],
    "topic": r["topic"], "subtopic": r["subtopic"],
    "examStage": int(r["exam_stage"]), "cumulative": r["cumulative"] == "yes",
    "difficulty": r["difficulty"], "sourceType": r["source_type"],
    "sourceRef": r["source_ref"], "legacySource": r["legacy_source"],
    "stem": r["stem"],
    "choices": [r["choice_correct"], r["choice_wrong_1"], r["choice_wrong_2"], r["choice_wrong_3"]],
    "correct": 0, "explanation": r["explanation"],
    "status": r["status"], "version": r["version"],
} for r in rows]

head = open('questions_v2.js', encoding='utf-8').read().split('window.QUESTION_BANK')[0]
open('questions_v2.js', 'w', encoding='utf-8').write(
    head + 'window.QUESTION_BANK = ' + json.dumps(bank, indent=2, ensure_ascii=False) + ';\n')
```

### 3.4 Adding a new exam round

Add the questions with the new `examStage` number, then add one entry to the module's `exams` array in `window.MODULE_CONFIG`:

```js
{ "stage": 4, "label": "Exam 4", "questions": 30 }
```

The new filter button, its availability count and its *Exam format* size option appear automatically.

### 3.5 Adding a whole new module

1. Write questions with a new `module` key, for example `"european"`.
2. Add one block to `window.MODULE_CONFIG`:

```js
{
  "key": "european",
  "name": "European Law",
  "short": "European Law",
  "blurb": "Institutions, competences and the internal market.",
  "exams": [
    { "stage": 1, "label": "Exam 1", "questions": 15 },
    { "stage": 2, "label": "Exam 2", "questions": 25 }
  ]
}
```

A new card appears on the home screen. No change to `index_v2.html` is required. A module with zero questions shows as disabled with a prompt to add rows, so you can register a module before writing its content.

---

## 4. Keeping the bank healthy

Run this check after a batch of edits:

```python
import json, re, collections
src = open('questions_v2.js', encoding='utf-8').read()
bank = json.loads(src[src.index('window.QUESTION_BANK') :].split('=', 1)[1].rstrip().rstrip(';'))
banned = re.compile(r'workshop|the notes|\bslides?\b|professor|lecturer', re.I)
ids = collections.Counter(q['id'] for q in bank)
for q in bank:
    assert len(q['choices']) == 4 and len(set(q['choices'])) == 4, q['id']
    assert q['correct'] == 0, q['id']
    assert ids[q['id']] == 1, q['id']
    assert not banned.search(q['stem'] + q['explanation'] + ''.join(q['choices'])), q['id']
print(len(bank), 'questions OK')
```

---

## 5. Credits and scope

Questions are written from the taught course material, with the prescribed textbooks used only to clarify concepts and resolve ambiguities. The Introduction to Law questions are carried over unchanged in substance from the earlier project; only their source labels were made neutral so that they do not depend on the order in which classes are taken.

This is a student-made study aid. It is not legal advice and it does not replace the prescribed materials or the official assessment information for the modules.
