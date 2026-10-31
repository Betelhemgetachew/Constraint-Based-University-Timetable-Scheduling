# Constraint Satisfaction Lab Scheduler for Data Science Courses

An automatic timetable generator that assigns Data Science and Analytics course sections at **USIU-Africa** to a day, time slot, lab and lecturer, with no clashes. It treats timetabling as a **Constraint Satisfaction Problem (CSP)** and solves it with **backtracking search**.

Developed for **DSA 2020B – Introduction to Artificial Intelligence**, School of Science and Technology, USIU-Africa.

---

## Overview

The scheduler was first built to place **65 course sections** (40+ courses, some with several sections) from the STA, DSA, MTH, APT and IST departments into **5 computer labs** in the Lillian K. Beam Building.

It was later extended into an interactive **Streamlit web app**. Users can enter their own courses, lecturers and number of labs, so the tool can handle changing staff, varying semester loads and other campus buildings.

## Features

- Generates a complete, conflict-free weekly lab timetable
- Pairs double-session classes automatically on **Mon/Wed** or **Tue/Thu**
- Accepts custom course lists, lecturers per department and lab count
- Lets you set the maximum number of back-to-back sessions per lecturer
- Optionally shuffles domains to produce different valid timetables
- Searches the generated timetable by course code
- Shows a summary of sessions per day
- Exports the timetable as CSV

## Repository Contents

| File | Description |
|------|-------------|
| `Classroom_Scheduler.ipynb` | Original notebook: problem definition, variables, constraints, the backtracking solver and a sample run with full documentation |
| `app.py` | Streamlit web app that wraps the solver in an interactive interface with custom inputs, search and CSV export |

## Weekly Time Model

| Days | Class type | Time slots | Session length |
|------|-----------|------------|----------------|
| Mon – Thu | Double session (twice a week, paired Mon/Wed or Tue/Thu) | 7:00–8:40, 9:00–10:40, 11:00–12:40, 1:20–3:00, 3:30–5:10, 5:30–7:10, 7:30–9:00 | 1 h 40 min |
| Fri | Full session (once a week) | 8:00–11:20, 1:20–4:40, 5:40–9:00 | 3 h 20 min |
| Sat | Full session (once a week) | 9:00–12:20, 1:20–4:40 | 3 h 20 min |

## How It Works

### CSP formulation

- **Variables:** each course section (e.g. `DSA1080_A`)
- **Domain:** every possible 4-tuple `(day, time_slot, lab, lecturer)`. Lecturers come from the department matching the course prefix (`DSA1080_A` → DSA lecturers).

  Domain size = time slots across all days × labs × lecturers in that department

### Constraints

| Constraint | Function | Rule |
|-----------|----------|------|
| Room conflict | `room_conflict()` | No two sections in the same lab at the same day and time |
| Lecturer conflict | `lecturer_conflict()` | A lecturer cannot teach two sections at the same time |
| Back-to-back limit | `back_to_back_ok()` | A lecturer cannot exceed the maximum consecutive slots in a day (default 2) |
| Same-course consistency | `same_course_consistency()` | A course cannot mix Mon/Wed with Tue/Thu, or weekday with Fri/Sat sessions |
| Day pairing | `paired_day_of()` | A Mon class is mirrored on Wed, and a Tue class on Thu, at the same time, lab and lecturer |

### Backtracking solver

1. Build the domain for every course section.
2. Order sections by domain size, smallest first (fail-first / most-constrained-variable heuristic).
3. For the next section, try each candidate assignment and check all constraints.
4. If the day has a pair (Mon↔Wed, Tue↔Thu), check the paired session against the same constraints.
5. If everything passes, commit the assignment and its pair, then recurse on the next section.
6. If a later step fails, undo the assignment and try the next candidate.
7. The search stops when every section is placed or it proves that no valid timetable exists.

Backtracking was chosen because it handles interdependent constraints well. It also guarantees a result: either a complete valid schedule or confirmation that none exists for the given inputs.

> **Note:** The solver fills the earliest available slots first. If all sections fit Mon–Thu, Friday and Saturday stay empty. They are valid days that are simply not needed. Enable **Shuffle each course domain** in the app to get more varied timetables.

## Installation

Requires **Python 3.8+**.

```bash
git clone <your-repo-url>
cd <repo-folder>
pip install streamlit pandas
```

## Usage

### Web app

```bash
streamlit run app.py
```

Then, in the browser:

1. **Sidebar inputs** (all optional, defaults are preloaded):
   - **Course codes:** comma-separated list. The first three letters must match a department code (`STA`, `DSA`, `MTH`, `APT`, `IST`).
   - **Lecturers per department:** comma-separated names. Leave blank to use placeholders such as `DSA_Lec1`.
   - **Number of labs:** 1–20 (default 5)
   - **Max consecutive slots per lecturer:** 1–5 (default 2)
   - **Shuffle each course domain:** randomises the search order
2. Click **Run scheduler**.
3. View the timetable and the summary by day, and download the CSV if you need it.
4. Use **Search for a Course** to look up one section, e.g. `DSA1080_A`.

If no feasible schedule exists, the app suggests adding lecturers or labs, or shortening the course list.

### Notebook

Open `Classroom_Scheduler.ipynb` in Jupyter and run all cells. The final cell prints the full timetable, e.g.:

```
DSA1080_A    -> Day=Mon/Wed   Time=5:30–7:10  Lab=5 Lecturer=DSA_Lec3
MTH2020_A    -> Day=Tue/Thu   Time=3:30–5:10  Lab=2 Lecturer=MTH_Lec6
```

## Default Data

- **Courses:** 65 sections across APT, DSA, IST, MTH and STA
- **Labs:** 5
- **Placeholder lecturers:** STA (8), DSA (10), MTH (15), APT (9), IST (7)

Lecturer names are anonymised placeholders. Replace them with real staff names through the app's sidebar.

## Team – Group One

| Member | ID | Responsibilities |
|--------|----|------------------|
| Betelhem Kebede | 670549 | Requirements & constraints, algorithm implementation |
| Snit Kahsay | 670552 | Algorithm implementation, GUI development, testing & documentation |
| Hana Gashaw | 670555 | Data preparation, GUI development |
| Vivian Jerono Kipsang | 670386 | Requirements & constraints, GUI development |
| Merhawit Tesfay | 670554 | Data preparation, testing & documentation |

## Disclaimer

This is an academic project. Lecturer names are placeholders, and generated timetables should be reviewed before real-world use.
