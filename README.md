# Chlorination Watch

Monitoring dashboard for the UNICEF inline chlorination programme in Borno State
— 64 water points across Jere, Konduga and Maiduguri (MMC).

Upload the KoboToolbox export and the page builds itself: free residual chlorine
(FRC) at each water point, stored water in households, water use, child health
referrals, and the alerts that follow from them.

Everything runs in the browser. No server, no database, no build step. **Nothing
you upload is transmitted anywhere.**

---

## What it reads

The export of the `ilc_frc_monitoring` form, as **XLSX** or **CSV**:

| Sheet | One row per | Key columns |
|---|---|---|
| submissions | visit (morning or evening) | `visit_date`, `visit_round`, `site`, `wp_status`, `frc_value`, `unit_status`, `stock_days`, `gps_waterpoint` |
| `household` | household visited | `hh_code`, `hh_site`, `hh_frc`, `hh_wp_frc`, `hh_size`, `hh_under5`, `hh_selection`, `hh_awd_case`, `hh_gps` |

Drop both CSV files at once, or one XLSX holding both sheets.

Two things it deliberately does not trust:

- **Chlorine bands are recomputed here** from `frc_value` and `hh_frc` rather than
  read from the form's `band_value` column, so the dashboard behaves the same
  whether Kobo exports choice *names* or *labels*.
- **Column headers are matched on their last path segment**, so
  `readings/frc_value` and `frc_value` both resolve. The "Include groups in
  headers" export option does not matter. Comma, semicolon and tab delimited
  CSV are all detected.

## The bands

| Reading (mg/L) | Meaning | Severity |
|---|---|---|
| 0.00 | No chlorine at all — untreated water | Critical |
| below 0.20 | Below the acceptable range | High |
| 0.20 – 0.30 | Low, still acceptable | Watch |
| **0.30 – 0.80** | **On target** | OK |
| 0.80 – 2.00 | High, still acceptable | Watch |
| above 2.00 | Above the acceptable range | High |

Stored water in the home passes at **0.20 mg/L or above**. Water use is measured
against the Sphere minimum of **15 litres per person per day**.

## Views

- **Overview** — eight indicators, where the readings fall, residual morning
  against evening, reporting coverage by day
- **Alerts** — sixteen rules graded Critical / High / Watch, split into a water
  lane and a health lane, each with its action and a copy-ready SMS or WhatsApp
  message, plus a daily digest
- **Water points** — all 64, sortable, latest reading each
- **Households** — residual lost between the point and the cup, water use per
  point, traced against randomly selected households
- **Map** — water points and households by their recorded GPS
- **Child health** — children under five with watery diarrhoea, and whether each
  referral was actually made

Each view has its own address (`#alert`, `#map`, …) so a link can open on one.

## Alerts do not send themselves

A static page has no server and browsers cannot send SMS or email. The alert
messages are **copy and paste**.

For automatic sending, point a **Kobo REST Service** at a small relay that
applies the same rules and calls an SMS gateway. Two things to design around:

1. REST Services fire on **submission creation only, never on edit** — a
   correction never reaches the relay, so a nightly sweep of the Kobo API is
   needed to catch up.
2. Retries cover roughly **two hours**. A longer outage loses the trigger
   entirely.

## Deploying

Static. No build command, no output directory, no environment variables.

**Vercel** — import this repository, leave every field at its default, deploy.
Vercel serves `index.html` at the root.

**Anywhere else** — `index.html` is self-contained apart from two CDN requests
(Google Fonts and SheetJS, for reading XLSX). Serve the folder over HTTPS. If
you remove the SheetJS tag the page still works and asks for CSV instead.

## Where data lives, by copy

| Copy | Where the upload goes | Who sees it |
|---|---|---|
| This self-hosted site | Your browser's local storage, if you choose "Remember in this browser" | Only you, only that browser |
| claude.ai artifact | A shared store on the page itself | Everyone the artifact is shared with |

On this self-hosted copy there is **no shared store**: each person who opens the
site uploads their own export. To share one dataset across a team from here you
would need a backend — Vercel Postgres or KV plus an API route — which this
repository does not include.

## Privacy

Household rows carry a household code, a landmark, household size, children
under five, and the child health fields.

- **Never commit a real export to this repository.** `.gitignore` excludes
  `*.csv`, `data/` and `exports/` for that reason. This repository contains code
  and the blank Kobo form only.
- The dashboard opens on **generated sample data**, clearly banded as such.
  Those figures are invented and describe no real household.
- Child health records tied to a household code and a GPS position are the most
  sensitive data here. Restrict who holds Kobo download rights, and treat the
  Child health view as need-to-know.

## The form

[`ILC-Kobo-Form-FRC-Monitoring.xlsx`](./ILC-Kobo-Form-FRC-Monitoring.xlsx) — the
XLSForm this dashboard reads, ready to upload to KoboToolbox
(**New project › Import an XLSForm**). It merges the daily water point record
and the household stored-water spot check into one form, with GPS at both
levels, and validates through ODK Validate.

One submission is one **visit**, not one day: a morning record and an evening
record are sent separately so a zero reading alerts within minutes rather than
sitting on a phone until evening. The dashboard reassembles the day on
`site` + `date` + `visit_round`.

## Licence

Programme material. Not for redistribution outside the implementing partners
without permission.
