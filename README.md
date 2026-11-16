# Disaster Relief Coordination System (DRCS)

A console-based C++ system that connects citizens reporting disasters with the agencies that respond to them. It verifies incoming reports, ranks them by urgency, alerts response agencies, tracks relief resources, and keeps all data between sessions.

> Built as a team project for **Object-Oriented Programming (CT-260)**, NED University of Engineering & Technology, CS&IT. Team: Zainab Kinza Sheikh, Saita Ahuja & Aatiqa Batool.
>
> **Status:** In progress. The core logic is being converted to data structures and algorithms, and a UI/UX layer is being added.

---

## Why this exists

Pakistan's major disasters, from the 2005 Kashmir earthquake to the 2010 and 2022 floods, have repeatedly exposed the same failures in disaster response: citizens cannot reliably reach help, unverified and duplicate reports overwhelm response teams, aid goes to the loudest caller rather than the most urgent need, and agencies such as NDMA, PDMA, and Rescue 1122 coordinate by phone.

DRCS is a prototype of a single system that addresses those gaps: one reporting channel, automated verification, severity-based priority, simultaneous agency notification, and one admin view of reports, volunteers, camps, and funds.

---

## Features

**For citizens**
- Submit a disaster report with location, reporter identity, disaster type, severity (1–10), and description
- Register as a Medical or Tech volunteer and receive an auto-generated ID
- Donate money or items (food, medicine), routed to the nearest relief camp or the central warehouse
- View emergency contact numbers for rescue, medical, fire, police, and NDMA
- Menus in **English, Urdu, Sindhi, and Pashto**

**Verification and anti-abuse**
- CNIC (13 digits) and phone number validation
- Officer reports require a 3-digit officer code
- Reports containing fake-content keywords are rejected automatically
- Cooldowns prevent spam: 24 hours for the same disaster in the same area, and 30 days between reports from different areas
- A CNIC can't be used to report a different disaster in the same location

**Response**
- Each verified report receives a priority score from its disaster type and severity
- Disasters that meet severity thresholds trigger automatic evacuation alerts
- Alerts are sent to NDMA, PDMA, and Rescue 1122 at the same time
- Relief resources are deployed from the local camp first, then the central warehouse

**Admin panel** (password protected, 12 functions)
- Dashboard statistics and a full report list
- Resource overview: volunteers, camp stock, and funds
- Impose lockdowns, order and lift evacuations, and view the evacuation log
- Deploy resources, follow up on dispatched resources, add relief camps, and mark reports resolved

**Persistence**
- All data is saved to plain-text, pipe-separated files between sessions, so nothing is lost when the program closes

---

## Design

The system is organized into three layers:

| Layer | Components |
|---|---|
| Presentation | Console menus, multi-language support (`SystemConfig`) |
| Business logic | `DisasterReport`, `Verifier`, `PriorityStrategy`, `AlertSystem`, `AdminDashboard` |
| Data | Text files for reports, volunteers, camps, funds, CNIC history, and evacuations |

### Design patterns

| Pattern | Where it is used | Purpose |
|---|---|---|
| **Factory Method** | `createDisaster()` and the seven disaster subclasses | Creates the right disaster type from a string without exposing construction logic |
| **Observer** | `AlertSystem` with `NDMAObserver`, `RescueObserver`, `PDMAObserver` | Notifies every registered agency when an alert is sent |
| **Strategy** | `PriorityStrategy` with `StandardPriority` and `UrgentPriority` | Lets priority formulas be swapped without changing the report class |
| **Template Method** | `Volunteer::deploy()` with `MedicalVolunteer` and `TechVolunteer` | Keeps the deployment sequence fixed while each volunteer type customizes its steps |
| **Facade** | `AdminDashboard` | Gives the admin panel one interface over camp, fund, volunteer, and alert subsystems |

### Object-oriented features
- Inheritance and polymorphism across the seven disaster types, with pure virtual methods for type name, weight, and instructions
- Composition (`DisasterReport` contains a `Location` and a `Reporter`) and aggregation (shared data owned by the dashboard's collections)
- Deep-copy and move semantics for `DisasterReport`, which owns a raw `Disaster*`
- `unique_ptr` for volunteer lifetime management, and a virtual destructor hierarchy

---

## Disaster types

| Type | Weight | Auto-evacuation |
|---|---|---|
| Flood | 30 | Severity ≥ 6 |
| Earthquake | 50 | Severity ≥ 5 |
| Cyclone | 45 | Always |
| Landslide | 40 | Severity ≥ 7 |
| Fire | 35 | Severity ≥ 7 |
| Drought | 25 | Never |
| Pandemic | 55 | Severity ≥ 8 |

---

## Getting started

**Requirements:** a C++14-compatible compiler (for example `g++`).

```bash
g++ -std=c++14 -o drcs <source-file>.cpp
./drcs
```

On first run, the system loads 15 pre-registered government volunteers, 8 relief camps (7 cities plus a central warehouse), and Rs. 5,000,000 in starting funds. Data files are created in the working directory.

**Before any public deployment:** the officer code and admin password are defined in the source. Move them into configuration and replace the default values. The admin password is currently protected only by a non-cryptographic hash.

---

## Limitations

- Console-only interface; there is no real SMS, radio, TV, or siren integration. These are simulated with printed output.
- Data is stored in plain-text files, which is suitable for a prototype but not for concurrent or large-scale use.
- The admin password uses a simple hash, not a secure password scheme.
- The verifier's keyword filter is simple substring matching, so it can reject valid text that contains a keyword.
- Dispatch currently serves reports in list order, so the priority score informs the report but does not yet control dispatch order.

## Roadmap

- Dispatch the highest-priority report first, using a priority queue
- Replace file storage with a structured database
- Add a graphical interface and an improved verification model
- Add voice integration so people who cannot read can navigate and report by speaking, and people who cannot type can submit reports by voice

---

*Developed at NED University of Engineering & Technology, Department of Computer Science & Information Technology.*
